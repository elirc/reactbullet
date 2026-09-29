# 🧱 Components & Styling

Components are where juniors spend most of their time, and where most maintainability damage happens. This chapter covers the practices this repo follows and the reasoning behind each.

## Principle 1: Colocate — keep things next to where they're used

State, styles, helper functions, child components: define them as close as possible to their usage. Feature-specific components live in the feature ([chapter 2](02-project-structure.md)); state lives in the lowest component that needs it ([chapter 6](06-state-management.md)).

**Why:** distance is cost. Code you can see on the same screen is code you can understand and safely change. And in React specifically, hoisting state higher than needed re-renders more of the tree than needed — colocation is a *performance* practice too.

## Principle 2: Extract components instead of writing render functions

A pattern you'll see in older codebases and should not copy:

```jsx
// ❌ nested render functions — grows without limit, can't be tested or reused
function Discussion() {
  function renderComments() {
    return <ul>{/* 40 lines... */}</ul>;
  }
  function renderHeader() { /* ... */ }
  return (
    <div>
      {renderHeader()}
      {renderComments()}
    </div>
  );
}
```

```jsx
// ✅ real components — testable, reusable, and React can optimize them
function CommentsList() { return <ul>...</ul>; }

function Discussion() {
  return (
    <div>
      <DiscussionHeader />
      <CommentsList />
    </div>
  );
}
```

The rule of thumb: **if a chunk of JSX is a nameable unit of UI, make it a component.** Render functions look harmless at 10 lines; by 300 lines the file is unreviewable, and unlike components, render functions can't have their own memoization, error boundary, or test.

## Principle 3: Too many props → split or compose

A component taking 10+ props is trying to be several components. Two fixes:

1. **Split it** into the several components it's pretending not to be.
2. **Compose with `children`/slots** — instead of `<Dialog titleText confirmLabel cancelLabel onConfirm bodyContent icon .../>`, let the caller pass JSX.

See it done in this repo: [`ConfirmationDialog`](../apps/react-vite/src/components/ui/dialog/confirmation-dialog/confirmation-dialog.tsx) takes a `triggerButton` and `confirmButton` as **elements**, not as a dozen config props. The caller keeps full control of those buttons; the dialog only owns the dialog-ness.

**Why:** every prop is a promise the component must keep forever. Composition passes control back to the caller so the shared component stays small and stable.

## Principle 4: Build a small internal UI library — and wrap third-party components

`src/components/ui/` is this app's internal component library: [`button`](../apps/react-vite/src/components/ui/button/button.tsx), `dialog`, `form`, `table`, `notifications`... Every feature builds screens out of these instead of reinventing buttons.

Also note [`link`](../apps/react-vite/src/components/ui/link/link.tsx): it *wraps* react-router's `Link` rather than letting features import react-router directly. That's deliberate:

- One place to apply app-wide styling/behavior.
- If you ever swap the underlying library, you change **one file**, not every feature.

**A warning that will save you pain:** wrong abstractions cost more than duplication. Don't extract a shared component from *one* usage, or force two similar-looking screens into one mega-component with boolean props (`isCompact`, `isAdminVariant`, `hideFooter`...). Wait until repetition is real, then extract the genuinely common part.

## Choosing component libraries (the landscape)

You'll rarely choose this at work (the codebase already chose), but you should understand the tiers:

| Tier | What you get | Examples | Trade-off |
|------|--------------|----------|-----------|
| **Fully styled** | Complete, designed components | [Chakra UI](https://chakra-ui.com/), [MUI](https://mui.com/material-ui/), [AntD](https://ant.design/), [Mantine](https://mantine.dev/) | Fast start; fighting their design later |
| **Headless** | Behavior + accessibility, zero styling | [Radix UI](https://www.radix-ui.com/), [Headless UI](https://headlessui.dev/), [react-aria](https://react-spectrum.adobe.com/react-aria/), [Ark UI](https://ark-ui.com/) | You style everything; total design control |
| **Copy-in code** | Headless + styles, vendored into your repo | [ShadCN UI](https://ui.shadcn.com/), [Park UI](https://park-ui.com/) | You own (and maintain) the code |

The headless tier exists because accessibility (focus traps, keyboard nav, ARIA) is brutally hard to get right; let a library do that, keep the pixels yourself. **This repo uses the headless approach: Radix primitives + Tailwind**, visible throughout `src/components/ui/`.

## Styling

Options, grouped by *when styles are generated*:

- **Build time (zero runtime):** [Tailwind](https://tailwindcss.com/), [vanilla-extract](https://github.com/seek-oss/vanilla-extract), [CSS modules](https://github.com/css-modules/css-modules), [Panda CSS](https://panda-css.com/)
- **Runtime (CSS-in-JS):** [styled-components](https://styled-components.com/), [emotion](https://emotion.sh/docs/introduction)

Two reasons the ecosystem has moved toward build-time: runtime CSS-in-JS does styling work on every render (a real cost in update-heavy apps — see [Performance](10-performance.md)), and **React Server Components can't run runtime styling at all**.

This repo uses **Tailwind**. Look at [`button.tsx`](../apps/react-vite/src/components/ui/button/button.tsx) and note the pattern: `cva` (class-variance-authority) declares variants (`variant`, `size`) as named bundles of Tailwind classes, and the `cn` utility merges them with caller-supplied classes. That's the standard modern Tailwind component recipe — you'll see it in most codebases using ShadCN too.

## Storybook — your component workbench

[Storybook](https://storybook.js.org/) renders components **in isolation**, outside the app. Each `*.stories.tsx` file ([example](../apps/react-vite/src/components/ui/button/button.stories.tsx)) defines named states of a component.

Why juniors should love it: you can build and debug a component without clicking through five screens of app to reach it, and the story catalogue doubles as living documentation of what UI already exists — check it **before** building a "new" component that already exists.

## Common junior mistakes

1. Copy-pasting a similar component and tweaking it, instead of checking `components/ui/` (and Storybook) first.
2. Extracting a shared abstraction after the *first* usage — inverted: duplicating after the *fifth*.
3. Adding boolean props to a shared component to serve one caller's special case. That's the caller's code; use composition.
4. Writing `renderX()` functions inside components as they grow, instead of extracting components.

## Try it yourself

1. Run Storybook (`yarn storybook` in `apps/react-vite`) and explore the catalogue.
2. Read [`confirmation-dialog.tsx`](../apps/react-vite/src/components/ui/dialog/confirmation-dialog/confirmation-dialog.tsx). List which decisions it keeps for itself vs. hands to the caller via element props. That boundary *is* component design.
3. Refactor practice: find any component in `features/` over ~150 lines and sketch how you'd split it. Don't look for a perfect answer — look for nameable UI units.
