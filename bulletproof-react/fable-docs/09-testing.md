# 🧪 Testing

The strategy in one sentence: **write mostly integration tests, a few unit tests for shared/complex pieces, and a thin layer of e2e tests for critical flows** — because confidence comes from testing things *wired together*, and integration tests buy the most confidence per minute of maintenance.

## The three test types, and where each earns its place

### Unit tests — small, isolated pieces

Test one component or function in isolation. Best for **shared building blocks** (used everywhere, so bugs multiply) and **genuinely complex logic**.

Real example: [`confirmation-dialog.test.tsx`](../apps/react-vite/src/components/ui/dialog/confirmation-dialog/__tests__/confirmation-dialog.test.tsx) — the shared dialog opens, confirms, cancels correctly. Worth unit-testing precisely because every feature relies on it.

The limit of unit tests: 100 passing unit tests prove the parts work *alone*. Apps break at the **connections** — the form that validates but sends the wrong payload, the list that fetches but renders stale cache.

### Integration tests — a feature, wired together 🏆

Render a whole route/feature with real router, real react-query, real form logic — everything except the actual server (MSW stands in) — and interact like a user.

Real example: [`discussion.test.tsx`](../apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx). Skeleton of what such a test does:

```tsx
test('should render discussion', async () => {
  const fakeDiscussion = await createDiscussion();      // seed the mock DB
  await renderApp(<DiscussionRoute />, { url: ..., path: ... });

  expect(await screen.findByText(fakeDiscussion.body)).toBeInTheDocument();
});
```

One test exercises: routing, the query hook, the API client, loading states, and rendering. When it passes, the *feature* works, not just its parts. **This is where most of your tests should live.**

### End-to-end (e2e) — the whole system, real browser

[Playwright](https://playwright.dev) drives an actual browser through the deployed/built app: register → create discussion → comment → delete. See [`e2e/tests/smoke.spec.ts`](../apps/react-vite/e2e/tests/smoke.spec.ts).

E2e tests are the most realistic and the most expensive: slow, flaky-prone, painful to debug. So keep them few and pointed at **money paths** — the flows where breakage is a company emergency (auth, checkout, the core loop). Playwright runs headed locally (great step-by-step debugging UI) and headless in CI on every deploy.

## The tools

### Vitest — the test runner

[Vitest](https://vitest.dev) runs the unit/integration tests. Jest-compatible API (`describe`, `test`, `expect`, mocks) but native to the Vite ecosystem — same config, same transforms, fast. If you know Jest, you know Vitest.

### Testing Library — query the screen, not the internals

[Testing Library](https://testing-library.com/)'s philosophy is the deepest idea in this chapter: **test what the user experiences, not how the component is implemented.**

```tsx
// ❌ implementation detail — breaks when you rename state, survives when the UI is actually broken
expect(component.state.isOpen).toBe(true);

// ✅ user-observable behavior
await userEvent.click(screen.getByRole('button', { name: /open/i }));
expect(screen.getByRole('dialog')).toBeInTheDocument();
```

Why this matters: tests coupled to implementation **fail on refactors that didn't break anything** (state renamed → red) and **pass on changes that broke everything** (state fine, render broken → green). Testing the rendered output inverts both. Swap Zustand for Redux and behavioral tests shouldn't change at all.

Practical habits: prefer `getByRole`/`getByLabelText` (they also nudge you toward accessible markup); use `findBy*` for things that appear asynchronously; use `userEvent` over `fireEvent`.

### MSW — the fake server that makes integration tests honest

The usual way to test components that fetch is mocking the fetch function — which means your test never exercises your API layer at all. [MSW](https://mswjs.io) is better: it **intercepts real HTTP requests** (service worker in the browser, request interception in Node tests) and answers them from handlers you define.

This repo goes further: the handlers ([`src/testing/mocks/handlers/`](../apps/react-vite/src/testing/mocks/handlers)) implement actual business logic over an in-memory database ([`db.ts`](../apps/react-vite/src/testing/mocks/db.ts)) — auth checks, ownership rules, pagination. That single mock backend serves **three purposes**:

1. **Development** — build the frontend with no backend running (or before the backend exists).
2. **Integration tests** — components make *real* requests through the *real* API client; only the far end is fake.
3. **API design** — the handlers double as an executable spec of what the real API should do.

One definition of the API's behavior, reused everywhere. When the real backend arrives, you delete nothing — MSW simply doesn't run in production.

## What to test (and skip) — guidance for your first tests

- ✅ New feature route → one integration test: render it, do the main user action, assert the visible result.
- ✅ Shared UI component or tricky pure function → unit tests.
- ✅ Business-critical flow → ensure it's covered in an e2e smoke test.
- ❌ Don't test that React renders props, that a library works (react-query is already tested), or private helpers a behavioral test already covers.
- ❌ Don't chase coverage numbers. One integration test that catches a real regression beats ten snapshot tests nobody reads.

## Try it yourself

1. Run `yarn test` in `apps/react-vite`. Open [`discussions.test.tsx`](../apps/react-vite/src/app/routes/app/discussions/__tests__/discussions.test.tsx) and map each line to: seed data → render → interact → assert.
2. Break the app deliberately (e.g. in `discussions-list.tsx`, render the wrong field). Which tests catch it? Now break only a CSS class. Note that no test catches it — and think about why that's mostly fine.
3. Write one integration test yourself: assert that a `USER`-role account does **not** see the delete button on someone else's comment (the PBAC rule from [chapter 8](08-security.md)). The existing comment tests show the setup patterns you need.
