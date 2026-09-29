# 01 Junior Engineer

## Local Setup And Run

### What Exists

- Root `prepare` installs dependencies in all three apps by chaining `yarn` commands (Ref: `package.json:7`).
- Each app has a local README with setup instructions and env file copy steps (Ref: `apps/react-vite/README.md:10-18`).
- React Vite env contract requires `VITE_APP_API_URL` and optionally mock flags/ports (Ref: `apps/react-vite/src/config/env.ts:4-13`, `apps/react-vite/.env.example:1-2`, `apps/react-vite/.env.example-e2e:1-4`).

### Practical Setup (React Vite, Recommended First Path)

1. `cd apps/react-vite`
2. `cp .env.example .env`
3. If you want local mock API server for E2E-like behavior, ensure `.env` includes:
   - `VITE_APP_API_URL=http://localhost:8080/api`
   - `VITE_APP_ENABLE_API_MOCKING=false`
   - `VITE_APP_MOCK_API_PORT=8080`
   - `VITE_APP_URL=http://localhost:3000`
   (Ref: `apps/react-vite/.env.example-e2e:1-4`)
4. `yarn install`
5. `yarn dev` (Vite on port 3000 by config) (Ref: `apps/react-vite/package.json:6`, `apps/react-vite/vite.config.ts:11-16`).

### Setup Gaps You Should Know

- There is no production backend setup script or migration workflow in this repo; API behavior is mocked with MSW handlers and fake DB (Ref: `apps/react-vite/src/testing/mocks/handlers/index.ts:13-23`, `apps/react-vite/src/testing/mocks/db.ts:39-97`).
- There is no ORM schema/migration layer (Prisma/Drizzle/etc.) checked into this repo; persistence is mock-only (Ref: `apps/react-vite/src/testing/mocks/db.ts:1-39`).

## Folder-By-Folder Orientation

## Frontend Top-Level (React Vite `src/`)

- `app/`: composition root, router, route modules (Ref: `apps/react-vite/src/app/index.tsx:1-10`, `apps/react-vite/src/app/router.tsx:24-88`).
- `components/`: shared UI primitives and layouts (Ref: `apps/react-vite/src/components/ui/button/button.tsx:9-76`, `apps/react-vite/src/components/layouts/dashboard-layout.tsx:79-205`).
- `features/`: domain modules (`auth`, `discussions`, `comments`, `users`, `teams`) (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:1-35`, `apps/react-vite/src/features/comments/components/comments-list.tsx:19-93`).
- `lib/`: cross-feature infrastructure (auth adapter, API client, query config) (Ref: `apps/react-vite/src/lib/auth.tsx:62-76`, `apps/react-vite/src/lib/api-client.ts:16-42`, `apps/react-vite/src/lib/react-query.ts:3-10`).
- `testing/`: app-level test scaffolding + MSW mocks (Ref: `apps/react-vite/src/testing/test-utils.tsx:57-94`, `apps/react-vite/src/testing/mocks/server.ts:1-5`).

## Backend-Like Top-Level Surfaces In This Repo

This repo does not include a dedicated production backend service folder. The backend contract is simulated by these surfaces:

- `apps/react-vite/mock-server.ts`: Express wrapper around MSW handlers for local API server mode (Ref: `apps/react-vite/mock-server.ts:10-30`).
- `apps/react-vite/src/testing/mocks/handlers/`: route/controller equivalents (`auth`, `discussions`, `comments`, `users`, `teams`) (Ref: `apps/react-vite/src/testing/mocks/handlers/index.ts:7-19`).
- `apps/react-vite/src/testing/mocks/db.ts`: fake model layer and persistence adapter (Ref: `apps/react-vite/src/testing/mocks/db.ts:4-39`, `apps/react-vite/src/testing/mocks/db.ts:69-97`).

## One Level Deeper: 3 Most Important Frontend Folders

### 1) `src/app/`

- Owns runtime composition (`AppProvider` + `AppRouter`) (Ref: `apps/react-vite/src/app/index.tsx:4-9`).
- Defines protected + lazy route tree and loader conversion (Ref: `apps/react-vite/src/app/router.tsx:14-80`).

### 2) `src/features/discussions/`

- API hooks define query/mutation contracts with typed inputs and invalidation rules (Ref: `apps/react-vite/src/features/discussions/api/create-discussion.ts:10-46`, `apps/react-vite/src/features/discussions/api/update-discussion.ts:10-47`).
- Components express feature UX lifecycle (list/detail/create/update/delete) (Ref: `apps/react-vite/src/features/discussions/components/discussions-list.tsx:42-92`, `apps/react-vite/src/features/discussions/components/discussion-view.tsx:25-50`).

### 3) `src/lib/`

- Auth identity model and route guard (Ref: `apps/react-vite/src/lib/auth.tsx:62-89`).
- HTTP policy (cookies, error notifications, 401 redirects) (Ref: `apps/react-vite/src/lib/api-client.ts:7-42`).

## One Level Deeper: 3 Most Important Backend-Like Folders

### 1) `src/testing/mocks/handlers/`

- Each file is effectively a controller group for one resource (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:18-222`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:13-131`).

### 2) `src/testing/mocks/`

- `utils.ts` handles “auth middleware” concerns like token decode and role checks (Ref: `apps/react-vite/src/testing/mocks/utils.ts:80-110`).
- `server.ts` and `browser.ts` adapt handlers to test-node and browser-worker runtimes (Ref: `apps/react-vite/src/testing/mocks/server.ts:1-5`, `apps/react-vite/src/testing/mocks/browser.ts:1-5`).

### 3) `mock-server.ts`

- Wraps handlers in Express with CORS and request logging for local API process mode (Ref: `apps/react-vite/mock-server.ts:12-21`, `apps/react-vite/mock-server.ts:23-30`).

## Entry Point Walkthroughs (Annotated)

### Frontend Entry Point (`main.tsx`)

```tsx
import * as React from 'react';
import { createRoot } from 'react-dom/client';

import './index.css';
import { App } from './app';
import { enableMocking } from './testing/mocks';

const root = document.getElementById('root');
if (!root) throw new Error('No root element found'); // hard stop if mount target is missing

enableMocking().then(() => {
  // app render is intentionally gated behind mock bootstrap
  createRoot(root).render(
    <React.StrictMode>
      <App /> {/* App composes provider + router */}
    </React.StrictMode>,
  );
});
```

Source: `apps/react-vite/src/main.tsx:1-17`

### Backend-Like Entry Point (`mock-server.ts`)

```ts
const app = express();

app.use(
  cors({
    origin: env.APP_URL, // frontend origin allowed by CORS
    credentials: true,   // cookie auth flow requires credentials
  }),
);

app.use(express.json()); // parse JSON bodies for handlers
app.use(logger());
app.use(createMiddleware(...handlers)); // mount MSW handlers as HTTP middleware

initializeDb().then(() => {
  // load persisted fake data before serving endpoints
  app.listen(env.APP_MOCK_API_PORT, () => {
    console.log(`Mock API server started at http://localhost:${env.APP_MOCK_API_PORT}`);
  });
});
```

Source: `apps/react-vite/mock-server.ts:10-30`

## TypeScript Orientation (Beginner-Relevant)

1. API domain entities are centrally typed in `types/api.ts` and reused across features (Ref: `apps/react-vite/src/types/api.ts:20-50`).
2. Input validation and runtime schemas use Zod, and inferred types (`z.infer`) become mutation input types (Ref: `apps/react-vite/src/features/discussions/api/create-discussion.ts:10-16`).
3. Generic helper types in `lib/react-query.ts` reduce repetitive query/mutation typing boilerplate (Ref: `apps/react-vite/src/lib/react-query.ts:12-26`).
4. Generic UI components like `Table` enforce entry shape by constraining to `BaseEntity` (Ref: `apps/react-vite/src/components/ui/table/table.tsx:132-143`).

## Three React Component Anatomies (Annotated)

### 1) Simple Component: `Button`

```tsx
export type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean;  // allow composition through Radix Slot
    isLoading?: boolean; // standardized loading UX
    icon?: React.ReactNode;
  };

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ asChild = false, isLoading, icon, children, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button'; // composable primitive
    return (
      <Comp ref={ref} {...props}>
        {isLoading && <Spinner size="sm" className="text-current" />}
        {!isLoading && icon && <span className="mr-2">{icon}</span>}
        <span className="mx-2">{children}</span>
      </Comp>
    );
  },
);
```

Source: `apps/react-vite/src/components/ui/button/button.tsx:39-74`

### 2) Medium Component: `CreateComment`

```tsx
export const CreateComment = ({ discussionId }: { discussionId: string }) => {
  const createCommentMutation = useCreateComment({
    discussionId, // ties invalidation key to current discussion
    mutationConfig: { onSuccess: () => addNotification({ type: 'success', title: 'Comment Created' }) },
  });

  return (
    <FormDrawer isDone={createCommentMutation.isSuccess}>
      <Form
        id="create-comment"
        schema={createCommentInputSchema} // runtime schema + TS inference
        options={{ defaultValues: { body: '', discussionId } }} // hidden contextual payload
        onSubmit={(values) => createCommentMutation.mutate({ data: values })}
      >
        {({ register, formState }) => (
          <Textarea
            label="Body"
            error={formState.errors['body']}
            registration={register('body')}
          />
        )}
      </Form>
    </FormDrawer>
  );
};
```

Source: `apps/react-vite/src/features/comments/components/create-comment.tsx:16-76`

### 3) Complex Component: `DashboardLayout`

```tsx
const navigation = [
  { name: 'Dashboard', to: paths.app.dashboard.getHref(), icon: Home },
  { name: 'Discussions', to: paths.app.discussions.getHref(), icon: Folder },
  checkAccess({ allowedRoles: [ROLES.ADMIN] }) && {
    name: 'Users', to: paths.app.users.getHref(), icon: Users, // role-aware nav item
  },
].filter(Boolean) as SideNavigationItem[];

// ...
<NavLink
  to={item.to}
  end={item.name !== 'Discussions'} // discussions has nested detail path, so keep active on subroutes
>
  <item.icon />
  {item.name}
</NavLink>

// ...
<DropdownMenuItem onClick={() => logout.mutate({})}>
  Sign Out {/* invokes auth mutation, not direct local state clear */}
</DropdownMenuItem>
```

Source: `apps/react-vite/src/components/layouts/dashboard-layout.tsx:85-93`, `apps/react-vite/src/components/layouts/dashboard-layout.tsx:103-124`, `apps/react-vite/src/components/layouts/dashboard-layout.tsx:190-195`

## Three Backend Route/Controller Anatomies (Annotated)

### 1) Simple Route: `GET /teams`

```ts
http.get(`${env.API_URL}/teams`, async () => {
  await networkDelay(); // simulate realistic latency
  const result = db.team.getAll(); // direct model fetch
  return HttpResponse.json({ data: result }); // consistent envelope shape
});
```

Source: `apps/react-vite/src/testing/mocks/handlers/teams.ts:9-15`

### 2) Important Mutation Route: `POST /discussions`

```ts
http.post(`${env.API_URL}/discussions`, async ({ request, cookies }) => {
  const { user, error } = requireAuth(cookies); // auth guard
  if (error) return HttpResponse.json({ message: error }, { status: 401 });

  const data = (await request.json()) as DiscussionBody;
  requireAdmin(user); // authorization guard

  const result = db.discussion.create({
    teamId: user?.teamId,  // enforce team scoping on create
    authorId: user?.id,
    ...data,
  });
  await persistDb('discussion'); // persist mutation side effect
  return HttpResponse.json(result);
});
```

Source: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:133-150`

### 3) Policy Route: `DELETE /comments/:commentId`

```ts
http.delete(`${env.API_URL}/comments/:commentId`, async ({ params, cookies }) => {
  const { user, error } = requireAuth(cookies);
  if (error) return HttpResponse.json({ message: error }, { status: 401 });

  const result = db.comment.delete({
    where: {
      id: { equals: params.commentId as string },
      ...(user?.role === 'USER' && {
        authorId: { equals: user.id }, // users can only delete own comments
      }),
    },
  });

  await persistDb('comment');
  return HttpResponse.json(result);
});
```

Source: `apps/react-vite/src/testing/mocks/handlers/comments.ts:98-123`

## Domain Glossary

### Product Domain Terms

- Team: tenant-like group boundary; users and discussions are team-scoped (Ref: `apps/react-vite/src/types/api.ts:34-37`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:44-48`).
- Discussion: long-form topic created by team members, admin-mutable (Ref: `apps/react-vite/src/types/api.ts:39-44`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:142-171`).
- Comment: discussion reply; delete permissions differ by role/ownership (Ref: `apps/react-vite/src/types/api.ts:46-50`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:114-118`).
- Role (`ADMIN`, `USER`): permission axis used in UI and handlers (Ref: `apps/react-vite/src/types/api.ts:24`, `apps/react-vite/src/lib/authorization.tsx:7-25`).

### Technical Terms In This Codebase

- `clientLoader`: route-level prefetch function used by React Router lazy modules (Ref: `apps/react-vite/src/app/routes/app/discussions/discussions.tsx:10-23`).
- Query Options Builder: function that returns standardized `queryKey`/`queryFn` pairs (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:20-27`).
- Mutation Config: optional callbacks merged into shared mutation wrappers (Ref: `apps/react-vite/src/features/comments/api/create-comment.ts:25-47`).
- Mock DB persistence: environment-aware serialization to file/localStorage (Ref: `apps/react-vite/src/testing/mocks/db.ts:69-77`).

## Junior Socratic Checkpoint

### Questions

1. Why does this app run `enableMocking()` before mounting React, and what breaks if it doesn’t?
2. Where is authentication status retrieved, and where is route-level protection enforced?
3. In discussions list/detail flow, where does pagination enter the system and where is it consumed?
4. Why do some actions disappear for users with `USER` role, and where is enforcement beyond the UI?
5. What is the difference between `useNotifications` state and React Query cache state?
6. How does one mutation (create comment) cause the comments list to refresh?
7. If you had to remove mock persistence safely, which files would you touch first and why?

### How To Self-Grade

Strong answers should include:

- Exact file-path and line-range evidence, not generic descriptions.
- Distinction between identity (`auth`) and permission (`authorization`).
- A complete flow from component -> API hook -> API client -> handler -> DB -> UI refresh.
- Recognition that this repo has no independent production backend/ORM layer and uses mock infrastructure instead.
- At least one risk statement (for example, stale cache key mismatch or client/server authorization drift).

