# 📖 Glossary

Plain-English definitions of terms used throughout these docs. Chapters where each concept is developed are linked.

**Barrel file** — an `index.ts` that re-exports a folder's contents so consumers import the folder. Formerly recommended as a feature's "public API"; now discouraged here because it defeats Vite tree-shaking. [ch. 2](02-project-structure.md)

**Cache invalidation** — marking cached data as stale so it gets refetched. In react-query: `queryClient.invalidateQueries({ queryKey })`. The step juniors forget after mutations. [ch. 5](05-api-layer.md)

**CDN (Content Delivery Network)** — a network of geographically distributed servers that cache your static files near users. [ch. 11](11-deployment.md)

**Code splitting** — breaking the JS bundle into chunks loaded on demand (usually per route) via dynamic `import()`. [ch. 10](10-performance.md)

**Colocation** — keeping code (state, styles, components, tests) as close as possible to where it's used. The recurring principle of this whole architecture. [ch. 2](02-project-structure.md), [ch. 4](04-components-and-styling.md)

**Discriminated union** — a TypeScript union where a property determines which shape applies, letting the compiler enforce "either these props or those, not both." Used by the `Authorization` component. [ch. 8](08-security.md)

**E2E (end-to-end) test** — a test driving a real browser through the full running system. Few, slow, high-confidence. [ch. 9](09-testing.md)

**Error boundary** — a React component that catches exceptions thrown during rendering below it and shows a fallback instead of unmounting the whole app. Place several, at meaningful seams. [ch. 7](07-error-handling.md)

**Feature folder** — a self-contained module under `src/features/` holding everything specific to one domain area (its api, components, hooks...). Deletable in one `rm -rf`. [ch. 2](02-project-structure.md)

**Fetcher** — a plain typed function that performs one API request via the shared client. Part 1 of the request-declaration pattern. [ch. 5](05-api-layer.md)

**Headless component library** — components that ship behavior and accessibility with zero styling (Radix, Headless UI). You bring the CSS. [ch. 4](04-components-and-styling.md)

**HttpOnly cookie** — a cookie flagged (by the *server*) as unreadable to JavaScript, making tokens in it unstealable via XSS. Preferred token storage. [ch. 8](08-security.md)

**Integration test** — a test rendering a feature with its real wiring (router, query client, forms), faking only the server (via MSW). The workhorse test type here. [ch. 9](09-testing.md)

**Interceptor** — a function the API client runs on every request or response; where cross-cutting concerns (auth headers, error toasts, 401 redirects) live once. [ch. 5](05-api-layer.md), [ch. 7](07-error-handling.md)

**JWT (JSON Web Token)** — a signed token the server issues at login and validates on each request; the common SPA session mechanism. [ch. 8](08-security.md)

**MSW (Mock Service Worker)** — intercepts real HTTP requests and answers them from handler functions + an in-memory DB. Powers development, integration tests, and API design here. [ch. 9](09-testing.md)

**Mutation** — a server operation that changes data (POST/PUT/PATCH/DELETE), managed by `useMutation`. Pairs with cache invalidation. [ch. 5](05-api-layer.md)

**PBAC (permission/policy-based access control)** — authorization by evaluating a policy function of the user *and* the resource ("only the comment's author may delete it"). [ch. 8](08-security.md)

**Prefetching** — fetching data into the cache before the user asks (e.g. on hover), so navigation renders instantly. [ch. 10](10-performance.md)

**Query key** — react-query's cache address for a piece of server data (`['discussions', { page }]`). Same key ⇒ shared cache entry. Declare once, reuse everywhere. [ch. 5](05-api-layer.md)

**RBAC (role-based access control)** — authorization by role alone ("admins may delete discussions"). [ch. 8](08-security.md)

**Re-render** — React re-running a component's function to compute new UI. Happens on state change, for the component and (by default) its subtree. Managing *where* state lives manages re-renders. [ch. 10](10-performance.md)

**Server cache state** — your local copy of data the server owns. Not app state; a cache with staleness problems — hence react-query, not Redux/Zustand, manages it. [ch. 6](06-state-management.md)

**SPA (single-page application)** — an app served as one `index.html` + JS that handles routing client-side. Deploys as static files; needs the catch-all redirect rule. [ch. 11](11-deployment.md)

**Tree-shaking** — the bundler dropping code nothing imports. Barrel files obscure what's used, hurting it. [ch. 2](02-project-structure.md)

**Unidirectional architecture** — the import rule *shared → features → app*, enforced by ESLint. Files may import from layers above them in that list, never below. [ch. 2](02-project-structure.md)

**XSS (cross-site scripting)** — attacker-controlled content executing as script in users' browsers. React escapes by default; sanitize (DOMPurify) anything that bypasses escaping. [ch. 8](08-security.md)

**Zod** — a runtime validation library whose schemas also generate TypeScript types (`z.infer`), giving one source of truth for form validation and API types. [ch. 5](05-api-layer.md)
