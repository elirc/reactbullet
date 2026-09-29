# Journal

## First-Pass Mental Model

This repo is best understood as one domain implemented three times over different routing/runtime stacks: Vite + React Router, Next App Router, and Next Pages Router (Ref: `apps/react-vite/src/app/router.tsx:24-80`, `apps/nextjs-app/src/app/layout.tsx:18-42`, `apps/nextjs-pages/src/pages/_app.tsx:18-21`).  
The core business domain is team-scoped discussions and comments with role-aware actions (admin vs user), and the runtime data layer is mocked via MSW handlers plus an in-memory/persisted fake DB instead of a production backend+DB (Ref: `apps/react-vite/src/types/api.ts:20-50`, `apps/react-vite/src/lib/authorization.tsx:7-26`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:18-222`, `apps/react-vite/src/testing/mocks/db.ts:39-97`).

## Important Discoveries While Inspecting

- The frontend blocks render until mock infrastructure is ready when mock mode is enabled, so the app startup path includes infrastructure bootstrapping, not just React mount (Ref: `apps/react-vite/src/main.tsx:11-17`, `apps/react-vite/src/testing/mocks/index.ts:3-9`).
- The route layer prefetches data with `clientLoader` + React Query before rendering route components, reducing first-view fetch thrash (Ref: `apps/react-vite/src/app/router.tsx:14-21`, `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:15-36`).
- “Backend” behavior is represented by MSW handlers and optional Express middleware wrapper for E2E/local API simulation (Ref: `apps/react-vite/src/testing/mocks/handlers/index.ts:13-23`, `apps/react-vite/mock-server.ts:10-30`).
- Persistence is file-based (`mocked-db.json`) in Node and `localStorage` in browser, so data durability and concurrency are intentionally simplified for teaching/demo purposes (Ref: `apps/react-vite/src/testing/mocks/db.ts:43-77`).
- Auth relies on a cookie token that is base64-encoded user payload in mock utilities, with explicit comment that real deployments should use secure/HttpOnly controls (Ref: `apps/react-vite/src/testing/mocks/utils.ts:53-103`, `apps/react-vite/src/testing/mocks/handlers/auth.ts:100-102`).

## Why These Files Were Chosen As Teaching Anchors

- `src/main.tsx`, `src/app/provider.tsx`, `src/app/router.tsx` define runtime skeleton and dependency wiring (Ref: `apps/react-vite/src/main.tsx:1-17`, `apps/react-vite/src/app/provider.tsx:17-52`, `apps/react-vite/src/app/router.tsx:24-88`).
- `src/lib/api-client.ts` and `src/lib/auth.tsx` define cross-feature boundaries (HTTP policy, auth identity model) (Ref: `apps/react-vite/src/lib/api-client.ts:16-42`, `apps/react-vite/src/lib/auth.tsx:62-89`).
- `features/discussions`, `features/comments`, and handler files carry the most complete end-to-end feature with CRUD + authorization + pagination (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:7-35`, `apps/react-vite/src/features/comments/api/get-comments.ts:22-47`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:13-131`).
- `testing/test-utils.tsx` shows the intended integration testing posture and fake-user bootstrap strategy (Ref: `apps/react-vite/src/testing/test-utils.tsx:57-94`).

## Where A Junior Might Get Confused

- Why an app called “frontend architecture” has Express in `devDependencies`; answer: Express powers optional mock API server for E2E/dev realism, not production app serving (Ref: `apps/react-vite/package.json:12-13`, `apps/react-vite/package.json:93`, `apps/react-vite/mock-server.ts:1-4`).
- Why route files have loaders and components; answer: data preloading and UI rendering are split, then stitched by router conversion utility (Ref: `apps/react-vite/src/app/router.tsx:14-21`, `apps/react-vite/src/app/routes/app/discussions/discussions.tsx:10-23`).
- Why `useUser` exists without custom Redux/context code; answer: `react-query-auth` config returns typed auth hooks via adapter layer (Ref: `apps/react-vite/src/lib/auth.tsx:1`, `apps/react-vite/src/lib/auth.tsx:75-76`).

## Where A Mid-Level Engineer Should Slow Down

- Query invalidation and cache key composition (`['discussions', { page }]` vs `['discussions', id]`) because collisions and stale views are common failure modes (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:24`, `apps/react-vite/src/features/discussions/api/get-discussion.ts:17`, `apps/react-vite/src/features/discussions/api/update-discussion.ts:40-42`).
- Auth and authorization distinction: auth identity (`requireAuth`) and policy enforcement (`requireAdmin`, client-side `Authorization`) occur in multiple layers and can drift (Ref: `apps/react-vite/src/testing/mocks/utils.ts:80-110`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:142`, `apps/react-vite/src/lib/authorization.tsx:63-82`).

## Where A Senior Engineer Should Be Skeptical

- Security realism: cookie token encoding is not signed JWT; this is demo-only and not production-safe (Ref: `apps/react-vite/src/testing/mocks/utils.ts:68-71`, `apps/react-vite/src/testing/mocks/handlers/auth.ts:100-102`).
- Data race/multi-user realism: local file/localStorage persistence lacks transactionality and cross-process consistency (Ref: `apps/react-vite/src/testing/mocks/db.ts:69-85`).
- Authorization enforcement symmetry: some UI actions are hidden client-side, but true enforcement should remain server-side; here enforcement exists in MSW handlers, but it is still a mock boundary (Ref: `apps/react-vite/src/features/discussions/components/create-discussion.tsx:27-71`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:142-149`).

## How To Use Checkpoints For Self-Study

1. Answer from memory first.
2. Re-open only the cited files.
3. Rewrite your answer as if explaining to a teammate under time pressure.
4. Self-grade against rubric criteria.
5. Record one misconception and one improved heuristic.

