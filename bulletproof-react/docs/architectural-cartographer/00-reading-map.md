# 00 Reading Map

## Mental Model In One Paragraph

`react-vite` is a role-aware collaboration app where routes prefetch data with React Query, UI modules consume typed feature APIs, and requests are routed into MSW handlers that act as a mock backend over an in-memory/file/localStorage DB; `nextjs-app` and `nextjs-pages` replicate the same domain modules with different routing shells (Ref: `apps/react-vite/src/app/router.tsx:24-80`, `apps/react-vite/src/features/discussions/api/get-discussion.ts:7-35`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:18-222`, `apps/react-vite/src/testing/mocks/db.ts:45-97`, `apps/nextjs-app/src/app/layout.tsx:18-42`, `apps/nextjs-pages/src/pages/_app.tsx:18-21`).

## Top 10 Files To Read In Order

1. `apps/react-vite/src/main.tsx`
   - Why now: it defines startup sequence and mock boot gating.
   - Understand before: `VITE_APP_*` env parsing.
   - Explain after: why `enableMocking()` runs before React render (Ref: `apps/react-vite/src/main.tsx:11-17`, `apps/react-vite/src/testing/mocks/index.ts:3-9`).
2. `apps/react-vite/src/config/env.ts`
   - Why now: runtime behavior depends on parsed env contract.
   - Understand before: Vite `import.meta.env`.
   - Explain after: required vars and defaults (Ref: `apps/react-vite/src/config/env.ts:4-13`, `apps/react-vite/src/config/env.ts:25-39`).
3. `apps/react-vite/src/app/provider.tsx`
   - Why now: global providers, error boundary, auth loader.
   - Understand before: React Query client basics.
   - Explain after: how global UX loading and error policies are enforced (Ref: `apps/react-vite/src/app/provider.tsx:17-50`).
4. `apps/react-vite/src/app/router.tsx`
   - Why now: routing tree + data loader adaptation.
   - Understand before: route modules pattern.
   - Explain after: lazy route conversion with `clientLoader/clientAction` (Ref: `apps/react-vite/src/app/router.tsx:14-21`, `apps/react-vite/src/app/router.tsx:24-88`).
5. `apps/react-vite/src/lib/api-client.ts`
   - Why now: all feature APIs share this behavior.
   - Understand before: Axios interceptors.
   - Explain after: request/response normalization and 401 redirect behavior (Ref: `apps/react-vite/src/lib/api-client.ts:7-42`).
6. `apps/react-vite/src/lib/auth.tsx`
   - Why now: auth hooks, schemas, and route guard.
   - Understand before: React Query Auth adapter concept.
   - Explain after: identity acquisition and `ProtectedRoute` logic (Ref: `apps/react-vite/src/lib/auth.tsx:13-17`, `apps/react-vite/src/lib/auth.tsx:62-89`).
7. `apps/react-vite/src/features/discussions/api/get-discussions.ts`
   - Why now: canonical feature API query pattern.
   - Understand before: query keys and query options.
   - Explain after: how paging enters key shape and fetcher params (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:20-41`).
8. `apps/react-vite/src/app/routes/app/discussions/discussion.tsx`
   - Why now: full route-level prefetch + view + nested comments.
   - Understand before: query option builders.
   - Explain after: parallel prefetch and render fallback behavior (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:15-36`, `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:45-70`).
9. `apps/react-vite/src/testing/mocks/handlers/discussions.ts`
   - Why now: backend semantics for auth, RBAC, pagination, CRUD.
   - Understand before: MSW handler API.
   - Explain after: team scoping and admin-only mutation enforcement (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:32-51`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:142-149`).
10. `apps/react-vite/src/testing/mocks/db.ts`
    - Why now: persistence mechanics and environment-dependent storage.
    - Understand before: mock architecture assumptions.
    - Explain after: Node vs browser persistence model and initialization path (Ref: `apps/react-vite/src/testing/mocks/db.ts:45-67`, `apps/react-vite/src/testing/mocks/db.ts:87-97`).

## Three Most Important Data Flows

1. App bootstrap flow
   - `main.tsx` calls `enableMocking()` then mounts `<App />` (Ref: `apps/react-vite/src/main.tsx:11-17`).
   - `enableMocking()` initializes DB and starts worker when enabled (Ref: `apps/react-vite/src/testing/mocks/index.ts:4-9`).
2. Protected route + identity flow
   - `AppProvider` renders `AuthLoader`; auth hook calls `/auth/me` (Ref: `apps/react-vite/src/app/provider.tsx:38-46`, `apps/react-vite/src/lib/auth.tsx:13-17`, `apps/react-vite/src/lib/auth.tsx:75-76`).
   - `ProtectedRoute` redirects unauthenticated users to login with `redirectTo` (Ref: `apps/react-vite/src/lib/auth.tsx:82-85`).
3. Discussion detail + comments flow
   - Route loader prefetches discussion and paginated comments in parallel (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:20-31`).
   - API layer hits `/discussions/:id` and `/comments` endpoints (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:12`, `apps/react-vite/src/features/comments/api/get-comments.ts:14-19`).
   - Handler layer enforces auth and returns enriched author objects (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:87-123`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:46-67`).

## Pre-Reading Checklist (10 Questions)

1. Can I explain the difference between `AppProvider` and `AppRouter`? (Ref: `apps/react-vite/src/app/index.tsx:4-9`)
2. Do I know where env vars are validated and transformed? (Ref: `apps/react-vite/src/config/env.ts:4-39`)
3. Do I know what makes a route “protected” in this app? (Ref: `apps/react-vite/src/app/router.tsx:41-44`, `apps/react-vite/src/lib/auth.tsx:78-89`)
4. Can I trace one query key from UI to handler response? (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:17`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:82-123`)
5. Do I know where role checks happen in UI and handlers? (Ref: `apps/react-vite/src/lib/authorization.tsx:14-26`, `apps/react-vite/src/testing/mocks/utils.ts:106-109`)
6. Can I explain how pagination state is passed and consumed? (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:14-16`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:30-51`)
7. Do I know what state is in Zustand vs React Query? (Ref: `apps/react-vite/src/components/ui/notifications/notifications-store.ts:17-32`, `apps/react-vite/src/features/discussions/api/get-discussions.ts:34-42`)
8. Can I explain why tests use MSW server mode? (Ref: `apps/react-vite/src/testing/mocks/server.ts:1-5`, `apps/react-vite/src/testing/setup-tests.ts:8-10`)
9. Do I know where fake persistence lives in browser and Node? (Ref: `apps/react-vite/src/testing/mocks/db.ts:47-57`, `apps/react-vite/src/testing/mocks/db.ts:64-77`)
10. Can I explain why Next.js variants still use same feature modules? (Ref: `apps/nextjs-app/src/features/discussions/api/get-discussion.ts:1-35`, `apps/nextjs-pages/src/features/discussions/api/get-discussion.ts:1-35`)

## Red Flags Checklist (Code Review + Architecture Reading)

1. Query key drift between fetchers and invalidators (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:24`, `apps/react-vite/src/features/discussions/api/delete-discussion.ts:29-31`).
2. Client-only authorization without handler enforcement (verify both layers) (Ref: `apps/react-vite/src/features/discussions/components/delete-discussion.tsx:28-52`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:204-214`).
3. Missing team scoping in list/read endpoints (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:44-48`, `apps/react-vite/src/testing/mocks/handlers/users.ts:31-35`).
4. Mutations that do not invalidate/refetch affected queries (Ref: `apps/react-vite/src/features/comments/api/create-comment.ts:39-43`).
5. Inconsistent response shape (`{data}` vs raw entity) causing UI assumptions to break (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:65-73`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:149`).
6. Silent failure pathways after auth errors (redirect side effects) (Ref: `apps/react-vite/src/lib/api-client.ts:33-38`).
7. Over-reliance on mock token encoding as if it were real JWT security (Ref: `apps/react-vite/src/testing/mocks/utils.ts:70-71`).
8. Loader/component duplicate fetches without benefit (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:20-31`, `apps/react-vite/src/features/discussions/components/discussion-view.tsx:9-11`).
9. Unhandled null user paths in policy checks (Ref: `apps/react-vite/src/lib/authorization.tsx:31-33`).
10. Feature boundary violations masked by relative imports (lint rule should catch) (Ref: `apps/react-vite/.eslintrc.cjs:46-98`).

