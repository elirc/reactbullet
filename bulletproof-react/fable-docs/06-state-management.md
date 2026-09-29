# 🗃️ State Management

The single most useful state-management insight for a junior: **"state" is not one thing.** There are five distinct kinds, each with different lifetimes and different best tools. Most state-management pain comes from using the wrong category's tool — usually, stuffing everything into one global store.

Before reaching for any tool, ask: *which kind of state is this?*

| Kind | Example in this app | Right tool here |
|------|---------------------|-----------------|
| Component state | Is the sidebar dropdown open? | `useState` / `useReducer` |
| Application state | Notification toasts queue | Zustand store |
| Server cache state | The list of discussions | react-query |
| Form state | The half-filled "create discussion" form | react-hook-form + Zod |
| URL state | Current page number, discussion id | Router (path + search params) |

## 1. Component state — `useState` / `useReducer`

State owned by one component. Start here **by default**; lift it up only when another component actually needs it.

- [`useState`](https://react.dev/reference/react/useState) for independent values.
- [`useReducer`](https://react.dev/reference/react/useReducer) when several pieces of state change together in response to named actions.

Real example: [`dashboard-layout.tsx`](../apps/react-vite/src/components/layouts/dashboard-layout.tsx) keeps its mobile-nav open/closed state entirely to itself — no other component cares, so nothing else should know it exists.

**Why start local:** every level you lift state adds re-renders and coupling. Local state is free; global state is a tax every future reader pays.

## 2. Application state — small global stores

Truly global client-side state: toasts, theme, global modals. This app has exactly **one** global store — notifications ([`notifications-store.ts`](../apps/react-vite/src/components/ui/notifications/notifications-store.ts)), using [Zustand](https://github.com/pmndrs/zustand):

```ts
export const useNotifications = create<NotificationsStore>((set) => ({
  notifications: [],
  addNotification: (notification) =>
    set((state) => ({
      notifications: [...state.notifications, { id: nanoid(), ...notification }],
    })),
  dismissNotification: (id) =>
    set((state) => ({
      notifications: state.notifications.filter((n) => n.id !== id),
    })),
}));
```

Note how little there is. That's the point — after server cache, form, and URL state are handled by their own tools, **very little genuinely global state remains**. If your global store is large, you've probably miscategorized server or form state into it.

(Also note a neat consequence of Zustand living outside React: the [API client interceptor](../apps/react-vite/src/lib/api-client.ts) — not a component — pushes error toasts via `useNotifications.getState().addNotification(...)`.)

Alternatives you'll meet in the wild: [Redux Toolkit](https://redux-toolkit.js.org/) (common in older/larger codebases), [Jotai](https://github.com/pmndrs/jotai), [MobX](https://mobx.js.org), [XState](https://xstate.js.org/), or plain React [context](https://react.dev/learn/passing-data-deeply-with-context) + hooks for low-velocity data.

## 3. Server cache state — react-query

Data fetched from the server is **not your state — it's a cache of someone else's state**. The server owns it; your copy can go stale. That reframing is why a cache library (react-query, [SWR](https://swr.vercel.app/), [Apollo](https://www.apollographql.com/) for GraphQL, [RTK Query](https://redux-toolkit.js.org/rtk-query)) beats storing server data in Redux/Zustand: staleness, refetching, deduplication, and invalidation are *cache problems*, and cache libraries solve them out of the box.

This was covered in depth in the [API Layer](05-api-layer.md) chapter — in this repo, server state never touches a global store. If you remember one rule: **fetched data goes in react-query, full stop.**

## 4. Form state — react-hook-form + Zod

Forms have their own state universe: values, per-field validation, dirty/touched flags, submit status. This repo wraps [react-hook-form](https://react-hook-form.com/) into its own [`Form` component](../apps/react-vite/src/components/ui/form/form.tsx) and [`Input`/field components](../apps/react-vite/src/components/ui/form/input.tsx), so feature code looks like:

```tsx
<Form onSubmit={...} schema={createDiscussionInputSchema}>
  {({ register, formState }) => (
    <>
      <Input label="Title" error={formState.errors['title']} registration={register('title')} />
      ...
    </>
  )}
</Form>
```

The Zod schema — the *same one* the API layer exports ([chapter 5](05-api-layer.md)) — drives validation. See it end-to-end in [`register-form.tsx`](../apps/react-vite/src/features/auth/components/register-form.tsx).

**Why wrap the library** instead of using react-hook-form directly everywhere: the wrapper bakes in the schema resolver, error display, and styling once. Features can't do forms inconsistently, and swapping form libraries someday means rewriting one folder. (Alternatives: [Formik](https://formik.org/), [React Final Form](https://github.com/final-form/react-final-form); validation alternative: [yup](https://github.com/jquense/yup).)

## 5. URL state — the address bar

Which discussion is open (`/app/discussions/:discussionId`), which page of results (`?page=2`) — keep it in the URL via the router, not in a store.

Real example: [`discussions-list.tsx`](../apps/react-vite/src/features/discussions/components/discussions-list.tsx) reads `page` from search params with `useSearchParams()`; pagination *is* navigation.

**Why:** URL state survives refresh, is shareable/bookmarkable, and gets back/forward buttons for free. If a user refreshes on page 3 and lands on page 1, someone stored URL state in a component. It's also the one state category that needs **no library state at all** — the browser holds it.

## Deciding: a flowchart in words

1. Does it come from the server? → **react-query** (5 → chapter 5).
2. Is it a form the user is filling in? → **react-hook-form**.
3. Should it survive refresh or be shareable? → **URL**.
4. Do multiple, unrelated parts of the app need it? → **small global store** (or context for low-velocity data).
5. Otherwise → **`useState` in the component.** This is the correct answer far more often than juniors expect.

## Common junior mistakes

1. **Global-by-default.** Putting fetched data or one screen's UI state into a global store "so it's accessible." Accessibility is coupling; see the flowchart.
2. **Mirroring server data into `useState`** (`useEffect(() => setItems(data), [data])`). You now have two copies that can disagree. Render from `data` directly.
3. **Storing derived state.** If it can be computed from existing state/props, compute it during render — don't `setState` it.
4. **Context for everything.** Context is a dependency-injection mechanism, not a state manager; every consumer re-renders on any change. Fine for themes/user; wrong for fast-changing data (see [Performance](10-performance.md)).

## Try it yourself

1. Pick any screen in the app (say, the discussions page) and inventory its state: list every piece and label it with one of the five categories. Check your labels against where the code actually keeps it.
2. In the app, go to page 2 of discussions and refresh. It stays on page 2. Find the line of code responsible.
3. Grep the codebase for `create(` from zustand. Count global stores. Reflect on why the number is so small.
