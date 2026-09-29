# 04 Reference Suite

## Doc 1: Junior Onboarding Guide

### Day 1 Orientation

1. Confirm repo shape: root manages three app implementations (Ref: `package.json:7`).
2. Pick one app to learn deeply first (`apps/react-vite` recommended because routing/data flow is explicit) (Ref: `apps/react-vite/src/app/router.tsx:24-88`).
3. Run setup from app README and verify env contract (Ref: `apps/react-vite/README.md:10-18`, `apps/react-vite/src/config/env.ts:4-39`).

### First Code Path To Trace

1. `main.tsx` -> `AppProvider` -> `AppRouter` (Ref: `apps/react-vite/src/main.tsx:11-17`, `apps/react-vite/src/app/index.tsx:4-9`).
2. Navigate to discussion detail route and follow loader+hook path (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:15-43`).
3. Trace request into handler and fake DB (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:81-124`, `apps/react-vite/src/testing/mocks/db.ts:39-97`).

### What To Learn First

- Query key shape discipline (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:24`, `apps/react-vite/src/features/comments/api/get-comments.ts:24`).
- Role/ownership policy mechanics (Ref: `apps/react-vite/src/lib/authorization.tsx:14-26`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:114-119`).

## Doc 2: Mid-Level Architecture Guide

### System Design Summary

- Feature modules hold domain APIs and UI.
- `lib/` owns cross-cutting concerns.
- Route modules orchestrate loader + render composition.
- Mock handlers provide backend semantics.

Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:1-35`, `apps/react-vite/src/lib/api-client.ts:16-42`, `apps/react-vite/src/app/routes/app/discussions/discussions.tsx:10-44`, `apps/react-vite/src/testing/mocks/handlers/index.ts:13-23`

### Data Flows To Memorize

1. Login/register -> `react-query-auth` -> route guard (Ref: `apps/react-vite/src/lib/auth.tsx:62-89`).
2. Discussion listing -> pagination -> table UI (Ref: `apps/react-vite/src/features/discussions/components/discussions-list.tsx:24-91`).
3. Comment create/delete -> invalidate infinite query -> re-render list (Ref: `apps/react-vite/src/features/comments/api/create-comment.ts:39-43`, `apps/react-vite/src/features/comments/api/delete-comment.ts:26-30`).

### Contract Boundaries

- HTTP response envelope expectations are encoded in feature fetchers.
- Handlers must preserve those expectations.

Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:11-13`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:123-124`

## Doc 3: Senior Ownership Guide

### Technical Debt Register

1. Sequential awaits in “parallel” loader flow (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:23-28`).
2. `any` in router conversion and some test/mock utilities (Ref: `apps/react-vite/src/app/router.tsx:14`, `apps/react-vite/src/testing/test-utils.tsx:19`).
3. Mock token model unsuitable for real security posture (Ref: `apps/react-vite/src/testing/mocks/utils.ts:68-71`).
4. Three-app duplication creates sync overhead (Ref: `apps/react-vite/src/features/users/api/get-users.ts:1-27`, `apps/nextjs-app/src/features/users/api/get-users.ts:1-27`, `apps/nextjs-pages/src/features/users/api/get-users.ts:1-27`).

### Upgrade Path

1. Introduce shared internal package for domain feature APIs/components.
2. Introduce typed route-module interface.
3. Separate demo-auth path from production-auth path.

## Doc 4: Code Review Guide

### Review Order

1. Contract shape first (request/response + query keys) (Ref: `apps/react-vite/src/features/**/api/*.ts` patterns).
2. Authorization enforcement symmetry (UI + handler) (Ref: `apps/react-vite/src/features/discussions/components/delete-discussion.tsx:28-52`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:193-214`).
3. Mutation side effects (`invalidateQueries`/`refetchQueries`) (Ref: `apps/react-vite/src/features/discussions/api/update-discussion.ts:39-43`).
4. Error/redirect behavior through API client interceptor (Ref: `apps/react-vite/src/lib/api-client.ts:26-40`).
5. Test update sufficiency (integration + policy edge cases) (Ref: `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx:77-145`).

### Fast Red-Flag Heuristics

1. Changed endpoint string without changed handler.
2. New mutation without cache invalidation.
3. UI auth check added, handler auth unchanged.
4. Response shape changed without fetcher type updates.

## Doc 5: Debugging Guide

### Most Useful Trace Route

1. Reproduce in browser.
2. Inspect component query state (`isLoading`, `data`) in feature components.
3. Inspect fetcher and query key.
4. Inspect handler and auth/policy check.
5. Inspect mock DB records.

Anchor refs:
- `apps/react-vite/src/features/comments/components/comments-list.tsx:19-93`
- `apps/react-vite/src/features/comments/api/get-comments.ts:22-47`
- `apps/react-vite/src/testing/mocks/handlers/comments.ts:14-131`
- `apps/react-vite/src/testing/mocks/db.ts:39-97`

### Common Failure Modes

1. Missing cookie -> 401 -> login redirect loop (Ref: `apps/react-vite/src/lib/api-client.ts:33-38`).
2. Wrong team scoping -> empty list with no explicit error (Ref: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:44-48`).
3. Query key mismatch -> stale UI after mutation (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:24`, `apps/react-vite/src/features/discussions/api/delete-discussion.ts:29-31`).

## Doc 6: Change Playbook

### Add Feature Safely (End-to-End)

1. Define/update domain types (`src/types/api.ts`) if contract changes (Ref: `apps/react-vite/src/types/api.ts:20-50`).
2. Add handler endpoint and policy checks (Ref pattern: `apps/react-vite/src/testing/mocks/handlers/discussions.ts:133-221`).
3. Add feature fetcher/query options + mutation hooks (Ref pattern: `apps/react-vite/src/features/comments/api/create-comment.ts:17-47`).
4. Add/modify route loader if prefetch needed (Ref: `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:15-36`).
5. Build UI components with shared form/table/dialog primitives (Ref: `apps/react-vite/src/features/discussions/components/create-discussion.tsx:27-70`).
6. Add integration tests with `renderApp` and fixture generators (Ref: `apps/react-vite/src/testing/test-utils.tsx:57-94`).

## Doc 7: Interview Walkthrough

### How To Explain This System Out Loud

1. “This repo demonstrates one domain across three front-end shells.”
2. “In the Vite app, route modules prefetch with React Query loaders.”
3. “Feature APIs are thin wrappers over a centralized Axios client.”
4. “A mock backend uses MSW handlers with role checks and fake persistence.”
5. “Tests run integration-first using those same handlers.”

Evidence anchors:
- `apps/react-vite/src/app/router.tsx:24-88`
- `apps/react-vite/src/lib/api-client.ts:16-42`
- `apps/react-vite/src/testing/mocks/handlers/index.ts:13-23`
- `apps/react-vite/src/testing/mocks/db.ts:45-97`
- `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx:77-145`

