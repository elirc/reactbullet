### Mission 18: Reverse-Engineer the Architecture Decisions
**Tier:** Senior  
**Time Estimate:** 60 minutes  
**Goal:** Infer architectural intent from constraints and repetition patterns.

**The Concept:**  
Architecture decisions are fossils: you infer intent from structure and tradeoffs.

**Design Intent Before You Read the Code:**  
Compare equivalent files across the three app implementations.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/users/api/get-users.ts:1-27`
- `apps/nextjs-app/src/features/users/api/get-users.ts:1-27`
- `apps/nextjs-pages/src/features/users/api/get-users.ts:1-27`
- `apps/react-vite/src/app/router.tsx:24-88`
- `apps/nextjs-app/src/app/layout.tsx:18-42`

```ts
// identical feature API module across app variants
export const getUsers = (): Promise<{ data: User[] }> => {
  return api.get(`/users`);
};
// repeated by design: domain parity across routing shells
```

```tsx
// vite uses explicit router tree + loaders
createBrowserRouter([...]);

// next app router uses file-system routes + layout prefetch/dehydrate
await queryClient.prefetchQuery(getUserQueryOptions());
```

**The Aha Moment:**  
The repo optimizes for teaching portability of domain architecture across frameworks.

**Socratic Checkpoint:**  
1. What was likely prioritized: DRYness or framework parity demonstration?  
2. Which duplication is intentional vs accidental?  
3. Where could drift become dangerous fastest?  
4. How would you preserve parity with less manual copy?  
5. What ownership model would you propose for the three app variants?

How to self-grade:
- Makes explicit tradeoff statements backed by file evidence.
- Proposes an actionable governance strategy.

**Connects To:**  
Mission 24 and Mission 25.

---

### Mission 19: Find the Bugs Before They Happen
**Tier:** Senior  
**Time Estimate:** 55 minutes  
**Goal:** Predict likely bugs from architecture and implementation seams.

**The Concept:**  
Preventive debugging means reading code for future incident patterns.

**Design Intent Before You Read the Code:**  
Hunt for mismatch risks: cache keys, authorization symmetry, response shape, side-effect timing.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/discussions/api/get-discussions.ts:24`
- `apps/react-vite/src/features/discussions/api/delete-discussion.ts:29-31`
- `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:23-31`
- `apps/react-vite/src/lib/authorization.tsx:63-82`

```ts
queryKey: page ? ['discussions', { page }] : ['discussions'],
// potential stale-data edge if mutations only invalidate ['discussions'] broadly
```

```ts
const promises = [
  queryClient.getQueryData(discussionQuery.queryKey) ?? (await queryClient.fetchQuery(discussionQuery)),
  queryClient.getQueryData(commentsQuery.queryKey) ?? (await queryClient.fetchInfiniteQuery(commentsQuery)),
]; // async sequencing subtlety can hurt latency
```

**The Aha Moment:**  
High-risk bugs live at boundary seams, not inside obvious happy-path blocks.

**Socratic Checkpoint:**  
1. Which cache key pattern is most fragile?  
2. Where could auth logic diverge between UI and handler?  
3. Which async pattern could create avoidable latency?  
4. Which route/component pair duplicates fetch responsibility?  
5. What one test would catch the highest-risk seam first?

How to self-grade:
- Predicts concrete bug symptom from each seam.
- Prioritizes risks by user impact.

**Connects To:**  
Mission 20 and Mission 23.

---

### Mission 20: The Bug Injection Challenge
**Tier:** Senior  
**Time Estimate:** 50 minutes  
**Goal:** Practice diagnosing failures from user symptoms without code hints.

**The Concept:**  
Incident response starts from symptoms, not source files.

**Design Intent Before You Read the Code:**  
Use symptom-first triage and map to likely layers.

**Find It In The Code:**  
Baseline triage anchors:
- `apps/react-vite/src/lib/api-client.ts:26-40`
- `apps/react-vite/src/testing/mocks/handlers/comments.ts:98-131`
- `apps/react-vite/src/features/comments/components/comments-list.tsx:80-90`

```ts
if (error.response?.status === 401) {
  window.location.href = paths.auth.login.getHref(redirectTo);
} // symptom: sudden nav to login
```

```ts
...(user?.role === 'USER' && { authorId: { equals: user.id } }),
// symptom: delete works for admin but fails for non-owner user
```

```tsx
{commentsQuery.hasNextPage && (
  <Button onClick={() => commentsQuery.fetchNextPage()}>Load More Comments</Button>
)}
// symptom: button disappears if pagination meta is wrong
```

**The Aha Moment:**  
Symptom-to-layer mapping is a senior debugging superpower.

**Socratic Checkpoint:**  
1. Which symptom indicates transport-layer issue vs policy issue?  
2. What logs would you add first and where?  
3. How do you isolate cache issue from handler issue?  
4. Which reproducible scenario would you script first?  
5. What evidence do you gather before proposing a fix?

How to self-grade:
- Produces a layer-by-layer triage plan.
- Prioritizes reproducibility and evidence collection.

**Connects To:**  
Mission 21 and Mission 22.

---

### Mission 21: Performance X-Ray
**Tier:** Senior  
**Time Estimate:** 55 minutes  
**Goal:** Identify concrete performance bottlenecks and propose targeted fixes.

**The Concept:**  
Performance work is reducing unnecessary work, not generic “optimization.”

**Design Intent Before You Read the Code:**  
Look for duplicated fetches, sequential waits, and unbounded UI state growth.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:23-31`
- `apps/react-vite/src/features/discussions/components/discussion-view.tsx:9-11`
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts:19-25`

```ts
const discussionPromise =
  queryClient.getQueryData(discussionQuery.queryKey) ??
  queryClient.fetchQuery(discussionQuery);
// make concurrent, avoid await during promise array construction
```

```tsx
// pass discussion data from route to view to avoid repeated hook call
<DiscussionView discussion={discussion} />
```

```ts
notifications: next.slice(-5), // cap memory/DOM pressure for long sessions
```

**The Aha Moment:**  
Performance wins come from removing duplicate work at boundaries.

**Socratic Checkpoint:**  
1. Which performance issue has highest user-visible impact?  
2. Which fix is lowest risk to ship quickly?  
3. How would you measure before/after?  
4. What regression test protects the fix?  
5. Which “optimization” would you avoid as premature?

How to self-grade:
- Ties each optimization to a measurable symptom.
- Differentiates high-impact vs low-impact improvements.

**Connects To:**  
Mission 23.

---

### Mission 22: The Security Audit
**Tier:** Senior  
**Time Estimate:** 60 minutes  
**Goal:** Evaluate security posture by tracing trust boundaries and policy enforcement.

**The Concept:**  
Security auditing is proving where trust is earned, not assumed.

**Design Intent Before You Read the Code:**  
Inspect token model, cookie attributes, policy checks, and sanitization.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/testing/mocks/utils.ts:53-103`
- `apps/react-vite/src/testing/mocks/handlers/auth.ts:100-102`
- `apps/react-vite/src/components/ui/md-preview/md-preview.tsx:12-16`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts:142-149`

```ts
const encodedToken = encode(sanitizedUser); 
// demo token is reversible payload, not signed JWT
```

```ts
'Set-Cookie': `${AUTH_COOKIE}=${result.jwt}; Path=/;` 
// missing HttpOnly/Secure/SameSite in demo handler
```

```tsx
__html: DOMPurify.sanitize(parse(value) as string), 
// markdown rendering includes sanitization guard
```

```ts
requireAdmin(user); 
// server-side authorization check for privileged mutation
```

**The Aha Moment:**  
Strong security posture comes from defense-in-depth across rendering, auth, and policy layers.

**Socratic Checkpoint:**  
1. Which findings are acceptable for demo-only context but not production?  
2. Which protections already exist against XSS?  
3. Where is authorization enforced server-side?  
4. What is your first production-hardening change?  
5. How would you test cookie/session hardening?

How to self-grade:
- Separates demo tradeoffs from production requirements.
- Lists concrete hardening actions in priority order.

**Connects To:**  
Mission 23 and Mission 25.

---

### Mission 23: Write the Test That Doesn't Exist
**Tier:** Senior  
**Time Estimate:** 70 minutes  
**Goal:** Design and write a missing high-value test for risky behavior.

**The Concept:**  
Great tests lock risky invariants, not easy scenarios.

**Design Intent Before You Read the Code:**  
Pick a permission or cache coherence edge case with real user impact.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/testing/test-utils.tsx:57-94`
- `apps/react-vite/src/testing/mocks/db.ts:39-97`
- `apps/react-vite/src/features/comments/components/comments-list.tsx:54-74`
- `apps/react-vite/src/lib/__tests__/authorization.test.tsx:5-83`

```tsx
// target invariant: non-owner USER must not delete owner comment
<Authorization policyCheck={POLICIES['comment:delete'](user.data as User, comment)}>
  <DeleteComment discussionId={discussionId} id={comment.id} />
</Authorization>
```

```ts
// support deterministic fixture setup for policy tests
await db.comment.create({ id: 'c-1', body: 'x', authorId: owner.id, discussionId, createdAt: Date.now() });
```

**The Aha Moment:**  
The best missing tests are usually around boundaries and policies, not rendering details.

**Socratic Checkpoint:**  
1. Which invariant are you pinning?  
2. Why is this invariant high risk?  
3. Which setup helper enables deterministic test state?  
4. What exact assertion proves policy enforcement?  
5. How will this test fail if a future refactor regresses behavior?

How to self-grade:
- Defines one crisp invariant and one crisp failure signal.
- Uses realistic integration setup rather than over-mocking internals.

**Connects To:**  
Mission 24 and Mission 25.

---

### Mission 24: The Git History Tells a Story
**Tier:** Senior  
**Time Estimate:** 45 minutes  
**Goal:** Infer architecture evolution and team priorities from commit history.

**The Concept:**  
History is a design artifact: commit messages expose pressures and direction.

**Design Intent Before You Read the Code:**  
Use recent history to infer why current structure exists.

**Find It In The Code:**  
Use terminal output from `git log --oneline -n 20` (captured in inspection), with these anchors:
- `10f330f Add nextjs app and pages version of the app (#198)`
- `4da36db Upgrade to react router 7 - lib mode (#227)`
- `5cdc208 Rename directory from "test" to "testing" (#221)`
- `74ff903 Next app router improvements (#222)`
- `79710eb Upgrade @playwright/test and its dependencies to fix CI (#259)`

```text
10f330f Add nextjs app and pages version of the app (#198)
4da36db Upgrade to react router 7 - lib mode (#227)
74ff903 Next app router improvements (#222)
79710eb Upgrade @playwright/test and its dependencies to fix CI (#259)
```

**The Aha Moment:**  
You can explain architecture better when you understand the sequence of constraints that shaped it.

**Socratic Checkpoint:**  
1. What does adding two Next variants imply about project goals?  
2. How might Router 7 upgrade have impacted loader patterns?  
3. Why does test infra maintenance show up in history frequently?  
4. What future drift risk does this history hint at?  
5. Which ownership process would you introduce next?

How to self-grade:
- Connects commits to architectural consequences, not just chronology.
- Proposes one process improvement grounded in history signals.

**Connects To:**  
Mission 25 for operational reference docs and team onboarding standards.

