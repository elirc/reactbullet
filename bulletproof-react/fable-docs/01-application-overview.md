# 💻 Application Overview

Before studying *how* the app is built, know *what* it is. Every architectural decision in these docs makes more sense when you can map it back to a concrete screen or feature.

## What the app does

It's a team discussion platform (think a minimal Discourse or internal forum):

- Users **register** and either create a new team (becoming its admin) or join an existing one.
- Team members create **discussions** (topics).
- Members post **comments** inside discussions.

[Live demo](https://bulletproof-react-app.netlify.app)

## Data model

Four entities. Learn these now — the `features/` folders in the code map almost one-to-one onto them.

```
Team ──< User ──< Discussion ──< Comment
        (role:
         ADMIN | USER)
```

- **Team** — has exactly one admin and many members. Created at registration if the user didn't join an existing one.
- **User** — belongs to a team and has a role:
  - `ADMIN`: create/edit/delete discussions, delete any comment, delete users, edit own profile
  - `USER`: edit own profile, create/delete **own** comments only
- **Discussion** — a topic created by a team member.
- **Comment** — a message inside a discussion.

> **Why roles matter architecturally:** the `ADMIN`/`USER` split is what drives the entire [Security](08-security.md) chapter — role-based access control (RBAC) for coarse rules like "only admins create discussions," and permission-based access control (PBAC) for fine-grained rules like "you can only delete *your own* comment." Notice the second rule can't be expressed as a role check alone; it depends on *which* comment. Keep that distinction in mind for later.

The TypeScript definitions for all four entities live in [`src/types/api.ts`](../apps/react-vite/src/types/api.ts). Open it — it's short, and every API chapter references these types.

## The three app implementations

This repo contains the **same app built three times** with different frameworks:

| App | Framework | When you'd choose it |
|-----|-----------|----------------------|
| [`apps/react-vite`](../apps/react-vite/README.md) | React + Vite (pure SPA) | Client-only apps, dashboards, internal tools |
| [`apps/nextjs-app`](../apps/nextjs-app/README.md) | Next.js App Router | New server-rendered apps; React Server Components |
| [`apps/nextjs-pages`](../apps/nextjs-pages/README.md) | Next.js Pages Router | Legacy Next.js codebases you'll encounter at work |

**These docs use `react-vite` for all code examples** because it has the least framework magic — what you see is plain React. Once you understand it, diff it against the Next.js versions to see exactly what a meta-framework changes (routing, data loading, rendering location) and what stays identical (the feature structure, the API layer, state management).

## Run it

```sh
cd apps/react-vite
cp .env.example .env
yarn install
yarn dev
```

There is no real backend — [MSW](https://mswjs.io/) (Mock Service Worker) intercepts every HTTP request in the browser and serves responses from an in-memory mock database ([`src/testing/mocks/db.ts`](../apps/react-vite/src/testing/mocks/db.ts)). This surprises most people the first time: the network tab shows real-looking requests, but no server is running. The [Testing chapter](09-testing.md) explains why this is a deliberate, powerful choice rather than a shortcut.

## Try it yourself

1. Run the app, register a user, create a discussion, and comment on it.
2. Open the browser DevTools **Network** tab while doing so. Find the `POST /discussions` request. Where did the response come from? (Hint: look for `mockServiceWorker.js`.)
3. Register a second user with the `USER` role and confirm you cannot delete the first user's comment. You've just observed PBAC — you'll implement it in [chapter 8](08-security.md).
