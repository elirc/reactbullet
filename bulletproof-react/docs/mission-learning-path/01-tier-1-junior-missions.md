### Mission 1: The App's Heartbeat
**Tier:** Junior  
**Time Estimate:** 35 minutes  
**Goal:** Explain exactly how this app starts and why mock boot happens before render.

**The Concept:**  
A startup sequence is like unlocking a building in order: utilities first, then lights, then people. Here, mock infra is part of “utilities.”

**Design Intent Before You Read the Code:**  
You are looking for execution order, not component visuals. Identify who blocks render and why.

**Find It In The Code:**  
Open `apps/react-vite/src/main.tsx:1-17` and `apps/react-vite/src/testing/mocks/index.ts:1-10`.

```tsx
enableMocking().then(() => {
  // render waits for mock setup to finish
  createRoot(root).render(
    <React.StrictMode>
      <App /> {/* app startup begins only after mock init path */}
    </React.StrictMode>,
  );
});
```

```ts
if (env.ENABLE_API_MOCKING) {
  const { worker } = await import('./browser'); // runtime-load browser worker
  const { initializeDb } = await import('./db'); // seed/reload fake DB
  await initializeDb();
  return worker.start(); // intercept network requests
}
```

**The Aha Moment:**  
Startup includes infrastructure readiness, not only UI mounting.

**Socratic Checkpoint:**  
1. Why is `enableMocking()` asynchronous?  
2. What breaks if render happens before `worker.start()`?  
3. Which env variable controls this branch?  
4. Why dynamic-import mock modules instead of static import?  
5. Where would you add startup diagnostics logs?

How to self-grade:
- Mentions exact async chain and gating effect.
- References env contract (`VITE_APP_ENABLE_API_MOCKING`).
- Describes one practical failure mode (unhandled requests or empty DB).

**Connects To:**  
Mission 2 and Mission 7, because folder orientation and data ingress both depend on startup understanding.

---

### Mission 2: The Folder Mental Map
**Tier:** Junior  
**Time Estimate:** 40 minutes  
**Goal:** Build a reliable map from folder name to responsibility.

**The Concept:**  
Think of folders as departments in a company: operations (`app`), product squads (`features`), platform (`lib`), QA lab (`testing`).

**Design Intent Before You Read the Code:**  
Avoid reading random files. Build the map first so future debugging has coordinates.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/app/index.tsx:1-10`
- `apps/react-vite/src/features/discussions/api/get-discussion.ts:1-35`
- `apps/react-vite/src/lib/api-client.ts:1-42`
- `apps/react-vite/src/testing/test-utils.tsx:57-94`

```tsx
export const App = () => {
  return (
    <AppProvider> {/* global runtime and shared policies */}
      <AppRouter /> {/* route tree and page-level composition */}
    </AppProvider>
  );
};
```

```ts
export const getDiscussionQueryOptions = (discussionId: string) => {
  return queryOptions({
    queryKey: ['discussions', discussionId], // feature-level contract
    queryFn: () => getDiscussion({ discussionId }),
  });
};
```

**The Aha Moment:**  
Most confusion comes from crossing responsibilities, not from hard code.

**Socratic Checkpoint:**  
1. Which folder owns route composition?  
2. Which folder owns HTTP transport policy?  
3. Which folder owns domain-specific API hooks?  
4. Where do test-only bootstrapping helpers live?  
5. Why is this split helpful during bug triage?

How to self-grade:
- Correctly maps each folder to one primary responsibility.
- Can cite at least one file proving each mapping.

**Connects To:**  
Mission 3 and Mission 8 because contracts and navigation both sit on top of this map.

---

### Mission 3: TypeScript Is a Contract
**Tier:** Junior  
**Time Estimate:** 45 minutes  
**Goal:** Trace how runtime schemas and static types reinforce each other.

**The Concept:**  
Contracts are shipping labels: if the label is wrong, the package arrives but causes damage later.

**Design Intent Before You Read the Code:**  
Find where types are declared, inferred, and enforced at runtime.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/types/api.ts:20-50`
- `apps/react-vite/src/features/discussions/api/create-discussion.ts:10-23`
- `apps/react-vite/src/lib/react-query.ts:12-26`

```ts
export const createDiscussionInputSchema = z.object({
  title: z.string().min(1, 'Required'),
  body: z.string().min(1, 'Required'),
});

export type CreateDiscussionInput = z.infer<typeof createDiscussionInputSchema>;
// one source of truth: runtime validation + compile-time type
```

```ts
export type MutationConfig<
  MutationFnType extends (...args: any) => Promise<any>,
> = UseMutationOptions<
  ApiFnReturnType<MutationFnType>,
  Error,
  Parameters<MutationFnType>[0]
>;
// mutation config auto-derives from mutation function signature
```

**The Aha Moment:**  
Good TS contracts reduce both runtime bugs and code review ambiguity.

**Socratic Checkpoint:**  
1. What does Zod validate that TS alone cannot?  
2. Why use `z.infer` instead of duplicating interfaces?  
3. Where is entity shape centralized?  
4. How does `MutationConfig` reduce repeated typing?  
5. What kind of bug appears if query return shape drifts?

How to self-grade:
- Distinguishes compile-time vs runtime guarantees.
- Explains at least one inferred type path end-to-end.

**Connects To:**  
Mission 6 and Mission 12, because typed props and API contracts extend this concept.

---

### Mission 4: Your First React Component
**Tier:** Junior  
**Time Estimate:** 30 minutes  
**Goal:** Read a simple reusable component and identify its extension points.

**The Concept:**  
Reusable UI primitives are like standardized connectors: they let teams plug features in quickly.

**Design Intent Before You Read the Code:**  
Look for composition patterns (`asChild`, props intersection), not just CSS classes.

**Find It In The Code:**  
Open `apps/react-vite/src/components/ui/button/button.tsx:39-76`.

```tsx
export type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean; // can render as Slot for composition
    isLoading?: boolean; // consistent async UX
    icon?: React.ReactNode;
  };

const Comp = asChild ? Slot : 'button';
// same behavior, customizable host element
```

**The Aha Moment:**  
Simple components are often the highest-leverage architecture decisions.

**Socratic Checkpoint:**  
1. Why combine native button props with variant props?  
2. What problem does `asChild` solve?  
3. Why render spinner inside button instead of callers doing it manually?  
4. Which prop controls icon behavior during loading?  
5. What accessibility behavior comes “for free” from native button attrs?

How to self-grade:
- Mentions composition and type-safety value, not only styling.
- Identifies one misuse risk prevented by this design.

**Connects To:**  
Mission 6 and Mission 16, where component contracts and composition scale up.

---

### Mission 5: Your First Node.js Route
**Tier:** Junior  
**Time Estimate:** 35 minutes  
**Goal:** Understand how backend-like routes are represented in this repo.

**The Concept:**  
A route is a border checkpoint: authenticate, validate, fetch/mutate, respond.

**Design Intent Before You Read the Code:**  
This repo uses MSW handlers as controllers. Don’t hunt for Express controllers first.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/testing/mocks/handlers/teams.ts:8-22`
- `apps/react-vite/mock-server.ts:19-22`

```ts
export const teamsHandlers = [
  http.get(`${env.API_URL}/teams`, async () => {
    await networkDelay(); // latency simulation
    const result = db.team.getAll(); // fake model read
    return HttpResponse.json({ data: result }); // API-like envelope
  }),
];
```

```ts
app.use(createMiddleware(...handlers)); 
// Express attaches handler collection as HTTP middleware chain
```

**The Aha Moment:**  
“Backend route” in this repo means handler semantics first, server framework second.

**Socratic Checkpoint:**  
1. Where does this route run in browser mode?  
2. Where does it run in mock-server mode?  
3. Why is `networkDelay` useful?  
4. What is the shape of the response envelope?  
5. Which file owns route registration?

How to self-grade:
- Correctly describes both execution modes (service worker vs Express middleware).
- Mentions `handlers/index.ts` aggregation role.

**Connects To:**  
Mission 11 and Mission 13, where side effects and middleware chain are deeper.

---

### Mission 6: Props Are a Typed Contract
**Tier:** Junior  
**Time Estimate:** 30 minutes  
**Goal:** Identify how prop typing encodes business assumptions.

**The Concept:**  
Props are small APIs. Bad props make every caller guess.

**Design Intent Before You Read the Code:**  
Focus on how component input types prevent misuse before runtime.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/comments/components/comments.tsx:4-17`
- `apps/react-vite/src/features/comments/components/create-comment.tsx:12-14`

```tsx
type CommentsProps = {
  discussionId: string; // explicit parent contract
};

export const Comments = ({ discussionId }: CommentsProps) => {
  return (
    <div>
      <CreateComment discussionId={discussionId} /> {/* shared context propagation */}
      <CommentsList discussionId={discussionId} />
    </div>
  );
};
```

**The Aha Moment:**  
Typed props are the fastest path to safe composition in feature trees.

**Socratic Checkpoint:**  
1. Why pass `discussionId` instead of reading it globally?  
2. What bug appears if prop becomes optional?  
3. Why is this better than re-parsing URL in every child?  
4. Which child behaviors depend on this prop?  
5. How would you test prop contract breakage?

How to self-grade:
- Connects prop typing to data fetching and mutation behavior.
- Names at least one failure mode from weak prop contracts.

**Connects To:**  
Mission 10 and Mission 14, where hooks and full traces depend on clean contracts.

---

### Mission 7: Following Data Into the App
**Tier:** Junior  
**Time Estimate:** 45 minutes  
**Goal:** Track one request from component to handler response.

**The Concept:**  
Data flow tracing is like package tracking: source, transit points, destination.

**Design Intent Before You Read the Code:**  
Pick one endpoint and follow it through every layer.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/features/discussions/api/get-discussion.ts:7-19`
- `apps/react-vite/src/lib/api-client.ts:16-24`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts:81-124`

```ts
export const getDiscussion = ({ discussionId }: { discussionId: string }) => {
  return api.get(`/discussions/${discussionId}`); // feature fetcher -> shared client
};
```

```ts
api.interceptors.response.use(
  (response) => response.data, // unwrap transport envelope
  (error) => Promise.reject(error),
);
```

```ts
const discussion = db.discussion.findFirst({ where: { id: { equals: discussionId } } });
const result = { ...discussion, author: author ? sanitizeUser(author) : {} };
return HttpResponse.json({ data: result }); // payload shape consumed by UI query
```

**The Aha Moment:**  
Most bugs are mismatches between two adjacent flow steps.

**Socratic Checkpoint:**  
1. Where does route param become HTTP path?  
2. Where is HTTP response transformed?  
3. Where is author data joined into discussion response?  
4. What happens if handler returns raw entity instead of `{data: ...}`?  
5. Which component assumes `data` wrapper exists?

How to self-grade:
- Shows complete path with at least three files.
- Identifies one contract mismatch risk.

**Connects To:**  
Mission 12 and Mission 14 for full API map and deep feature trace.

---

### Mission 8: Navigation Is the App's Skeleton
**Tier:** Junior  
**Time Estimate:** 45 minutes  
**Goal:** Explain how route config, protected routes, and layout navigation work together.

**The Concept:**  
Navigation is a skeleton: if joints are wrong, all movement feels broken.

**Design Intent Before You Read the Code:**  
Track route tree plus nav UI construction from shared path constants.

**Find It In The Code:**  
Open:
- `apps/react-vite/src/config/paths.ts:1-46`
- `apps/react-vite/src/app/router.tsx:24-80`
- `apps/react-vite/src/components/layouts/dashboard-layout.tsx:85-124`

```tsx
{
  path: paths.app.root.path,
  element: (
    <ProtectedRoute>
      <AppRoot /> {/* authenticated shell */}
    </ProtectedRoute>
  ),
  children: [
    { path: paths.app.discussions.path, lazy: () => import('./routes/app/discussions/discussions').then(convert(queryClient)) },
  ],
}
```

```tsx
const navigation = [
  { name: 'Dashboard', to: paths.app.dashboard.getHref(), icon: Home },
  { name: 'Discussions', to: paths.app.discussions.getHref(), icon: Folder },
  checkAccess({ allowedRoles: [ROLES.ADMIN] }) && { name: 'Users', to: paths.app.users.getHref(), icon: Users },
].filter(Boolean) as SideNavigationItem[];
```

**The Aha Moment:**  
Path constants + route tree + role-aware menu produce predictable navigation behavior.

**Socratic Checkpoint:**  
1. Where is login redirection enforced for protected routes?  
2. Why is discussions `NavLink` configured differently with `end`?  
3. How does role-based menu filtering work?  
4. Which constants power both links and route matching?  
5. What is one common bug when constants and route config drift?

How to self-grade:
- Can narrate from `paths.ts` through router to layout menu.
- Includes one concrete drift scenario and detection strategy.

**Connects To:**  
Mission 9 and Mission 14, where state ownership and end-to-end traces rely on this route skeleton.

