# 03 Senior Engineer

## Architectural Critique (Scores 1-5)

Scale: 1 = weak, 5 = strong.

1. Scalability: **3.5/5**
   - Strength: feature-based module boundaries and strict import zones improve team scale (Ref: `apps/react-vite/.eslintrc.cjs:46-98`).
   - Limiter: three parallel app implementations duplicate logic and can drift (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:1-35`, `apps/nextjs-app/src/features/discussions/api/get-discussion.ts:1-35`).
2. TypeScript discipline: **4/5**
   - Strength: strict TS config + Zod inference + generic query helpers (Ref: `apps/react-vite/tsconfig.json:9`, `apps/react-vite/src/features/discussions/api/create-discussion.ts:10-16`, `apps/react-vite/src/lib/react-query.ts:12-26`).
   - Limiter: explicit `any` in key glue points increases hidden risk (Ref: `apps/react-vite/src/app/router.tsx:14`, `apps/react-vite/src/testing/mocks/db.ts:52`, `apps/react-vite/src/lib/__tests__/authorization.test.tsx` uses typed utilities but some test helpers allow `any` at `apps/react-vite/src/testing/test-utils.tsx:19`).
3. Separation of concerns: **4/5**
   - Strength: app/lib/features/testing layers are clearly differentiated (Ref: `apps/react-vite/src/app/index.tsx:1-10`, `apps/react-vite/src/lib/api-client.ts:16-42`, `apps/react-vite/src/features/comments/api/create-comment.ts:30-47`).
4. Testability: **4/5**
   - Strength: MSW + test utils enable realistic integration tests without brittle mocks (Ref: `apps/react-vite/src/testing/setup-tests.ts:8-27`, `apps/react-vite/src/testing/test-utils.tsx:57-94`).
   - Limiter: handler-level permission negative tests are thin.
5. Maintainability: **3.5/5**
   - Strength: consistent API file conventions reduce cognitive load (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:20-42`, `apps/react-vite/src/features/users/api/get-users.ts:11-27`).
   - Limiter: duplicated route/data fetch in some layers can confuse ownership (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:41-43`, `apps/react-vite/src/features/discussions/components/discussion-view.tsx:9-11`).
6. Security posture: **2.5/5** (for production), **4/5** (for demo intent)
   - Strong for demo transparency: explicit comments warn about cookie security gaps (Ref: `apps/react-vite/src/testing/mocks/handlers/auth.ts:100-102`).
   - Weak for production: token is reversible base64 payload, not signed JWT (Ref: `apps/react-vite/src/testing/mocks/utils.ts:68-71`).
7. Performance: **3/5**
   - Strength: route-based lazy loading and prefetching patterns (Ref: `apps/react-vite/src/app/router.tsx:28-79`, `apps/react-vite/src/features/discussions/components/discussions-list.tsx:63-67`).
   - Limiter: sequential awaits inside “parallel” loader promises and duplicate discussion queries.

## Performance Audit (Findings + Corrected Snippets)

## Finding 1: Loader “parallel” fetch executes sequentially

Issue: `await` is used while building the `promises` array, so one fetch may block the next.

Current pattern (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:23-28`):

```ts
const promises = [
  queryClient.getQueryData(discussionQuery.queryKey) ??
    (await queryClient.fetchQuery(discussionQuery)),
  queryClient.getQueryData(commentsQuery.queryKey) ??
    (await queryClient.fetchInfiniteQuery(commentsQuery)),
] as const;
```

Corrected pattern:

```ts
const discussionPromise =
  queryClient.getQueryData(discussionQuery.queryKey) ??
  queryClient.fetchQuery(discussionQuery);

const commentsPromise =
  queryClient.getQueryData(commentsQuery.queryKey) ??
  queryClient.fetchInfiniteQuery(commentsQuery);

const [discussion, comments] = await Promise.all([
  discussionPromise,
  commentsPromise,
]);
```

## Finding 2: Duplicate query subscription on discussion detail

Issue: route loads discussion, then `DiscussionRoute` and `DiscussionView` each invoke `useDiscussion`, increasing render/query overhead.

Evidence:
- `DiscussionRoute` uses `useDiscussion` (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:41-43`).
- `DiscussionView` also uses `useDiscussion` (Ref: `apps/react-vite/src/features/discussions/components/discussion-view.tsx:9-11`).

Corrected direction:

```tsx
// Route owns useDiscussion and passes data downward
const discussion = discussionQuery.data?.data;
<DiscussionView discussion={discussion} />
```

```tsx
// DiscussionView becomes pure presentational component
export const DiscussionView = ({ discussion }: { discussion: Discussion }) => {
  // no extra query call
};
```

## Finding 3: Notification list can grow unbounded in long sessions

Evidence: store only appends/removes manually; no TTL or max length guard (Ref: `apps/react-vite/src/components/ui/notifications/notifications-store.ts:19-30`).

Corrected direction:

```ts
addNotification: (notification) =>
  set((state) => {
    const next = [...state.notifications, { id: nanoid(), ...notification }];
    return { notifications: next.slice(-5) }; // keep only latest 5
  }),
```

## Security Audit (Findings + Corrected Snippets)

## Finding 1: Token format is reversible and unsigned

Evidence: auth token is just base64-encoded JSON of sanitized user (Ref: `apps/react-vite/src/testing/mocks/utils.ts:68-71`).

Risk: token tampering is trivial in any non-demo environment.

Corrected direction:

```ts
// Replace mock encode/decode with signed JWT verification in real backend
import jwt from 'jsonwebtoken';

const token = jwt.sign({ sub: user.id, role: user.role }, JWT_SECRET, { expiresIn: '15m' });
const decoded = jwt.verify(token, JWT_SECRET);
```

## Finding 2: Cookie attributes lack `HttpOnly`, `Secure`, `SameSite`

Evidence: mock handler sets `Set-Cookie` with only `Path=/` and comment acknowledging gap (Ref: `apps/react-vite/src/testing/mocks/handlers/auth.ts:100-102`, `apps/react-vite/src/testing/mocks/handlers/auth.ts:125-126`).

Corrected direction:

```ts
'Set-Cookie': `${AUTH_COOKIE}=${token}; Path=/; HttpOnly; Secure; SameSite=Lax;`
```

## Finding 3: Client-side policy checks are not sufficient by themselves

Evidence: UI hides actions with `Authorization`, but true enforcement is in handlers (`requireAdmin` / ownership filters) (Ref: `apps/react-vite/src/features/discussions/components/delete-discussion.tsx:28-52`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:204-214`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:114-118`).

Recommendation: continue dual-layer checks; never remove handler-side constraints.

## TypeScript Discipline Review

Strengths:

1. strict mode enabled (Ref: `apps/react-vite/tsconfig.json:9`).
2. typed API domain core (Ref: `apps/react-vite/src/types/api.ts:20-50`).
3. schema-driven form payload typing via Zod inference (Ref: `apps/react-vite/src/lib/auth.tsx:23-55`).

Weak spots:

1. `any` in router conversion function (`m: any`) dilutes route-module typing (Ref: `apps/react-vite/src/app/router.tsx:14`).
2. mock data error handling typed as `any` hides narrow failure handling (Ref: `apps/react-vite/src/testing/mocks/db.ts:52`).
3. loose test utility parameters (`userProperties?: any`) can hide malformed fixtures (Ref: `apps/react-vite/src/testing/test-utils.tsx:19`).

## Custom Abstractions Inventory

1. `configureAuth` wrapper around auth endpoints and route guard (Ref: `apps/react-vite/src/lib/auth.tsx:62-89`).
2. Query option builders per feature (`getDiscussionQueryOptions`, etc.) (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:15-20`).
3. Form system wrapper (`Form`, `FormField`, etc.) around React Hook Form + Zod (Ref: `apps/react-vite/src/components/ui/form/form.tsx:173-217`).
4. Authorization abstraction (`Authorization` component + `POLICIES`) (Ref: `apps/react-vite/src/lib/authorization.tsx:14-82`).
5. Notification store + renderer (Ref: `apps/react-vite/src/components/ui/notifications/notifications-store.ts:11-32`, `apps/react-vite/src/components/ui/notifications/notifications.tsx:4-20`).

## Testing Assessment + One Runnable Test

Current coverage posture:

- Integration-heavy route tests exist for discussion operations (Ref: `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx:32-145`).
- Authorization component behavior is tested at UI gate level (Ref: `apps/react-vite/src/lib/__tests__/authorization.test.tsx:5-83`).
- Gap: no explicit integration test that non-owner `USER` cannot delete someone else’s comment through handler policy.

### Runnable Test Proposal

Create file: `apps/react-vite/src/features/comments/api/__tests__/delete-comment-policy.test.tsx`

```tsx
import { createDiscussion, createUser, renderApp, screen, userEvent, waitFor } from '@/testing/test-utils';
import { db } from '@/testing/mocks/db';
import { CommentsList } from '@/features/comments/components/comments-list';

test('user cannot delete another user comment', async () => {
  const owner = await createUser({ role: 'USER' });
  const outsider = await createUser({ role: 'USER', teamId: owner.teamId });
  const discussion = await createDiscussion({ teamId: owner.teamId, authorId: owner.id });

  await db.comment.create({
    id: 'c-1',
    body: 'owner comment',
    authorId: owner.id,
    discussionId: discussion.id,
    createdAt: Date.now(),
  });

  await renderApp(<CommentsList discussionId={discussion.id} />, {
    user: outsider,
    path: '/app/discussions/:discussionId',
    url: `/app/discussions/${discussion.id}`,
  });

  // outsider should not see delete control for owner's comment
  expect(screen.queryByRole('button', { name: /delete comment/i })).not.toBeInTheDocument();

  await waitFor(async () => {
    const commentInDb = await db.comment.findFirst({ where: { id: { equals: 'c-1' } } });
    expect(commentInDb).toBeTruthy();
  });
});
```

## Bug Injection Exercise (Symptoms + Test Scenario Only)

1. Symptom: “Comments sometimes disappear after creating one.”
   - Test scenario: create comment on page 2 of paginated comments; verify correct query key invalidation and next-page behavior.
2. Symptom: “Admin can’t open Users page after login redirect.”
   - Test scenario: login from deep URL with `redirectTo`, assert role-aware nav and protected route behavior.
3. Symptom: “Deleting discussion works in UI but list still shows stale row until refresh.”
   - Test scenario: delete discussion and assert cache invalidation for `['discussions']`.
4. Symptom: “Random unauthorized errors after hot reload.”
   - Test scenario: verify cookie presence and `requireAuth` path in both browser worker and mock-server modes.
5. Symptom: “User can delete someone else’s comment in one environment but not another.”
   - Test scenario: compare browser MSW mode vs Express middleware mode role enforcement on `DELETE /comments/:commentId`.

## Git History Learning Exercise

Use recent history as signals about architecture evolution (Ref: `git log --oneline -n 20`, captured during inspection).

1. `10f330f Add nextjs app and pages version of the app (#198)`
   - Implies architecture extraction into reusable feature modules across multiple runtime shells.
2. `4da36db Upgrade to react router 7 - lib mode (#227)`
   - Implies router internals changed and data-loading patterns likely adapted.
3. `5cdc208 Rename directory from "test" to "testing" (#221)`
   - Implies intentional standardization and stronger testing identity.
4. `74ff903 Next app router improvements (#222)`
   - Implies parity and divergence management between app implementations.
5. `79710eb Upgrade @playwright/test and its dependencies to fix CI (#259)`
   - Implies CI reliability and e2e stability are active maintenance concerns.

## If I Owned This Codebase (Refactor Plan)

1. Consolidate shared feature logic into reusable package or sync tooling across 3 apps.
   - Effort: High (2-3 weeks)
   - Impact: High
2. Replace mock token encoding with signed JWT in non-demo deployment path.
   - Effort: Medium (3-5 days)
   - Impact: High (security)
3. Fix sequential loader awaits and remove duplicate discussion query invocation.
   - Effort: Small (1 day)
   - Impact: Medium (performance + clarity)
4. Introduce typed route-module interfaces replacing `any` in router conversion.
   - Effort: Medium (2-3 days)
   - Impact: Medium (type safety)
5. Expand negative permission integration tests (owner vs non-owner, admin vs user).
   - Effort: Small/Medium (2 days)
   - Impact: High (regression prevention)
6. Add explicit API response schema validation for handler responses in tests.
   - Effort: Medium (3-4 days)
   - Impact: Medium (contract drift prevention)

