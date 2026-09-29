# 🌐 Deployment

The original doc is four bullet points; a junior deserves the mental model behind them.

## What a built SPA actually is

Run `yarn build` in `apps/react-vite` and look at `dist/`:

```
dist/
  index.html
  assets/
    index-4f3a91.js      # your app, minified, hashed
    react-vendor-....js  # split chunks (see ch. 10)
    index-....css
```

That's the entire deployable: **static files**. No Node server, no runtime — any dumb file host can serve an SPA. This is why SPA deployment is so much simpler than backend deployment, and why the recommended platforms are all "static hosting + CDN":

- [Netlify](https://www.netlify.com/) (the demo app runs here) · [Vercel](https://vercel.com/) · [AWS CloudFront + S3](https://aws.amazon.com/cloudfront/) · [Cloudflare](https://www.cloudflare.com/en-gb/cdn/)

(Contrast: the `nextjs-app` variant *does* need a server runtime for SSR/RSC — that's a real cost of choosing a meta-framework, and why Vercel/Netlify abstract it for you.)

## Why a CDN

A CDN copies your static files to servers around the world and serves each user from the nearest one. For a static SPA there's no reason *not* to: latency drops globally, your origin does nothing, and traffic spikes are absorbed by the edge. All four platforms above do this by default — "deploy to a CDN" is less a decision than a default you shouldn't undo.

## The three things that actually bite people

### 1. The SPA redirect rule

Your app routes client-side: `/app/discussions/123` exists only in react-router's head. A user *refreshing* on that URL asks the file host for `/app/discussions/123` — which isn't a file → **404**. Every SPA host needs the rewrite rule "serve `index.html` for any unknown path." That's what [`public/_redirects`](../apps/react-vite/public/_redirects) is (Netlify syntax):

```
/*  /index.html  200
```

If you ever see "works when I navigate, 404s when I refresh," this is it. Memorize this bug; you will meet it.

### 2. Caching + content hashes

Note the hashes in filenames (`index-4f3a91.js`). They enable the standard caching strategy: cache hashed assets *forever* (a content change ⇒ new hash ⇒ new URL), but **never cache `index.html`** (it's the pointer to the current hashes). Platforms configure this for you; know it anyway so cache-related "users see the old version" bugs make sense.

### 3. Environment variables are baked in at build time

There is no server to read env vars at runtime — `env.API_URL` ([`src/config/env.ts`](../apps/react-vite/src/config/env.ts)) is inlined into the JS **during build**. Consequences:

- Changing an env var requires a rebuild, not a restart.
- Per-environment (staging/prod) means per-environment *builds*.
- **Never put secrets in frontend env vars.** They ship, readable, to every browser. Frontend env vars are configuration, not secrets.

## CI/CD in this repo

Deployment isn't a manual `yarn build` + drag-and-drop; it's the last step of a pipeline. See [`.github/workflows/react-vite-ci.yml`](../.github/workflows/react-vite-ci.yml): on every push, CI runs lint, type checks, unit/integration tests, e2e tests, and the production build. Platforms like Netlify then deploy on green. The chain from [chapter 3](03-project-standards.md) completes here: editor → pre-commit hook → CI → deploy, each layer catching what the previous one missed.

## Try it yourself

1. `yarn build`, then `yarn preview` — you're now serving the real production bundle locally. Hard-refresh a deep route and confirm the dev server's SPA fallback handles it.
2. Deploy `dist/` to a free Netlify/Vercel account (drag-and-drop works). Refresh a deep route *without* the redirect rule, observe the 404, add the rule, redeploy.
3. Find where `env.API_URL` ended up: grep the built JS in `dist/assets/` for your API URL string. Now you've *seen* "baked in at build time."
