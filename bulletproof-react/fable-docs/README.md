# 🧭 Fable Docs — Bulletproof React for Junior Engineers

This is a rewritten, expanded version of the [original docs](../docs/) aimed at **upskilling a junior software engineer**. The originals tell you *what* the conventions are; these docs also explain **why they exist, how the real code in this repo implements them, and what goes wrong when you skip them**.

All code excerpts come from the actual [`apps/react-vite`](../apps/react-vite/) app in this repo, so you can open every file mentioned and see it in context.

## How to use these docs

1. Read in order. Each chapter builds on the previous one.
2. When a chapter links to a source file, **open it**. Reading real code is the point.
3. Do the "Try it yourself" exercises. You learn architecture by breaking and rebuilding it, not by reading about it.
4. When something is unclear, check the [Glossary](12-glossary.md).

## Reading path

| # | Chapter | You'll learn |
|---|---------|--------------|
| 1 | [Application Overview](01-application-overview.md) | What the app does, its data model, and how to run it |
| 2 | [Project Structure](02-project-structure.md) | Why code is organized into `features/`, and the one-direction import rule |
| 3 | [Project Standards](03-project-standards.md) | The tooling (ESLint, Prettier, TypeScript, Husky) that keeps a team's code consistent |
| 4 | [Components & Styling](04-components-and-styling.md) | How to structure components so they stay maintainable |
| 5 | [API Layer](05-api-layer.md) | The 3-part pattern every API call in this repo follows |
| 6 | [State Management](06-state-management.md) | The five kinds of state and which tool fits each |
| 7 | [Error Handling](07-error-handling.md) | Interceptors, error boundaries, and production error tracking |
| 8 | [Security](08-security.md) | Auth tokens, XSS, and role/permission-based access control |
| 9 | [Testing](09-testing.md) | The testing trophy: unit vs integration vs e2e, and MSW |
| 10 | [Performance](10-performance.md) | Code splitting, re-render control, and the `children` trick |
| 11 | [Deployment](11-deployment.md) | What actually happens when you ship an SPA |
| 12 | [Glossary](12-glossary.md) | Plain-English definitions of every term used in these docs |

## The one-paragraph summary of this whole architecture

Organize code by **feature**, not by file type. Make imports flow in **one direction** (shared → features → app) so any file's dependencies are predictable. Put every API call behind a **typed fetcher + react-query hook** pair. Classify state by *what kind of state it is* before picking a tool. Keep everything — state, styles, components, tests — **as close as possible to where it's used**. Enforce all of it with tooling (ESLint, TypeScript, Husky) instead of code-review nagging, because rules that machines enforce are the only rules that survive a growing team.

If you internalize that paragraph, everything else in these docs is detail.
