### Mission 9: State Has a Home and a Reason
**Tier:** Mid-Level  
**Time Estimate:** 45 minutes  
**Goal:** Distinguish server state from UI/global client state and justify each placement.

**The Concept:**  
State placement is like choosing storage: fridge for perishable (remote data cache), shelf for local utilities (notifications).

**Design Intent Before You Read the Code:**  
Find one state slice in React Query and one in Zustand. Compare lifecycle and invalidation strategy.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/discussions/api/get-discussions.ts:20-42`
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts:11-32`
- `apps/react-vite/src/components/ui/notifications/notifications.tsx:4-20`

```ts
return useQuery({
  ...getDiscussionsQueryOptions({ page }), // remote cache key controls freshness
  ...queryConfig,
});
```

```ts
export const useNotifications = create<NotificationsStore>((set) => ({
  notifications: [], // purely client/UI state
  addNotification: (notification) => set((state) => ({
    notifications: [...state.notifications, { id: nanoid(), ...notification }],
  })),
}));
```

**The Aha Moment:**  
Correct state home reduces both complexity and accidental coupling.

**Socratic Checkpoint:**  
1. Why are discussions in React Query, not Zustand?  
2. Why are toast notifications in Zustand, not React Query?  
3. What invalidates discussions data?  
4. What dismisses notifications data?  
5. What bug appears if these are swapped?

How to self-grade:
- Uses lifecycle reasoning (source-of-truth, refetch, invalidation).
- Names specific APIs used in each store type.

**Connects To:**  
Mission 10 and Mission 11.

---

### Mission 10: The Custom Hook Ecosystem
**Tier:** Mid-Level  
**Time Estimate:** 40 minutes  
**Goal:** Map hook layering from feature hooks to shared auth hooks.

**The Concept:**  
Hooks are local service adapters. Too flat or too deep both create fragility.

**Design Intent Before You Read the Code:**  
Track one feature hook and one shared hook, then identify their boundaries.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/comments/api/get-comments.ts:22-47`
- `apps/react-vite/src/lib/auth.tsx:62-76`
- `apps/react-vite/src/hooks/use-disclosure.ts:3-10`

```ts
export const useInfiniteComments = ({ discussionId }: UseCommentsOptions) => {
  return useInfiniteQuery({
    ...getInfiniteCommentsQueryOptions(discussionId), // feature-specific wrapper
  });
};
```

```ts
export const { useUser, useLogin, useLogout, useRegister, AuthLoader } =
  configureAuth(authConfig); // shared auth hook ecosystem
```

```ts
export const useDisclosure = (initial = false) => {
  const [isOpen, setIsOpen] = React.useState(initial);
  return { isOpen, open, close, toggle };
};
```

**The Aha Moment:**  
Custom hooks are boundaries of responsibility, not just convenience functions.

**Socratic Checkpoint:**  
1. What does `useInfiniteComments` hide from components?  
2. Why does auth use a shared adapter hook set?  
3. What is the role of tiny utility hooks like `useDisclosure`?  
4. How would you detect hook over-abstraction?  
5. Which hook would you avoid mocking in integration tests?

How to self-grade:
- Differentiates feature, shared-platform, and utility hook layers.
- Gives one practical refactoring rule.

**Connects To:**  
Mission 12 and Mission 14.

---

### Mission 11: Side Effects Are Promises to the System
**Tier:** Mid-Level  
**Time Estimate:** 45 minutes  
**Goal:** Audit mutation side effects for correctness and completeness.

**The Concept:**  
A side effect is a promise you make to future code: “state after this action will be coherent.”

**Design Intent Before You Read the Code:**  
Track `onSuccess` side effects for both cache and UX feedback.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/comments/api/create-comment.ts:30-47`
- `apps/react-vite/src/features/comments/components/create-comment.tsx:18-27`
- `apps/react-vite/src/lib/api-client.ts:26-40`

```ts
return useMutation({
  onSuccess: (...args) => {
    queryClient.invalidateQueries({
      queryKey: getInfiniteCommentsQueryOptions(discussionId).queryKey, // cache coherence promise
    });
    onSuccess?.(...args); // caller-level side effects preserved
  },
  mutationFn: createComment,
});
```

```ts
addNotification({
  type: 'success',
  title: 'Comment Created', // UX side effect on mutation success
});
```

**The Aha Moment:**  
Side effects are where architecture quality is felt by users.

**Socratic Checkpoint:**  
1. Why invalidate query instead of mutating cache directly here?  
2. What happens if `onSuccess?.(...args)` is removed?  
3. Which side effects happen in API client interceptor vs mutation hook?  
4. What breaks if query key is wrong?  
5. How would you test side-effect correctness?

How to self-grade:
- Mentions both cache and UI side effects.
- Identifies at least one side-effect ordering risk.

**Connects To:**  
Mission 15 and Mission 19.

---

### Mission 12: The Full API Contract
**Tier:** Mid-Level  
**Time Estimate:** 55 minutes  
**Goal:** Reconstruct the app’s API from code and validate contract symmetry.

**The Concept:**  
API contracts are treaties: both sides must keep shape and semantics aligned.

**Design Intent Before You Read the Code:**  
Pair each feature fetcher with handler endpoint and response shape.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/discussions/api/get-discussions.ts:7-18`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts:19-73`
- `apps/react-vite/src/features/users/api/get-users.ts:7-15`
- `apps/react-vite/src/testing/mocks/handlers/users.ts:21-40`

```ts
// fetcher expects envelope { data, meta }
export const getDiscussions = (page = 1): Promise<{ data: Discussion[]; meta: Meta }> => {
  return api.get(`/discussions`, { params: { page } });
};
```

```ts
// handler returns matching envelope
return HttpResponse.json({
  data: result,
  meta: { page, total, totalPages },
});
```

**The Aha Moment:**  
Contract drift usually starts as tiny shape mismatches.

**Socratic Checkpoint:**  
1. Which endpoints return raw entities vs wrapped `{data}`?  
2. Why should this be standardized?  
3. Where are type expectations declared for each fetcher?  
4. What test catches contract drift earliest?  
5. Which endpoint has role-dependent behavior that should be documented?

How to self-grade:
- Produces an endpoint map with request/response shape.
- Cites at least one asymmetry risk.

**Connects To:**  
Mission 14 and Mission 22.

---

### Mission 13: The Middleware Chain
**Tier:** Mid-Level  
**Time Estimate:** 45 minutes  
**Goal:** Explain every policy layer between component and handler.

**The Concept:**  
Middleware chains are airport security lanes: each checkpoint validates a different thing.

**Design Intent Before You Read the Code:**  
Distinguish transport middleware (Axios/Express) from business-policy checks (auth/role).

**Find It In The Code:**  
Open:
- `apps/react-vite/src/lib/api-client.ts:7-42`
- `apps/react-vite/mock-server.ts:12-22`
- `apps/react-vite/src/testing/mocks/utils.ts:80-110`

```ts
api.interceptors.request.use(authRequestInterceptor); // client transport policy
api.interceptors.response.use(
  (response) => response.data,
  (error) => { /* notify + redirect on 401 */ },
);
```

```ts
app.use(cors({ origin: env.APP_URL, credentials: true }));
app.use(express.json());
app.use(createMiddleware(...handlers)); // server-side middleware chain
```

```ts
export function requireAuth(cookies: Record<string, string>) {
  const encodedToken = cookies[AUTH_COOKIE] || Cookies.get(AUTH_COOKIE);
  // auth policy gate before handler logic proceeds
}
```

**The Aha Moment:**  
Reliability comes from layered checks, not one “smart” function.

**Socratic Checkpoint:**  
1. Which layer adds `withCredentials`?  
2. Which layer checks auth token existence?  
3. Which layer enforces admin role?  
4. Where is unauthorized redirect performed?  
5. What risks appear if one layer is removed?

How to self-grade:
- Correctly sequences middleware/policy checkpoints.
- Identifies one layer-specific failure symptom.

**Connects To:**  
Mission 21 and Mission 22.

---

### Mission 14: End-to-End Feature Trace
**Tier:** Mid-Level  
**Time Estimate:** 60 minutes  
**Goal:** Trace “create comment” from button click to persisted result and UI refresh.

**The Concept:**  
Feature tracing is operational debugging in advance.

**Design Intent Before You Read the Code:**  
Use chronological order: UI event -> mutation hook -> API client -> handler -> DB -> invalidation -> re-render.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/comments/components/create-comment.tsx:51-57`
- `apps/react-vite/src/features/comments/api/create-comment.ts:17-47`
- `apps/react-vite/src/testing/mocks/handlers/comments.ts:75-90`
- `apps/react-vite/src/testing/mocks/db.ts:80-85`

```tsx
onSubmit={(values) => {
  createCommentMutation.mutate({ data: values }); // user action starts feature flow
}}
```

```ts
export const createComment = ({ data }: { data: CreateCommentInput }): Promise<Comment> => {
  return api.post('/comments', data); // transport hop
};
```

```ts
const result = db.comment.create({
  authorId: user?.id,
  ...data,
});
await persistDb('comment'); // persistence side effect
return HttpResponse.json(result);
```

**The Aha Moment:**  
If you can narrate this flow without reading, you can debug it under pressure.

**Socratic Checkpoint:**  
1. Which line injects `discussionId` into payload?  
2. Where is authenticated user attached as comment author?  
3. How does list refresh happen after create?  
4. Which layer enforces auth?  
5. What if persistence fails after create?

How to self-grade:
- Traces all seven stages without missing one.
- Names at least one partial-failure recovery concern.

**Connects To:**  
Mission 19 and Mission 23.

---

### Mission 15: Read the Diff Like an Engineer
**Tier:** Mid-Level  
**Time Estimate:** 50 minutes  
**Goal:** Evaluate a hypothetical change request using risk-first diff reading.

**The Concept:**  
Diff reading is threat modeling with code context.

**Design Intent Before You Read the Code:**  
Assume requested change: “Add `editedAt` to comments and show it in UI.”

**Find It In The Code:**  
Open baseline files:
- `apps/react-vite/src/types/api.ts:46-50`
- `apps/react-vite/src/testing/mocks/handlers/comments.ts:36-67`
- `apps/react-vite/src/features/comments/components/comments-list.tsx:62-77`

```ts
export type Comment = Entity<{
  body: string;
  discussionId: string;
  author: User;
}>; // diff should extend contract here first
```

```ts
const comments = db.comment.findMany(/* ... */).map(({ authorId, ...comment }) => ({
  ...comment, // diff must ensure editedAt survives mapping
  author: author ? sanitizeUser(author) : {},
}));
```

```tsx
<span className="text-xs font-semibold">
  {formatDate(comment.createdAt)} {/* diff may introduce dual timestamp rendering */}
</span>
```

**The Aha Moment:**  
Good diff reading asks “what else must change?” before “does this compile?”

**Socratic Checkpoint:**  
1. Which files must change for contract + persistence + UI parity?  
2. Where can hidden stale-cache bugs appear?  
3. What tests must be added/updated?  
4. Which layer should set `editedAt` truthfully?  
5. How can this change break backward compatibility?

How to self-grade:
- Includes all touched layers, not only UI.
- Identifies at least two regression vectors.

**Connects To:**  
Mission 19 and Mission 24.

---

### Mission 16: Composition Over Inheritance
**Tier:** Mid-Level  
**Time Estimate:** 40 minutes  
**Goal:** Identify composition patterns and explain why they scale in this app.

**The Concept:**  
Composition is assembling Lego bricks; inheritance is carving one big statue.

**Design Intent Before You Read the Code:**  
Focus on shared primitives reused by feature components.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/components/ui/form/form.tsx:182-205`
- `apps/react-vite/src/features/discussions/components/create-discussion.tsx:47-70`
- `apps/react-vite/src/components/ui/button/button.tsx:60-70`

```tsx
<Form id="create-discussion" schema={createDiscussionInputSchema}>
  {({ register, formState }) => (
    <>
      <Input registration={register('title')} error={formState.errors['title']} />
      <Textarea registration={register('body')} error={formState.errors['body']} />
    </>
  )}
</Form>
// feature composes shared primitives instead of subclassing a base form component
```

```tsx
const Comp = asChild ? Slot : 'button'; // host element composition point
```

**The Aha Moment:**  
Composition keeps feature code flexible while shared components stay small.

**Socratic Checkpoint:**  
1. Where does form behavior live vs feature behavior?  
2. Why is render-prop form API useful here?  
3. How does `asChild` enable composition in routing/link contexts?  
4. What inheritance-style alternative would be harder to maintain?  
5. Which abstraction here is most reusable across features?

How to self-grade:
- Gives concrete composition examples from files.
- Explains one maintainability gain and one tradeoff.

**Connects To:**  
Mission 17 and Mission 18.

---

### Mission 17: TypeScript's Hidden Work
**Tier:** Mid-Level  
**Time Estimate:** 45 minutes  
**Goal:** Spot implicit type guarantees that reviewers should verify consciously.

**The Concept:**  
Type systems do hidden labor; if you stop noticing it, regressions slip through reviews.

**Design Intent Before You Read the Code:**  
Find inferred types, generic constraints, and places where `any` bypasses safety.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/lib/react-query.ts:12-26`
- `apps/react-vite/src/components/ui/table/table.tsx:132-177`
- `apps/react-vite/src/app/router.tsx:14-21`

```ts
export type QueryConfig<T extends (...args: any[]) => any> = Omit<
  ReturnType<T>,
  'queryKey' | 'queryFn'
>;
// callers inherit the exact option shape of each query builder
```

```ts
export const Table = <Entry extends BaseEntity>({ data, columns }: TableProps<Entry>) => {
  // Entry must have id + createdAt through BaseEntity
};
```

```ts
const convert = (queryClient: QueryClient) => (m: any) => {
  // 'any' bypasses type safety for lazy route modules
};
```

**The Aha Moment:**  
Senior-level TS review is mostly about preserving invisible guarantees.

**Socratic Checkpoint:**  
1. Which generic here gives the highest leverage?  
2. Where does `any` currently leak risk?  
3. What type test or compile rule could catch route-module drift?  
4. Why do inferred configs reduce copy/paste bugs?  
5. What would break if `Table` dropped `BaseEntity` constraint?

How to self-grade:
- Names one hidden guarantee and one hidden risk.
- Suggests one concrete typing improvement.

**Connects To:**  
Mission 18 and Mission 23.

