# Journal

## Mission Design Rationale

The mission order follows the way the system executes in reality: startup -> routing -> typed contracts -> component composition -> API chain -> handler policy -> persistence -> testing/ownership loops (Ref: `apps/react-vite/src/main.tsx:11-17`, `apps/react-vite/src/app/router.tsx:24-88`, `apps/react-vite/src/features/discussions/api/get-discussion.ts:7-35`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:18-222`, `apps/react-vite/src/testing/mocks/db.ts:39-97`).

## Why Missions Are Ordered This Way

1. Missions 1-8 teach “read unfamiliar code and orient quickly.”
2. Missions 9-17 teach “trace behavior and evaluate implementation decisions.”
3. Missions 18-24 teach “prevent regressions, critique architecture, and own outcomes.”

## Chosen Code Paths

- Canonical runtime chain: `main.tsx -> AppProvider -> AppRouter` (Ref: `apps/react-vite/src/main.tsx:11-17`, `apps/react-vite/src/app/index.tsx:4-9`).
- Canonical domain chain: `discussions/comments` because it includes list/detail, pagination, role checks, create/update/delete, and UI feedback loops (Ref: `apps/react-vite/src/features/discussions/components/discussions-list.tsx:42-92`, `apps/react-vite/src/features/comments/components/comments-list.tsx:19-93`).
- Canonical backend simulation chain: `handlers/* + db.ts` (Ref: `apps/react-vite/src/testing/mocks/handlers/index.ts:13-23`, `apps/react-vite/src/testing/mocks/db.ts:39-97`).

## Skills Developed Per Tier

- Junior: mapping files to runtime responsibilities.
- Mid-Level: connecting contracts, state, and side effects.
- Senior: risk modeling, architectural critique, and test strategy.

## What A Senior Engineer Does Differently At Each Checkpoint

1. Looks for implicit coupling (query key, response shape, policy symmetry).
2. Checks failure paths as carefully as happy paths.
3. Uses minimal but high-leverage tests to pin risky behavior.
4. Plans migrations by preserving contracts while swapping infrastructure.

