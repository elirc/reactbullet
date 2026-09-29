# ⚠️ Error Handling

Errors *will* happen: networks fail, servers 500, code throws. The question is whether the user sees a helpful message and keeps working, or a blank white screen. This app handles errors at three layers — memorize the layers, not just the code.

| Layer | Catches | Mechanism |
|-------|---------|-----------|
| API | Failed HTTP requests | axios response interceptor |
| UI | Exceptions thrown during render | React error boundaries (plural) |
| Production | Everything, after shipping | Error tracking service (Sentry) |

## Layer 1: API errors — handled once, in the interceptor

You saw this code in [chapter 5](05-api-layer.md); now focus on its error branch ([`api-client.ts`](../apps/react-vite/src/lib/api-client.ts)):

```ts
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    const message = error.response?.data?.message || error.message;
    useNotifications.getState().addNotification({
      type: 'error',
      title: 'Error',
      message,
    });

    if (error.response?.status === 401) {
      const redirectTo = searchParams.get('redirectTo') || window.location.pathname;
      window.location.href = paths.auth.login.getHref(redirectTo);
    }

    return Promise.reject(error);
  },
);
```

What to learn from each piece:

- **Every failed request produces a toast, automatically.** No feature can forget to surface an error, because no feature is responsible for surfacing errors. This is the payoff of the single API client: cross-cutting concerns get exactly one implementation.
- **401 (unauthenticated) is special-cased**: the user's session is dead, so no local handling makes sense — redirect to login, remembering where they were (`redirectTo`) so login returns them there. Interceptors are also the standard place for token-refresh logic in apps that use refresh tokens.
- **`return Promise.reject(error)` still propagates the error.** The interceptor adds behavior, it doesn't swallow the failure — react-query still sees the rejection and sets `isError`, so screens can *also* react locally (e.g. keep a modal open). A classic interceptor bug is returning normally here, which silently converts every failure into a "success" with `undefined` data.

## Layer 2: render errors — error boundaries, plural

When a component throws during render, React unmounts the tree up to the nearest [error boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary). **No boundary at all ⇒ the entire app unmounts ⇒ white screen.**

The junior-level takeaway is not "add an error boundary" but "**add several, at meaningful seams**." One app-wide boundary technically prevents the white screen but replaces your whole app with one error message — a crash in a sidebar widget shouldn't take down the discussion the user is reading.

This repo places boundaries at both levels:

- **App-wide** — last resort, in [`provider.tsx`](../apps/react-vite/src/app/provider.tsx), rendering [`MainErrorFallback`](../apps/react-vite/src/components/errors/main.tsx).
- **Per-section** — e.g. the discussion route wraps the comments section in its own boundary ([`discussion.tsx`](../apps/react-vite/src/app/routes/app/discussions/discussion.tsx)):

```tsx
<ErrorBoundary fallback={<div>Failed to load comments. Try to refresh the page.</div>}>
  <Comments discussionId={discussionId} />
</ErrorBoundary>
```

If comments crash, the discussion itself stays usable. The fallback is scoped, honest, and actionable.

Practical notes:

- Class components are the raw mechanism, but everyone uses [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary) (as this repo does) for a component + reset API.
- Boundaries catch **render/lifecycle** errors only — not event handlers, not async code. Async failures are Layer 1's job (promises reject; react-query exposes `isError`). If an event handler can throw, `try/catch` it yourself.

## Layer 3: production — error tracking

In development you see errors in the console. In production, *users* hit errors and you hear nothing — unless something reports them. That's error tracking: a service like [Sentry](https://sentry.io/) captures every uncaught error with stack trace, browser, OS, and frequency, and alerts you.

Things a junior should know before their first on-call:

- **Upload source maps.** Production JS is minified; without source maps every stack trace is `a.b @ chunk-4f3a.js:1:48231`. With them, it points at your actual source line.
- Don't build this yourself (`window.onerror` + a logging endpoint). Grouping, deduplication, release tracking, and alerting are the hard 95% you'd be rebuilding.
- Error tracking closes the loop with Layers 1–2: boundaries keep the user experience intact *and* the `onError` hook is where you report the caught error to Sentry. Graceful degradation without reporting just hides your bugs from you.

## How the three layers cooperate — one story

A user opens a discussion; the comments endpoint 500s.

1. The **interceptor** pops an error toast and rejects the promise.
2. react-query marks the comments query as errored; the comments component renders its error state — or, if a rendering bug in that path throws, the **section boundary** contains it to the comments panel. The discussion stays readable.
3. **Sentry** records the event; you see it in the morning with a stack trace, before most users notice.

That's the goal state: the user is inconvenienced, not blocked; you are informed, not oblivious.

## Try it yourself

1. Temporarily make [`Comments`](../apps/react-vite/src/features/comments/components/comments.tsx) throw at the top of its render (`throw new Error('boom')`). Observe that the discussion page survives and shows the scoped fallback. Then remove the route-level boundary and observe the blast radius grow.
2. In the MSW handlers ([`src/testing/mocks/handlers`](../apps/react-vite/src/testing/mocks/handlers)), make an endpoint return a 500 and watch the interceptor's toast appear with the server's message.
3. Answer without looking: which layer handles (a) a 401, (b) a `TypeError` while rendering a list item, (c) a bug only occurring on Safari in production?
