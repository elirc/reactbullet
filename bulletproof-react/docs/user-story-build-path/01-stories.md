## Story 1: Clearer Dashboard Role Summary
**Difficulty:** Easy  
**Estimated Time:** 1 hour  
**Skills You'll Practice:** reading route components, small UI edits, acceptance-driven checks

**The Story:**  
As a signed-in user, I want a clearer role summary on the dashboard so that I understand what actions I can take immediately.

**Acceptance Criteria:**
- [ ] Dashboard shows a dedicated “You can do this” section title.
- [ ] Role-specific bullets remain correct for `USER` and `ADMIN`.
- [ ] Existing navigation and layout behavior is unchanged.

**Files You'll Likely Touch:**
- `apps/react-vite/src/app/routes/app/dashboard.tsx` - existing role-based messaging lives here.

**High-Level Implementation Plan:**
1. Update copy and structure in `dashboard.tsx` around current role conditional blocks.
2. Keep role branching logic intact (`user.data?.role === ROLES.USER/ADMIN`).
3. Manually verify both role branches by testing with two users.

**Tips:**
- Reuse existing role constants from `@/lib/authorization`.
- Avoid touching route config; this is a pure presentation task.
- Keep text changes scoped to avoid noisy diff.

**What Could Go Wrong:**
- Accidentally removing one role branch.
- Introducing JSX nesting mistakes that break rendering.

**Stretch Goal:**  
Add one-line contextual help below each role list explaining where to find those actions.

**Connects To:**  
Story 2, because both are low-risk route-level UI refinements.

---

## Story 2: Profile Metadata Expansion
**Difficulty:** Easy  
**Estimated Time:** 1.5 hours  
**Skills You'll Practice:** component extraction reading, route UI updates, formatting utilities

**The Story:**  
As a user, I want to see my account creation date on the profile page so that I can confirm account age.

**Acceptance Criteria:**
- [ ] Profile page displays `Created At` using the app’s date formatting utility.
- [ ] Existing profile fields still render.
- [ ] No API contract changes are required.

**Files You'll Likely Touch:**
- `apps/react-vite/src/app/routes/app/profile.tsx` - displays profile entries.
- `apps/react-vite/src/utils/format.ts` - existing date formatting helper (read-only unless enhancement needed).

**High-Level Implementation Plan:**
1. Import and use `formatDate` in `profile.tsx`.
2. Add a new `Entry` row for `Created At`.
3. Verify with seeded/mock user data.

**Tips:**
- Reuse the existing `Entry` helper component in `profile.tsx`.
- Keep date formatting consistent with discussions/comments lists.
- Don’t duplicate formatting logic inline.

**What Could Go Wrong:**
- Passing a non-number value into `formatDate`.
- Displaying raw timestamp due missing formatter call.

**Stretch Goal:**  
Show relative age text (for example, “Member for X days”) next to formatted date.

**Connects To:**  
Story 3, where you will modify another existing feature UI component.

---

## Story 3: Comment Composer Character Feedback
**Difficulty:** Easy  
**Estimated Time:** 2 hours  
**Skills You'll Practice:** form UX polish, controlled field observation, small component updates

**The Story:**  
As a user, I want to see live character count while typing a comment so that I can keep comments concise.

**Acceptance Criteria:**
- [ ] Comment form shows live character count below textarea.
- [ ] Count resets after successful submission.
- [ ] Existing validation and submit behavior still work.

**Files You'll Likely Touch:**
- `apps/react-vite/src/features/comments/components/create-comment.tsx` - comment creation form.
- `apps/react-vite/src/components/ui/form/textarea.tsx` - optional if adding standardized helper text support.

**High-Level Implementation Plan:**
1. Read current `Form` render-prop usage in `create-comment.tsx`.
2. Wire character count using form state value observation.
3. Render count text near textarea.
4. Verify count resets with existing `isDone` drawer flow.

**Tips:**
- Stay inside existing `Form` pattern used throughout features.
- Avoid adding new global state for this local concern.
- Confirm count updates for pasted text too.

**What Could Go Wrong:**
- Count not updating due stale closure.
- Count persisting between drawer opens unexpectedly.

**Stretch Goal:**  
Add soft warning style when comment exceeds a chosen threshold.

**Connects To:**  
Story 4, where you’ll improve data-rich table presentation.

---

## Story 4: Discussion List Author Column
**Difficulty:** Medium  
**Estimated Time:** 2.5 hours  
**Skills You'll Practice:** table composition, typed column definitions, feature UI data binding

**The Story:**  
As a team member, I want to see who authored each discussion in the list so that I can navigate conversations faster.

**Acceptance Criteria:**
- [ ] Discussions table includes an `Author` column.
- [ ] Author displays `firstName lastName`.
- [ ] Existing pagination and view/delete actions still work.

**Files You'll Likely Touch:**
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx` - table column definitions.
- `apps/react-vite/src/features/discussions/api/get-discussions.ts` - confirm data includes `author`.
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts` - confirm list response includes `author`.

**High-Level Implementation Plan:**
1. Confirm `author` exists in `Discussion` type and list payload.
2. Add a new table column in `discussions-list.tsx` with safe rendering for missing author.
3. Validate sorting/pagination unaffected.

**Tips:**
- Use `Cell` render function pattern already used for date/action columns.
- Keep fallback text for rare missing author object.
- Don’t alter query key or pagination logic.

**What Could Go Wrong:**
- Runtime error if author is unexpectedly null/empty object.
- Misaligned table columns on narrow screens.

**Stretch Goal:**  
Make author name clickable to open profile (or placeholder action).

**Connects To:**  
Story 5, where you’ll add richer list behavior on existing comment data.

---

## Story 5: Comments “Only Mine” Toggle
**Difficulty:** Medium  
**Estimated Time:** 3 hours  
**Skills You'll Practice:** local state composition, filtered rendering, authorization-aware UI

**The Story:**  
As a user, I want a toggle to show only my comments so that I can quickly review my contributions.

**Acceptance Criteria:**
- [ ] A toggle appears in comments section.
- [ ] When enabled, list shows only comments authored by current user.
- [ ] Toggle off restores full list without refetch.

**Files You'll Likely Touch:**
- `apps/react-vite/src/features/comments/components/comments.tsx` - toggle placement.
- `apps/react-vite/src/features/comments/components/comments-list.tsx` - filtering logic.
- `apps/react-vite/src/lib/auth.tsx` - existing `useUser` source (read for usage pattern).

**High-Level Implementation Plan:**
1. Add local toggle state in `comments.tsx`.
2. Pass toggle state as prop into `CommentsList`.
3. Filter flattened comment array in `comments-list.tsx` when toggle enabled.
4. Keep pagination and load-more behavior sensible with filtered view.

**Tips:**
- Use current user id from `useUser`.
- Filter after data flattening to preserve paging compatibility.
- Keep default behavior unchanged when toggle is false.

**What Could Go Wrong:**
- Filtering before flattening can break pagination assumptions.
- Non-authenticated edge cases causing null user access.

**Stretch Goal:**  
Persist toggle preference in URL query string.

**Connects To:**  
Story 6, where you’ll introduce cross-component UX behavior via shared state.

---

## Story 6: Auto-Dismiss Notifications
**Difficulty:** Medium  
**Estimated Time:** 4 hours  
**Skills You'll Practice:** Zustand store evolution, timed side effects, UI feedback lifecycle

**The Story:**  
As a user, I want success notifications to disappear automatically so that the screen stays uncluttered.

**Acceptance Criteria:**
- [ ] Success/info notifications auto-dismiss after a timeout.
- [ ] Error notifications remain until manually dismissed.
- [ ] Manual close still works for all notification types.

**Files You'll Likely Touch:**
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts` - notification state model.
- `apps/react-vite/src/components/ui/notifications/notifications.tsx` - render list and timer orchestration.
- `apps/react-vite/src/components/ui/notifications/notification.tsx` - close interaction compatibility.
- `apps/react-vite/src/components/ui/notifications/__tests__/notifications.test.ts` - extend tests.

**High-Level Implementation Plan:**
1. Extend notification model with optional timeout metadata.
2. Add effect in `Notifications` list renderer to schedule dismiss for non-error types.
3. Ensure cleanup on unmount/re-render.
4. Add/update tests for timed dismiss and manual dismiss behavior.

**Tips:**
- Keep timer logic close to rendering layer, not inside store mutators.
- Prevent duplicate timers for same notification id.
- Use fake timers in tests for deterministic behavior.

**What Could Go Wrong:**
- Memory leaks from uncleared timers.
- Race between manual dismiss and timer callback.
- Unexpected dismissal of errors.

**Stretch Goal:**  
Support per-notification custom duration.

**Connects To:**  
Story 7, where you’ll add a new API endpoint and integrate new state data.

---

## Story 7: Discussion Stats Endpoint + UI Badge
**Difficulty:** Medium  
**Estimated Time:** 5 hours  
**Skills You'll Practice:** adding endpoint, query hook creation, UI integration

**The Story:**  
As a user, I want to see the comment count for a discussion so that I can prioritize active threads.

**Acceptance Criteria:**
- [ ] New endpoint returns discussion stats including comment count.
- [ ] Discussion detail page shows comment count badge.
- [ ] Stats refresh after creating/deleting comments.

**Files You'll Likely Touch:**
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts` - add stats route.
- `apps/react-vite/src/features/discussions/api/get-discussion.ts` or new `get-discussion-stats.ts` - new query hook.
- `apps/react-vite/src/app/routes/app/discussions/discussion.tsx` - prefetch/use stats.
- `apps/react-vite/src/features/comments/api/create-comment.ts` - invalidate stats query.
- `apps/react-vite/src/features/comments/api/delete-comment.ts` - invalidate stats query.

**High-Level Implementation Plan:**
1. Define stats response shape and add handler endpoint (for example `/discussions/:id/stats`).
2. Add API fetcher + query options + hook in discussions API module.
3. Integrate query into discussion route and render badge.
4. Invalidate/refetch stats on comment create/delete.
5. Add at least one integration test validating badge updates.

**Tips:**
- Keep stats query key distinct from discussion detail key.
- Reuse existing auth/team checks in handler.
- Avoid computing comment count in UI from paginated data.

**What Could Go Wrong:**
- Stats endpoint not team-scoped.
- Badge stale due missing invalidation.
- Query key collision with existing discussion query.

**Stretch Goal:**  
Show “Last active” timestamp using latest comment time.

**Connects To:**  
Story 8, where you’ll introduce a new persisted domain concept.

---

## Story 8: Discussion Tags (Full-Stack)
**Difficulty:** Hard  
**Estimated Time:** 8 hours  
**Skills You'll Practice:** domain model evolution, form updates, handler persistence, list/detail rendering

**The Story:**  
As an admin, I want to tag discussions so that teams can categorize conversations by topic.

**Acceptance Criteria:**
- [ ] Discussion create/update supports tags input.
- [ ] Tags persist and appear in discussions list and detail.
- [ ] Existing discussion CRUD and permissions still work.

**Files You'll Likely Touch:**
- `apps/react-vite/src/types/api.ts` - extend `Discussion` contract.
- `apps/react-vite/src/testing/mocks/db.ts` - add `tags` field in discussion model.
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts` - ensure tags handled in create/update/list/detail.
- `apps/react-vite/src/features/discussions/api/create-discussion.ts` - schema and type updates.
- `apps/react-vite/src/features/discussions/api/update-discussion.ts` - schema and type updates.
- `apps/react-vite/src/features/discussions/components/create-discussion.tsx` - form field.
- `apps/react-vite/src/features/discussions/components/update-discussion.tsx` - form field.
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx` - tag preview column.
- `apps/react-vite/src/features/discussions/components/discussion-view.tsx` - tag display.

**High-Level Implementation Plan:**
1. Introduce `tags: string[]` in discussion type and mock DB model.
2. Update create/update Zod schemas and payload handling.
3. Parse user tag input into normalized array.
4. Render tags in list and detail views.
5. Add/update tests for tag persistence and display.

**Tips:**
- Normalize whitespace/case when parsing tags.
- Keep backward compatibility for existing discussions without tags.
- Reuse existing form primitives; do not invent a custom modal.

**What Could Go Wrong:**
- Breaking existing records that lack `tags`.
- Saving empty strings as tags.
- Forgetting to include tags in update default values.

**Stretch Goal:**  
Add clickable tag filter on discussions page.

**Connects To:**  
Story 9, where metadata drives ordering and prioritization.

---

## Story 9: Pin Discussions (Admin Priority Feed)
**Difficulty:** Hard  
**Estimated Time:** 9 hours  
**Skills You'll Practice:** policy-protected mutations, sorting strategy, cross-view consistency

**The Story:**  
As an admin, I want to pin important discussions so that they appear first for everyone on the team.

**Acceptance Criteria:**
- [ ] Admin can pin/unpin a discussion.
- [ ] Pinned discussions sort above unpinned discussions in list.
- [ ] Non-admin users cannot pin/unpin.

**Files You'll Likely Touch:**
- `apps/react-vite/src/types/api.ts` - add `isPinned` to discussion contract.
- `apps/react-vite/src/testing/mocks/db.ts` - add `isPinned` field.
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts` - add pin/unpin endpoint and list sorting.
- `apps/react-vite/src/features/discussions/api/get-discussions.ts` - ensure sorted result consumption remains valid.
- `apps/react-vite/src/features/discussions/api/update-discussion.ts` or new `toggle-pin-discussion.ts` - mutation hook.
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx` - pin indicator + action.
- `apps/react-vite/src/lib/authorization.tsx` - optional policy helper for pin action.
- `apps/react-vite/src/app/routes/app/discussions/__tests__/discussions.test.tsx` - update coverage.

**High-Level Implementation Plan:**
1. Extend discussion model/type with `isPinned`.
2. Add handler mutation route (admin-only) for pin toggling.
3. Update list handler query to sort pinned first then created date.
4. Add UI control in list for admins and indicator for all users.
5. Add mutation invalidation for discussions list.
6. Add tests for admin success and user forbidden behavior.

**Tips:**
- Keep pin action separate from full update mutation if possible.
- Ensure sorting happens server-side to keep behavior consistent.
- Use existing `Authorization` pattern for UI-level control.

**What Could Go Wrong:**
- UI indicates pinned but list not re-sorted due stale cache.
- Non-admin can trigger mutation endpoint.
- Sorting logic unstable across pages.

**Stretch Goal:**  
Persist pinned filter preference per user.

**Connects To:**  
Story 10, where performance and architecture choices become central.

---

## Story 10: Optimistic Comment UX With Safe Rollback
**Difficulty:** Expert  
**Estimated Time:** 12 hours  
**Skills You'll Practice:** optimistic caching strategy, rollback design, error UX, architectural tradeoff analysis

**The Story:**  
As a user, I want comment create/delete actions to feel instant so that collaboration feels real-time.

**Acceptance Criteria:**
- [ ] Comment create appears immediately (optimistic) before server response.
- [ ] Comment delete removes immediately (optimistic) before server response.
- [ ] Failed mutations rollback cleanly and show clear error notification.
- [ ] Pagination/infinite query cache remains coherent after optimistic updates.

**Files You'll Likely Touch:**
- `apps/react-vite/src/features/comments/api/create-comment.ts` - optimistic `onMutate` + rollback context.
- `apps/react-vite/src/features/comments/api/delete-comment.ts` - optimistic remove + rollback context.
- `apps/react-vite/src/features/comments/api/get-comments.ts` - query key helpers for cache targeting.
- `apps/react-vite/src/features/comments/components/comments-list.tsx` - ensure optimistic entries render safely.
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts` - error/success feedback handling.
- `apps/react-vite/src/lib/react-query.ts` - optional helper typing improvements for optimistic context.
- `apps/react-vite/src/app/routes/app/discussions/__tests__/discussion.test.tsx` - extend integration coverage.
- `apps/react-vite/e2e/tests/smoke.spec.ts` - optional update for UX-level assertions.

**High-Level Implementation Plan:**
1. Implement `onMutate` in create/delete hooks to snapshot previous cache state.
2. Apply optimistic cache updates against `['comments', discussionId]` infinite query pages.
3. Implement `onError` rollback using snapshot context.
4. Keep `onSettled` invalidation to resync authoritative server state.
5. Ensure optimistic item identity strategy is stable (temporary id handling).
6. Add tests for success path and rollback path.
7. Document tradeoffs (latency vs complexity vs consistency).

**Tips:**
- Start with create OR delete optimistic path first, then add the second.
- Keep rollback context strongly typed.
- Avoid optimistic updates that mutate cached arrays in place.
- Verify behavior across multiple pages, not only first page.
- Preserve existing authorization behavior; optimistic UI must still respect policy.

**What Could Go Wrong:**
- Rollback context mismatch causing duplicated or lost comments.
- Cross-page cache corruption in infinite query structure.
- Temporary optimistic ids colliding with real ids.

**Stretch Goal:**  
Add lightweight conflict banner when server response materially differs from optimistic state.

**Connects To:**  
No further story; this is the ownership-level capstone.

