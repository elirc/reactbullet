# ⚙️ Project Standards

The theme of this chapter: **rules that machines enforce are the only rules that survive**. Style guides in wikis get ignored; lint errors that block your commit don't. Each tool below automates one category of rule so humans never have to argue about it in code review.

| Tool | Enforces | When it runs |
|------|----------|--------------|
| ESLint | Code *correctness and conventions* (unused vars, import rules, a11y) | Editor + pre-commit + CI |
| Prettier | Code *formatting* (quotes, semicolons, line width) | On save + pre-commit |
| TypeScript | *Type* correctness | Editor + build |
| Husky + lint-staged | All of the above, on changed files, **before a commit can land** | `git commit` |

## ESLint — the convention enforcer

ESLint parses your code and flags rule violations. In this repo it's doing heavy lifting well beyond "no unused variables" — open [`apps/react-vite/.eslintrc.cjs`](../apps/react-vite/.eslintrc.cjs) and find:

- `import/no-restricted-paths` — the **architecture rules** from [chapter 2](02-project-structure.md). The folder structure is enforced here, not in a wiki.
- `import/order` — imports sorted into groups (builtin → external → internal) with blank lines between. Ends every "import order" review comment forever.
- `check-file/filename-naming-convention` — all files kebab-case (see below).
- React, a11y (`jsx-a11y`), and testing-library plugin rules.

**Junior takeaway:** when ESLint yells at you, read the rule name in the error (e.g. `import/no-restricted-paths`) and look it up. The rule's docs explain *why* it exists — that's free mentorship.

## Prettier — the formatting robot

Prettier rewrites your code into one canonical format. Config: [`.prettierrc`](../apps/react-vite/.prettierrc). Two things to set up once:

1. **Format on save** in your editor (VS Code: `"editor.formatOnSave": true` — this repo ships it in [`.vscode/settings.json`](../apps/react-vite/.vscode/settings.json)).
2. Never manually format again.

Why teams use it: formatting debates ("tabs vs spaces") consume review time and produce noisy diffs. Prettier makes format a non-topic. Bonus: if format-on-save suddenly does nothing, you almost certainly have a **syntax error** — Prettier failing to parse is an early warning.

ESLint and Prettier overlap (ESLint has formatting rules too); this repo uses `eslint-config-prettier` to turn off ESLint's formatting rules so the two tools never fight. If you ever see ESLint and Prettier "correcting" each other in a loop, that config is missing.

## TypeScript — the refactoring safety net

ESLint catches patterns; TypeScript catches **type mismatches everywhere your data flows**. Where it earns its keep is refactoring: change a type in [`src/types/api.ts`](../apps/react-vite/src/types/api.ts) (say, rename `Discussion.title` to `Discussion.heading`) and the compiler lists *every* file that breaks. In plain JS, you'd find those files one runtime crash at a time.

The workflow for a large refactor: **change the type declarations first, then fix the compiler errors until they're gone.** The errors are your to-do list.

Two honest limits juniors should know:

- **TypeScript checks compile time, not runtime.** It trusts you that an API response matches your type. That's why this repo *validates* external data with Zod schemas at the boundary ([API Layer](05-api-layer.md)) — Zod checks at runtime what TS can only assume.
- `as` casts and `any` are you overriding the checker. Every one is a place the safety net has a hole. Sometimes necessary; never free.

Recommended reading: [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/).

## Husky — the pre-commit gate

[Husky](https://typicode.github.io/husky/) runs scripts on git hooks. This repo's [`.husky/pre-commit`](../.husky/pre-commit) runs **lint-staged**, which runs ESLint/Prettier/type checks **only on the files you changed** (checking the whole repo on every commit would be too slow).

Effect: a commit that violates the standards is rejected *before* it exists. CI still runs the full checks (see [`.github/workflows`](../.github/workflows/)), but pre-commit hooks give you the feedback in seconds instead of after a push.

> If you're ever tempted by `git commit --no-verify` to skip the hook: the failure doesn't disappear, it just moves to CI where it wastes more of everyone's time. Fix it locally.

## Absolute imports — `@/` instead of `../../..`

Configured in `tsconfig.json`:

```json
"compilerOptions": {
  "baseUrl": ".",
  "paths": { "@/*": ["./src/*"] }
}
```

So `src/components/ui/button/button.tsx` is imported as `@/components/ui/button/button` from anywhere. Two wins:

- **Moves don't break imports.** Relocate a file and its `@/...` imports still point at the same targets (imports *of* the file update via your editor's rename).
- **Readability.** `@/` visibly means "our code", distinct from `node_modules` packages.

Why one `@/*` alias instead of `@components`, `@hooks`, etc.: one alias is zero-maintenance and can never collide with a real npm package name.

(Vite also needs the alias in [`vite.config.ts`](../apps/react-vite/vite.config.ts) — `tsconfig` paths only inform the type checker, the bundler resolves separately. Forgetting one of the two is a classic setup bug: types pass but the build fails, or vice versa.)

## File naming — kebab-case, enforced

All files and folders are `kebab-case` (`create-discussion.ts`, not `CreateDiscussion.ts`), enforced by ESLint:

```js
'check-file/filename-naming-convention': [
  'error',
  { '**/*.{ts,tsx}': 'KEBAB_CASE' },
  { ignoreMiddleExtensions: true }, // allows vite.config.ts, smoke.spec.ts
],
'check-file/folder-naming-convention': [
  'error',
  { 'src/**/!(__tests__)': 'KEBAB_CASE' },
],
```

The convention itself matters less than *having exactly one*. Kebab-case has a practical edge: it avoids the classic macOS/Windows vs Linux **case-sensitivity trap**, where `Button.tsx` vs `button.tsx` works on your Mac and breaks in CI. Note: component *files* are kebab-case, but the component *inside* is still PascalCase (`discussion-view.tsx` exports `DiscussionView`).

## Try it yourself

1. Break a rule on purpose: create `src/features/discussions/components/MyComponent.tsx` and try to commit it. Watch which tool catches it and where.
2. Rename a field in `src/types/api.ts` and run `yarn tsc --noEmit`. Count the errors — that's the refactor's blast radius, computed for free. (Then revert.)
3. Open `.eslintrc.cjs` and pick two rules you don't recognize. Look them up. Explain to yourself (or a rubber duck) what bug each prevents.
