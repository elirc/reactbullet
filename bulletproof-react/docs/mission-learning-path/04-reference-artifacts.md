### Mission 25: Write the Docs That Don't Exist
**Tier:** Senior  
**Time Estimate:** 120 minutes  
**Goal:** Produce working-reference documents that engineers can use while shipping changes.

## Doc 1: Junior Onboarding Checklist

### Day 1 Actions

1. Run `yarn prepare` at repo root to install all app dependencies (Ref: `package.json:7`).
2. Pick `apps/react-vite` as initial training target and copy `.env.example` to `.env` (Ref: `apps/react-vite/README.md:15-18`, `apps/react-vite/.env.example:1-2`).
3. Run `yarn dev` and confirm port `3000` (Ref: `apps/react-vite/package.json:6`, `apps/react-vite/vite.config.ts:11-13`).
4. Open startup chain files in order: `main.tsx`, `app/index.tsx`, `app/provider.tsx`, `app/router.tsx` (Ref: `apps/react-vite/src/main.tsx:1-17`, `apps/react-vite/src/app/router.tsx:24-88`).

### First PR Checklist

1. Confirm feature API hook uses existing query key conventions (Ref: `apps/react-vite/src/features/discussions/api/get-discussions.ts:24`).
2. Confirm mutation invalidates/refetches affected query keys (Ref: `apps/react-vite/src/features/comments/api/delete-comment.ts:26-30`).
3. Confirm authorization is enforced in handler, not only UI (Ref: `apps/react-vite/src/testing/mocks/handlers/comments.ts:114-119`).
4. Add or update integration tests with `renderApp` (Ref: `apps/react-vite/src/testing/test-utils.tsx:57-94`).

### Questions To Ask Teammates

1. Which app variant is production-priority today?
2. Which cross-app parity checks are required before merge?
3. What policies are currently “demo-only” vs production-hard requirements?

## Doc 2: Architecture Guide For New Engineers

### Navigation Strategy

1. Start at route composition: `app/router.tsx` (Ref: `apps/react-vite/src/app/router.tsx:24-88`).
2. Jump to feature route module and its API hooks.
3. Jump to matching handler and fake DB model.
4. Verify tests covering that path.

### Runtime Ownership

1. `app/` owns startup, providers, routes.
2. `features/` owns business workflows and UI.
3. `lib/` owns cross-cutting platform code.
4. `testing/mocks/` owns backend simulation.

Ref: `apps/react-vite/src/app/index.tsx:1-10`, `apps/react-vite/src/features/comments/components/comments.tsx:1-17`, `apps/react-vite/src/lib/api-client.ts:1-42`, `apps/react-vite/src/testing/mocks/handlers/index.ts:13-23`

## Doc 3: Code Review Checklist

1. Contract shape:
   - Does fetcher type match handler response envelope? (Ref: `apps/react-vite/src/features/discussions/api/get-discussion.ts:11-13`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:123-124`)
2. Authorization symmetry:
   - Is action protected in UI and handler? (Ref: `apps/react-vite/src/features/discussions/components/delete-discussion.tsx:28-52`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:204-214`)
3. Cache coherence:
   - Are relevant query keys invalidated/refetched? (Ref: `apps/react-vite/src/features/discussions/api/update-discussion.ts:39-43`)
4. Error handling:
   - Are user-visible errors surfaced without hiding root cause? (Ref: `apps/react-vite/src/lib/api-client.ts:26-31`)
5. Test updates:
   - Is there at least one test that would fail before this change? (Ref baseline: `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx:77-145`)

## Doc 4: Debugging Playbook

### Failure Mode A: Redirect Loop to Login

1. Confirm cookie/token exists.
2. Verify `requireAuth` decode path.
3. Verify API client 401 redirect behavior.

Refs: `apps/react-vite/src/testing/mocks/utils.ts:80-103`, `apps/react-vite/src/lib/api-client.ts:33-38`

### Failure Mode B: Stale List After Mutation

1. Confirm mutation success callback ran.
2. Inspect query key invalidated.
3. Compare list query key shape.

Refs: `apps/react-vite/src/features/comments/api/create-comment.ts:39-43`, `apps/react-vite/src/features/comments/api/get-comments.ts:24`

### Failure Mode C: Action Visible But Fails On Click

1. Check `Authorization` component policy.
2. Check handler-side policy guard.

Refs: `apps/react-vite/src/lib/authorization.tsx:63-82`, `apps/react-vite/src/testing/mocks/handlers/comments.ts:114-119`

## Doc 5: Change Playbook

### Branch -> Implement -> Test -> Review -> Merge

1. Create branch per feature slice.
2. Update type contract first.
3. Update handler contract second.
4. Update feature API hook + component.
5. Add integration test.
6. Run lint/type/test commands.
7. Open PR with explicit risk checklist.

Anchor refs: `apps/react-vite/src/types/api.ts:20-50`, `apps/react-vite/src/testing/mocks/handlers/discussions.ts:133-221`, `apps/react-vite/src/features/discussions/api/create-discussion.ts:17-45`, `apps/react-vite/src/features/discussions/components/create-discussion.tsx:47-70`, `apps/react-vite/package.json:6-15`

## Doc 6: Senior Ownership Notes

### Monitor

1. Cross-app drift between `react-vite`, `nextjs-app`, `nextjs-pages`.
2. Authorization rule consistency.
3. Query key consistency and invalidation quality.
4. Test reliability in CI (Playwright + Vitest).

Refs: `apps/react-vite/src/features/users/api/get-users.ts:1-27`, `apps/nextjs-app/src/features/users/api/get-users.ts:1-27`, `apps/react-vite/src/lib/authorization.tsx:14-26`, `apps/react-vite/playwright.config.ts:14-55`

### Improve

1. Remove `any` hotspots.
2. Fix sequential loader awaits.
3. Harden security path for non-demo deployment.

Refs: `apps/react-vite/src/app/router.tsx:14`, `apps/react-vite/src/app/routes/app/discussions/discussion.tsx:23-28`, `apps/react-vite/src/testing/mocks/handlers/auth.ts:100-102`

### Leave Alone For Now

1. Basic feature architecture boundaries; they are coherent and enforced by lint rules (Ref: `apps/react-vite/.eslintrc.cjs:46-98`).

## Doc 7: Interview Walkthrough

### Practice Answer Skeleton

1. “We built one collaboration domain across three frontend shells.”
2. “The Vite variant uses explicit router loaders and React Query for server state.”
3. “Feature modules own API hooks + UI; platform modules own auth and transport.”
4. “A mock backend layer provides handler-level business rules and persistence simulation.”
5. “Integration tests exercise realistic user flows through the same contracts.”

Evidence anchors:
- `apps/react-vite/src/app/router.tsx:24-88`
- `apps/react-vite/src/features/discussions/api/get-discussions.ts:20-42`
- `apps/react-vite/src/lib/auth.tsx:62-89`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts:18-222`
- `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx:77-145`

