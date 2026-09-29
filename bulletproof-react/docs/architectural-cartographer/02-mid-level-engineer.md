# 02 Mid-Level Engineer

## Full-Stack Architecture Diagram (Structured Text)

```text
[Browser / React UI]
  -> Route modules call feature hooks
     (apps/react-vite/src/app/routes/**, apps/react-vite/src/features/**)
  -> Hooks call api-client (Axios instance)
     (apps/react-vite/src/lib/api-client.ts)
  -> HTTP request goes to:
     A) MSW Service Worker in browser (mock mode)
        (apps/react-vite/src/testing/mocks/browser.ts)
     B) Express + MSW middleware process (e2e/local API mode)
        (apps/react-vite/mock-server.ts)
  -> Handler executes auth + role checks + model ops
     (apps/react-vite/src/testing/mocks/handlers/*.ts + utils.ts)
  -> Data layer uses @mswjs/data models
     (apps/react-vite/src/testing/mocks/db.ts)
  -> Response returned to UI -> React Query cache update/invalidation
     (apps/react-vite/src/features/**/api/*.ts)
  -> Components re-render with updated data
     (apps/react-vite/src/features/**/components/*.tsx)
```

## TypeScript System Deep Dive

### 1) Domain Contract Layer

- `Entity<T>` composes business fields with shared metadata (`id`, `createdAt`) to keep UI tables/components generic (Ref: `apps/react-vite/src/types/api.ts:5-13`).
- Domain types (`User`, `Discussion`, `Comment`) unify API and UI expectations across features (Ref: `apps/react-vite/src/types/api.ts:20-50`).

### 2) Runtime Validation + Inferred Types

- Zod schemas define runtime constraints and generate static types for input payloads, preventing hook/component divergence (Ref: `apps/react-vite/src/features/comments/api/create-comment.ts:10-16`, `apps/react-vite/src/lib/auth.tsx:23-55`).

### 3) Query/Mutation Helper Types

- `ApiFnReturnType`, `QueryConfig`, and `MutationConfig` abstract repeated generic wiring, reducing local type ceremony while preserving safety (Ref: `apps/react-vite/src/lib/react-query.ts:12-26`).

### 4) Generic UI Contracts

- `Table<Entry extends BaseEntity>` enforces base identity fields and allows domain-specific columns, keeping table internals reusable (Ref: `apps/react-vite/src/components/ui/table/table.tsx:126-177`).

## State Management Deep Dive (Notifications + Query Cache)

### Why This Is Important

This codebase deliberately separates:
- server-state cache in React Query
- local UI/global client state in Zustand

(Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:34-42`, `apps/react-vite/src/components/ui/notifications/notifications-store.ts:17-32`)

### Example: “Comment Created” Experience

1. Component calls mutation and emits notification on success (Ref: `apps/react-vite/src/features/comments/components/create-comment.tsx:18-27`).
2. Mutation invalidates comments query key for current discussion (Ref: `apps/react-vite/src/features/comments/api/create-comment.ts:39-43`).
3. Notifications store appends toast object in Zustand (Ref: `apps/react-vite/src/components/ui/notifications/notifications-store.ts:19-25`).
4. `Notifications` component renders toasts from store (Ref: `apps/react-vite/src/components/ui/notifications/notifications.tsx:4-19`).

## API Contract Map

## Auth

- `POST /auth/register` -> returns `{jwt, user}`, sets cookie header (Ref: `apps/react-vite/src/testing/mocks/handlers/auth.ts:30-104`).
- `POST /auth/login` -> returns `{jwt, user}` (Ref: `apps/react-vite/src/testing/mocks/handlers/auth.ts:112-127`).
- `GET /auth/me` -> returns `{data: user}` (Ref: `apps/react-vite/src/testing/mocks/handlers/auth.ts:152-158`).

## Discussions

- `GET /discussions?page=n` -> `{data: Discussion[], meta}` team-scoped (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:19-73`).
- `GET /discussions/:discussionId` -> `{data: Discussion}` (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:81-124`).
- `POST/PATCH/DELETE /discussions/:id` -> admin-gated (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:133-221`).

## Comments

- `GET /comments?discussionId=x&page=n` -> paginated comments (Ref: `apps/react-vite/src/testing/mocks/handlers/comments.ts:14-67`).
- `POST /comments` -> create with authenticated `authorId` (Ref: `apps/react-vite/src/testing/mocks/handlers/comments.ts:75-90`).
- `DELETE /comments/:commentId` -> owner-or-admin policy (Ref: `apps/react-vite/src/testing/mocks/handlers/comments.ts:114-119`).

## Component Interaction Map

```text
DiscussionsRoute
  -> CreateDiscussion
     -> useCreateDiscussion mutation
  -> DiscussionsList
     -> useDiscussions query
     -> prefetch discussion + comments on hover
         (getDiscussionQueryOptions + getInfiniteCommentsQueryOptions)
```

Ref: `apps/react-vite/src/app/routes/app/discussions/discussions.tsx:25-41`, `apps/react-vite/src/features/discussions/components/discussions-list.tsx:62-67`

```text
DiscussionRoute
  -> DiscussionView
     -> useDiscussion query
     -> UpdateDiscussion mutation path
  -> Comments
     -> CreateComment
     -> CommentsList
        -> useInfiniteComments
        -> DeleteComment
```

Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:57-70`, `apps/react-vite/src/features/discussions/components/discussion-view.tsx:35-44`, `apps/react-vite/src/features/comments/components/comments.tsx:8-16`, `apps/react-vite/src/features/comments/components/comments-list.tsx:19-93`

## Full-Stack Feature Trace (Discussion Detail Page)

## 1) UI

- User navigates to `/app/discussions/:discussionId`; route component mounts (Ref: `apps/react-vite/src/app/router.tsx:55-60`).

## 2) State/Data Fetching

- `clientLoader` prefetches discussion + infinite comments in parallel (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:20-31`).
- `DiscussionView` executes `useDiscussion` query keyed by discussionId (Ref: `apps/react-vite/src/features/discussions/components/discussion-view.tsx:9-11`).

## 3) HTTP

- `api.get('/discussions/:id')` and `api.get('/comments')` called via feature fetchers (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:12`, `apps/react-vite/src/features/comments/api/get-comments.ts:14-19`).

## 4) Middleware / Interceptors

- Request interceptor sets `Accept` and `withCredentials` (Ref: `apps/react-vite/src/lib/api-client.ts:7-13`).

## 5) Service / Business Logic (Handler Layer)

- `requireAuth` verifies cookie token and user presence (Ref: `apps/react-vite/src/testing/mocks/utils.ts:80-103`).
- Discussion handler enforces team match + author enrichment (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:92-123`).
- Comment handler paginates by discussionId and enriches author (Ref: `apps/react-vite/src/testing/mocks/handlers/comments.ts:36-67`).

## 6) Data Layer

- Handlers query `db.discussion`, `db.comment`, `db.user` models and return computed payloads (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:92-117`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:36-58`).

## 7) Response

- Handler response wrapped as `HttpResponse.json({data,meta})` (Ref: `apps/react-vite/src/testing/mocks/handlers/comments.ts:59-67`).
- Axios response interceptor unwraps `response.data` so hooks consume direct payload object (Ref: `apps/react-vite/src/lib/api-client.ts:21-24`).

## 8) UI Update

- Query results populate `DiscussionView` and `CommentsList`; loading boundaries drop when queries resolve (Ref: `apps/react-vite/src/features/discussions/components/discussion-view.tsx:13-24`, `apps/react-vite/src/features/comments/components/comments-list.tsx:23-33`).

## Diff Reading Exercise (Hypothetical But Realistic)

Change request: “Only admins can create comments.”

What to inspect in order:

1. UI guard should be introduced in `CreateComment` or its parent (Ref baseline: `apps/react-vite/src/features/comments/components/create-comment.tsx:30-75`).
2. Handler must enforce role, not only UI (Ref baseline: `apps/react-vite/src/testing/mocks/handlers/comments.ts:75-90`).
3. Policy utilities may need extension for reusable rule checks (Ref baseline: `apps/react-vite/src/lib/authorization.tsx:14-26`).
4. Tests for user role should fail until updated (Ref baseline: `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx:77-145`).

Critical review questions:

1. Did the change add handler-level authorization, or only hide UI?
2. Did query invalidation behavior remain intact after mutation path changes?
3. Are role-based tests covering both positive and negative cases?

## Non-Obvious Patterns And Why They Were Chosen

- Route module conversion pattern (`clientLoader/clientAction`) keeps data prefetch logic colocated with route files while still injecting shared query client at router composition time (Ref: `apps/react-vite/src/app/router.tsx:14-21`).
- Domain features use “API file + component file” pairing, making side effects explicit and testable (Ref: `apps/react-vite/src/features/discussions/api/create-discussion.ts:17-45`, `apps/react-vite/src/features/discussions/components/create-discussion.tsx:13-73`).
- The same domain modules are duplicated across three app shells to teach portability across routing frameworks (Ref: `apps/nextjs-app/src/features/comments/api/get-comments.ts:1-47`, `apps/nextjs-pages/src/features/comments/api/get-comments.ts:1-47`).

## Mid-Level Socratic Checkpoint

### Questions

1. Why is loader-based prefetch useful even when components also call queries?
2. Which cache keys are at highest risk of inconsistency, and why?
3. Where are the strongest and weakest authorization guarantees?
4. How would you detect an ownership-policy bug in comment deletion without reading backend code first?
5. What coupling exists between Axios response interceptor shape and feature fetcher return types?
6. If you swapped MSW DB with real API, which layers could remain unchanged?
7. What exact files would you change to add “pinned discussion” support?
8. Where would you add test coverage first for that feature and why?

### How To Self-Grade

Strong answers include:

- Explicit route/component/hook/handler file citations.
- A separation between business-policy concerns and rendering concerns.
- Correct identification of response shape dependencies (`{data}`, `{meta}`, raw entity returns).
- A migration strategy that preserves feature APIs while changing infrastructure layer.

