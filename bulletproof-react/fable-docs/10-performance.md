# 🚄 Performance

Junior framing first: **don't optimize speculatively.** Measure (React DevTools Profiler, Lighthouse), find the actual bottleneck, fix that. But *architectural* performance decisions — code splitting, state placement, styling strategy — are cheap on day one and expensive to retrofit, so learn them now.

Two different performance problems, often conflated:

1. **Load performance** — how fast the app *arrives* (bundle size, code splitting, images).
2. **Runtime performance** — how fast it *responds* once loaded (re-renders, state design).

## Load performance

### Code splitting — ship the page they asked for, not the whole app

By default a bundler produces one JS file containing your entire app; users on the login page download the admin dashboard. Code splitting breaks the bundle into chunks fetched on demand.

**Split at the route level** — it's the natural seam (a user visits one route at a time). See [`app/router.tsx`](../apps/react-vite/src/app/router.tsx): each route's component is loaded via a dynamic `import()` in its `lazy` property, so Vite emits one chunk per route and the browser fetches a chunk on first navigation to it.

Two cautions:

- **Don't over-split.** Hundreds of tiny chunks mean hundreds of round-trips; you can make loading *slower*. Routes first; go finer only for something huge and rarely used (a chart library, an editor).
- Splitting below the route level trades initial load for a spinner *later* — you're moving the wait, not deleting it.

### Images

- **Lazy-load** below-the-fold images (`loading="lazy"`).
- Serve **modern formats** (WebP/AVIF) — dramatically smaller than PNG/JPEG.
- Use **`srcset`** so phones don't download desktop-sized images.

### Web vitals

Google's user-experience metrics (LCP, CLS, INP) affect both real UX and search ranking. Check [Lighthouse](https://web.dev/measure/) / [PageSpeed Insights](https://pagespeed.web.dev/) before and after perf work — it's your measurement layer.

## Runtime performance

The dominant cost in React apps is **unnecessary re-renders**. Core mental model: *when state changes, React re-renders that component and — by default — everything below it.* All re-render optimization is about narrowing "everything below it."

### Weapon #1: put state low

The cheapest optimization is state placement — covered in [chapter 6](06-state-management.md), restated here as a perf rule:

- **Keep state as low in the tree as possible.** State in the root re-renders the world; state in a leaf re-renders a leaf.
- **Don't merge unrelated state.** One giant store/context object means every consumer re-renders when *any* field changes. Split by usage.

### Weapon #2: `children` — free memoization nobody teaches

The most underrated trick in React:

```jsx
// ❌ PureComponent re-renders on every count change —
//    it's created inside Counter's render
const Counter = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>count is {count}</button>
      <PureComponent />
    </div>
  );
};

// ✅ passed as children, it does NOT re-render when count changes
const App = () => (
  <Counter>
    <PureComponent />
  </Counter>
);

const Counter = ({ children }) => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>count is {count}</button>
      {children}
    </div>
  );
};
```

**Why it works:** `children` is created by `App`'s render, not `Counter`'s. When `Counter`'s state changes, the `children` element is *the same object as last time*, and React skips re-rendering it. No `memo`, no `useMemo` — just component composition. Layout components (sidebars, shells) get this benefit automatically, which is one more reason the layout/children pattern is everywhere in this repo.

### Weapon #3: lazy state initialization

```js
// ❌ myExpensiveFn() runs on EVERY render (its result is discarded after the first — but it still ran)
const [state, setState] = useState(myExpensiveFn());

// ✅ initializer function — runs exactly once, on mount
const [state, setState] = useState(() => myExpensiveFn());
```

Tiny syntax difference, real cost difference. Reach for it whenever the initial value requires computation (parsing localStorage, building a big structure).

### Context, used wisely

Context re-renders **every consumer** on any value change and has no built-in selectors. So:

- Fine for **low-velocity** data: theme, locale, current user.
- Wrong for fast-changing data — use a store with selectors (Zustand/Jotai have them built in; [`use-context-selector`](https://github.com/dai-shi/use-context-selector) retrofits them).
- Before reaching for context at all: would [lifting state up](https://react.dev/learn/sharing-state-between-components#lifting-state-up-by-example) or [composition](https://react.dev/learn/passing-data-deeply-with-context#before-you-use-context) solve it? Context is for *transporting* data past many layers, not for managing it.

### Styling choice as a perf decision

Runtime CSS-in-JS (emotion, styled-components) computes styles during render — a per-update tax in update-heavy apps. Zero-runtime solutions (Tailwind, vanilla-extract, CSS modules) pay the cost at build time. This is one reason this repo uses Tailwind, and why RSC-era codebases are migrating off runtime CSS-in-JS.

## Bonus: perceived performance — prefetching

Sometimes the fastest fetch is one that already happened. React-query lets you warm the cache before navigation. [`discussions-list.tsx`](../apps/react-vite/src/features/discussions/components/discussions-list.tsx) prefetches the *next page* of discussions on hover of the pagination link:

```tsx
onMouseEnter={() => {
  queryClient.prefetchQuery(getDiscussionsQueryOptions({ page: nextPage }));
}}
```

By the time the user finishes clicking, the data is often already cached — the page renders instantly. Note it reuses the same `getDiscussionsQueryOptions` from [chapter 5](05-api-layer.md): because prefetch and query share the exact same key, the cache connects them automatically. The pattern pays off again.

## What this chapter deliberately didn't lead with

`React.memo`, `useMemo`, `useCallback`. They're real tools, but juniors reach for them first when they should be last: they add complexity, are easy to break silently (one unstable prop and the memo does nothing), and usually paper over a state-placement problem the earlier weapons fix outright. Structure first, memoize later, measure always. (The React Compiler is progressively making manual memoization unnecessary anyway.)

## Try it yourself

1. Run the app, open React DevTools Profiler, record while typing in a form. Look at what re-renders per keystroke and explain why using the "state placement" lens.
2. Build the counter example above both ways in a scratch file; add a `console.log` in `PureComponent`. Watch the log confirm the `children` claim.
3. `yarn build` in `apps/react-vite`, then inspect the output chunks in `dist/assets/`. Match chunks to routes. Delete the `lazy` dynamic import from one route, rebuild, and watch the chunks merge.
