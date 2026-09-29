# 🔐 Security

Two separate questions, constantly confused, worth keeping apart forever:

- **Authentication** — *who are you?* (login, tokens, sessions)
- **Authorization** — *what are you allowed to do?* (roles, permissions)

One rule frames this whole chapter: **client-side security is UX, not security.** Anything enforced only in the browser can be bypassed by anyone with DevTools. Hiding a delete button doesn't protect the delete endpoint — the *server* must check every request. The client-side checks below exist to give legitimate users an honest UI, not to stop attackers.

## Authentication

### How SPAs authenticate

On login/register the server issues a token (commonly a [JWT](https://jwt.io/)); the client includes it with every subsequent request (header or cookie); the server validates it per request.

The token has to survive page refresh, so it must be stored somewhere persistent. Your two options, and the trade-off every frontend interview asks about:

| Storage | Risk | Notes |
|---------|------|-------|
| `localStorage` / `sessionStorage` | **XSS** — any injected script can read it and exfiltrate the token | Simple, but you're one XSS bug away from stolen sessions |
| Cookie with `HttpOnly` | CSRF (mitigated with `SameSite` + CSRF tokens) | JavaScript *cannot read it at all* — XSS can't steal what JS can't see |

**Prefer `HttpOnly` cookies.** Only the server can set `HttpOnly` (that's the point), which is why the client's job reduces to `withCredentials = true` on the API client so cookies ride along on requests — see [`api-client.ts`](../apps/react-vite/src/lib/api-client.ts). (This demo app simulates the cookie with `js-cookie` in MSW handlers because there's no real server; in production the API sets it.)

### XSS, since it's the threat model

Cross-Site Scripting ([XSS](https://owasp.org/www-community/attacks/xss/)) = attacker-controlled content executing as script in your users' browsers. React escapes rendered strings by default — you inherit a lot of protection — but the escape hatches are where apps get burned: `dangerouslySetInnerHTML`, and rendering user-supplied HTML/markdown.

This app renders user-written markdown (discussion bodies), which is exactly such a hatch. Look at [`md-preview.tsx`](../apps/react-vite/src/components/ui/md-preview/md-preview.tsx): the markdown output is run through **DOMPurify** before being injected. The rule: **any user content that bypasses React's escaping must be sanitized first.** Full client-side risk list: [OWASP Top 10 client-side](https://owasp.org/www-project-top-10-client-side-security-risks/).

### The user object as auth state

"Is someone logged in, and who?" is global state — but it's *server* state (the server knows; you're caching it). So this repo handles it with react-query via [`react-query-auth`](https://github.com/alan2207/react-query-auth) — see [`src/lib/auth.tsx`](../apps/react-vite/src/lib/auth.tsx), which wires four things together (`useUser`, `useLogin`, `useRegister`, `useLogout`) and exposes them app-wide.

The convention throughout the app: **user object present ⇒ authenticated.** Protected routes just check it — see the redirect-to-login logic in the same file — and after login, users are returned to the page they originally wanted (`redirectTo`).

## Authorization

The app's rules from [chapter 1](01-application-overview.md): admins manage everything; users manage only their own comments. Two mechanisms implement this, both in [`src/lib/authorization.tsx`](../apps/react-vite/src/lib/authorization.tsx) — read it, it's ~80 lines and worth every one.

### RBAC — role-based access control

Coarse-grained: access depends only on the user's role.

```tsx
const useAuthorization = () => {
  // ...
  const checkAccess = ({ allowedRoles }: { allowedRoles: RoleTypes[] }) => {
    if (allowedRoles && allowedRoles.length > 0 && user.data) {
      return allowedRoles.includes(user.data.role);
    }
    return true;
  };
  return { checkAccess, role: user.data.role };
};
```

Used declaratively via the `Authorization` component — e.g. only admins see the delete-discussion button ([`delete-discussion.tsx`](../apps/react-vite/src/features/discussions/components/delete-discussion.tsx)):

```tsx
<Authorization allowedRoles={[ROLES.ADMIN]}>
  <ConfirmationDialog ... />
</Authorization>
```

### PBAC — permission (policy) based access control

Some rules can't be answered by role alone: *"can this user delete this comment?"* depends on **which comment**. That's a policy — a function of the user *and* the resource:

```ts
export const POLICIES = {
  'comment:delete': (user: User, comment: Comment) => {
    if (user.role === 'ADMIN') return true;
    if (user.role === 'USER' && comment.author?.id === user.id) return true;
    return false;
  },
};
```

Used by passing the evaluated policy to the same component ([`comments-list.tsx`](../apps/react-vite/src/features/comments/components/comments-list.tsx)):

```tsx
<Authorization policyCheck={POLICIES['comment:delete'](user.data, comment)}>
  <DeleteComment id={comment.id} />
</Authorization>
```

**How to choose:** if the rule mentions only the role → RBAC. If it mentions the resource ("own comment", "their team") → PBAC. Centralizing policies in one `POLICIES` object means the rules live in one auditable place instead of scattered `if` statements.

One elegant detail worth studying in `authorization.tsx`: the `AuthorizationProps` type uses a **discriminated union** so callers must pass *either* `allowedRoles` *or* `policyCheck`, never both — TypeScript enforcing correct API usage at compile time. Steal this technique.

## Common junior mistakes

1. **Treating hidden UI as protection.** Repeat: the server must check every request. Ask your backend teammates where each rule is enforced server-side.
2. **Storing JWTs in `localStorage` by default** because tutorials do. Know the trade-off; prefer `HttpOnly` cookies when the backend supports them.
3. **Rendering user HTML/markdown without sanitizing.** Grep for `dangerouslySetInnerHTML` in any codebase you join.
4. **Scattering permission `if`s across components** instead of centralizing policies. Six months later nobody can answer "who can delete a comment?"

## Try it yourself

1. Log in as a `USER` and use React DevTools to render a delete button anyway (or hit the endpoint directly from the console). In this MSW-mocked app the "server" ([`handlers/comments.ts`](../apps/react-vite/src/testing/mocks/handlers/comments.ts)) checks too — confirm the request fails. That's client checks as UX, server checks as security, in one experiment.
2. Add a hypothetical policy `'discussion:edit'` (admins, or the discussion's author) to `POLICIES`. Notice you can write and reason about it without touching any component.
3. Explain to a rubber duck why `HttpOnly` specifically defeats token theft via XSS, but doesn't defeat XSS itself. (What can injected script still do while the user's tab is open?)
