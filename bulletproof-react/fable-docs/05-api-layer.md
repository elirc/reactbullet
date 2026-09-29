# 📡 API Layer

Every network call in this codebase follows **one pattern, three parts**. Learn it once here and you can read (and write) any `api/` file in any feature. This chapter walks through the real files line by line.

## Part 0: One configured API client

All requests go through a single pre-configured axios instance — [`src/lib/api-client.ts`](../apps/react-vite/src/lib/api-client.ts):

```ts
export const api = Axios.create({
  baseURL: env.API_URL,
});

api.interceptors.request.use(authRequestInterceptor);
api.interceptors.response.use(
  (response) => {
    return response.data;            // ① unwrap: callers get data, not axios envelopes
  },
  (error) => {
    const message = error.response?.data?.message || error.message;
    useNotifications.getState().addNotification({   // ② every API error → toast, automatically
      type: 'error',
      title: 'Error',
      message,
    });

    if (error.response?.status === 401) {           // ③ session expired → redirect to login
      // ...
      window.location.href = paths.auth.login.getHref(redirectTo);
    }

    return Promise.reject(error);
  },
);
```

What the interceptors buy you:

1. **Unwrapping** — `api.get('/discussions')` resolves directly to the payload. Every caller skips writing `.then(res => res.data)`.
2. **Centralized error UX** — no feature ever writes its own "show error toast" logic; failures surface consistently for free. (More in [Error Handling](07-error-handling.md).)
3. **Centralized auth handling** — a 401 anywhere logs the user out to the login page, preserving where they were (`redirectTo`).

**Why one instance matters:** if features created their own `fetch`/axios calls, base URLs, headers, credentials and error handling would drift apart one file at a time. With one client, a change like "add an auth header to every request" is a one-file change.

## Parts 1–3: The request declaration pattern

Every endpoint gets its own file in the feature's `api/` folder containing: **types/schema → fetcher → hook**. Never inline API calls inside components.

### A query (reading data) — [`get-discussions.ts`](../apps/react-vite/src/features/discussions/api/get-discussions.ts)

```ts
// 1️⃣ THE FETCHER — a plain typed function that calls the endpoint
export const getDiscussions = (
  page = 1,
): Promise<{ data: Discussion[]; meta: Meta }> => {
  return api.get(`/discussions`, { params: { page } });
};

// 2️⃣ THE QUERY OPTIONS — cache key + fetcher, bundled
export const getDiscussionsQueryOptions = ({ page }: { page?: number } = {}) => {
  return queryOptions({
    queryKey: page ? ['discussions', { page }] : ['discussions'],
    queryFn: () => getDiscussions(page),
  });
};

// 3️⃣ THE HOOK — what components actually use
export const useDiscussions = ({ queryConfig, page }: UseDiscussionsOptions) => {
  return useQuery({
    ...getDiscussionsQueryOptions({ page }),
    ...queryConfig,
  });
};
```

Understanding each layer:

- **The fetcher** knows *how* to talk to the server, nothing else. Being a plain function, it's usable outside React too (tests, route loaders, prefetching).
- **The `queryKey`** is react-query's cache address. Same key ⇒ same cached data, shared by every component that asks. Page 2 has a different key than page 1, so they're cached independently. Getting keys consistent is *the* react-query skill — which is exactly why they're declared once here and exported, instead of being retyped at each call site.
- **The hook** is a thin veneer so components write `useDiscussions({ page })` and receive `{ data, isLoading, error, ... }` — with caching, deduping, and refetching handled by react-query. `queryConfig` lets a caller tweak options (e.g. `enabled`) without changing the shared declaration.

### A mutation (writing data) — [`create-discussion.ts`](../apps/react-vite/src/features/discussions/api/create-discussion.ts)

```ts
// 1️⃣ SCHEMA — runtime validation + the TS type derived from it
export const createDiscussionInputSchema = z.object({
  title: z.string().min(1, 'Required'),
  body: z.string().min(1, 'Required'),
});
export type CreateDiscussionInput = z.infer<typeof createDiscussionInputSchema>;

// 2️⃣ FETCHER
export const createDiscussion = ({ data }: { data: CreateDiscussionInput }): Promise<Discussion> => {
  return api.post(`/discussions`, data);
};

// 3️⃣ HOOK
export const useCreateDiscussion = ({ mutationConfig }: UseCreateDiscussionOptions = {}) => {
  const queryClient = useQueryClient();
  const { onSuccess, ...restConfig } = mutationConfig || {};

  return useMutation({
    onSuccess: (...args) => {
      queryClient.invalidateQueries({
        queryKey: getDiscussionsQueryOptions().queryKey,   // ← the key insight
      });
      onSuccess?.(...args);
    },
    ...restConfig,
    mutationFn: createDiscussion,
  });
};
```

Three things to internalize here:

- **`z.infer` — one source of truth.** The Zod schema does double duty: it validates the form at runtime ([used directly by the form component](../apps/react-vite/src/features/discussions/components/create-discussion.tsx)) *and* generates the TypeScript type. Change the schema, and the type updates itself. Without this you'd maintain a type and a validator separately, and they *would* drift.
- **Cache invalidation on success.** After creating a discussion, the cached discussions list is stale. `invalidateQueries` tells react-query "everything under the `['discussions']` key is outdated — refetch it." Notice it reuses `getDiscussionsQueryOptions().queryKey` instead of hand-typing `['discussions']` — so the key literally cannot drift between the query and the invalidation. **Forgotten invalidation is the #1 "why doesn't the UI update after I save?" bug.**
- **Caller hooks are preserved.** The declaration invalidates the cache, *then* calls the caller's `onSuccess` (close the modal, show a toast). Shared behavior in the declaration, per-screen behavior at the call site.

## Why not just `fetch` in `useEffect`?

Because you'd be hand-rebuilding what react-query provides, badly: loading/error state flags, caching, request deduplication (two components asking for the same data = one request), refetch-on-focus, race-condition handling when a component re-requests with new params. The `useEffect` + `useState` fetching pattern is roughly 30 lines of subtle bugs per usage. Know how to write it (interviews ask), don't ship it.

## Recap: the pattern as a checklist

Adding a new endpoint to a feature? Create `features/<feature>/api/<verb-noun>.ts` with:

- [ ] Zod schema + inferred input type (mutations) / response type (queries)
- [ ] Fetcher function using the shared `api` client
- [ ] `queryOptions` with a well-chosen `queryKey` (queries)
- [ ] `useX` hook wrapping `useQuery`/`useMutation`
- [ ] `invalidateQueries` for every cached query the mutation stales

## Try it yourself

1. Read [`update-discussion.ts`](../apps/react-vite/src/features/discussions/api/update-discussion.ts) and [`delete-discussion.ts`](../apps/react-vite/src/features/discussions/api/delete-discussion.ts) without help. You should now be able to predict their entire contents before opening them. Verify.
2. In the running app, open React Query Devtools (bottom corner). Create a discussion and watch the `['discussions']` query flip to stale and refetch. That's the `onSuccess` invalidation you just read.
3. Comment out the `invalidateQueries` call in `create-discussion.ts`, create a discussion in the UI, and observe the bug it causes. Restore it.
