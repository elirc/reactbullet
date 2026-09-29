# Junior Onboarding: 20 User Stories

This training set is designed for `apps/react-vite` first, then transferable to `nextjs-app` and `nextjs-pages`.

- Stories 1-10 include detailed implementation plans.
- Stories 11-20 include high-level implementation plans only.

---

## 1) Improve Dashboard Welcome Clarity
**User Story:** As a logged-in user, I want clearer dashboard copy so I can quickly understand what I can do in the app.

**Acceptance Criteria**
- Dashboard headline clearly identifies team collaboration context.
- Role-based capability list remains accurate for `USER` and `ADMIN`.
- No navigation or auth behavior changes.

**Likely Files**
- `apps/react-vite/src/app/routes/app/dashboard.tsx`

**Detailed Implementation Plan**
1. Open `dashboard.tsx` and identify existing role-based sections.
2. Rewrite static copy only (keep all role checks intact).
3. Add one short explanatory line above capability lists.
4. Keep conditional rendering for `ROLES.USER` and `ROLES.ADMIN` unchanged.
5. Run app and verify both roles manually (login as each role).
6. Confirm no lint/type errors introduced.

---

## 2) Show Account Creation Date in Profile
**User Story:** As a user, I want to see my account creation date in my profile so I can confirm when my account was created.

**Acceptance Criteria**
- `Created At` appears in profile details.
- Date uses existing shared formatting utility.
- Existing profile fields still render correctly.

**Likely Files**
- `apps/react-vite/src/app/routes/app/profile.tsx`
- `apps/react-vite/src/utils/format.ts` (read-only unless needed)

**Detailed Implementation Plan**
1. Open `profile.tsx`, inspect `Entry` component usage.
2. Import `formatDate` from `@/utils/format`.
3. Add a new `Entry` row for `Created At`.
4. Pass `formatDate(user.data.createdAt)` as value.
5. Verify null safety remains intact (`if (!user.data) return null`).
6. Run app and confirm formatted value appears.

---

## 3) Add Email Copy Button on Profile
**User Story:** As a user, I want a “Copy Email” button so I can quickly copy my email address.

**Acceptance Criteria**
- A button appears next to email.
- Clicking copies the logged-in email to clipboard.
- Success notification is shown.

**Likely Files**
- `apps/react-vite/src/app/routes/app/profile.tsx`
- `apps/react-vite/src/components/ui/button/button.tsx` (reuse existing)
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts` (consume, not change)

**Detailed Implementation Plan**
1. In `profile.tsx`, locate email `Entry` rendering.
2. Add a small action button near email value.
3. Implement click handler using `navigator.clipboard.writeText`.
4. Use `useNotifications` store (`addNotification`) for success feedback.
5. Add fallback notification for clipboard failure.
6. Verify button works on supported browser and does not break layout.

---

## 4) Add Author Column to Discussions Table
**User Story:** As a team member, I want to see discussion authors in the list so I can scan ownership quickly.

**Acceptance Criteria**
- Discussions list has an `Author` column.
- Author displays first + last name.
- Existing pagination and actions still work.

**Likely Files**
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx`

**Detailed Implementation Plan**
1. Open `discussions-list.tsx`, inspect current `columns` array.
2. Insert `Author` column before action columns.
3. Use a custom `Cell` renderer and safely access `entry.author`.
4. Add fallback text like `Unknown Author` for safety.
5. Verify table renders without column alignment issues.
6. Validate discussions list still supports “View” prefetch and delete action.

---

## 5) Add “No Bio Yet” Placeholder in Profile
**User Story:** As a user, I want a placeholder when bio is empty so the profile doesn’t look broken.

**Acceptance Criteria**
- Empty bio renders `No bio yet`.
- Non-empty bio renders original value.
- No API changes required.

**Likely Files**
- `apps/react-vite/src/app/routes/app/profile.tsx`

**Detailed Implementation Plan**
1. Open `profile.tsx` and find bio `Entry`.
2. Replace direct `user.data.bio` binding with a normalized value.
3. Use `const bioValue = user.data.bio?.trim() ? user.data.bio : 'No bio yet';`
4. Pass `bioValue` into `Entry`.
5. Validate with user records containing empty or whitespace bio.
6. Verify no behavior changes elsewhere.

---

## 6) Add Comment Character Counter
**User Story:** As a user writing a comment, I want a live character counter so I can control message length.

**Acceptance Criteria**
- Counter updates while typing in create-comment drawer.
- Counter resets after successful submission.
- Existing form validation remains intact.

**Likely Files**
- `apps/react-vite/src/features/comments/components/create-comment.tsx`

**Detailed Implementation Plan**
1. Open `create-comment.tsx`, locate `<Form>` render function.
2. Use form state/watch utilities available in render-prop methods.
3. Derive count from current `body` value.
4. Render counter below `<Textarea>`.
5. Ensure default body value is empty so counter starts at zero.
6. Submit comment and verify reset behavior with drawer completion flow.

---

## 7) Add “My Comments Only” Toggle
**User Story:** As a user, I want to filter comments to only mine so I can review my contributions quickly.

**Acceptance Criteria**
- Toggle appears in comments section.
- Enabled toggle filters list to comments by current user.
- Disabled toggle restores full list without refetching.

**Likely Files**
- `apps/react-vite/src/features/comments/components/comments.tsx`
- `apps/react-vite/src/features/comments/components/comments-list.tsx`
- `apps/react-vite/src/lib/auth.tsx` (consume `useUser`)

**Detailed Implementation Plan**
1. Add local toggle state in `comments.tsx`.
2. Pass toggle state to `CommentsList` via prop.
3. In `comments-list.tsx`, compute `comments` as today, then apply optional filter by `comment.author.id`.
4. Pull active user from `useUser`.
5. Keep existing loading/empty/pagination states stable.
6. Verify filtered and unfiltered mode with seeded comments.

---

## 8) Auto-Dismiss Success Notifications
**User Story:** As a user, I want success notifications to auto-dismiss so the UI stays clean.

**Acceptance Criteria**
- Success notifications disappear after timeout.
- Error notifications remain manual-dismiss.
- Manual close still works for all notifications.

**Likely Files**
- `apps/react-vite/src/components/ui/notifications/notifications.tsx`
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts`
- `apps/react-vite/src/components/ui/notifications/__tests__/notifications.test.ts`

**Detailed Implementation Plan**
1. Add optional timeout behavior at render layer (`notifications.tsx`).
2. For each success/info notification, schedule `dismissNotification(id)` with `setTimeout`.
3. Track timeout cleanup to avoid leaks on rerender/unmount.
4. Skip auto-dismiss scheduling for `error` type.
5. Add/adjust tests with fake timers.
6. Verify no duplicate-dismiss warnings in console.

---

## 9) Add Discussion “Edited At” Metadata
**User Story:** As a user, I want to see when a discussion was last edited so I can trust content freshness.

**Acceptance Criteria**
- Discussion type includes `editedAt` (nullable initially).
- Update discussion mutation sets `editedAt`.
- Detail page displays `Edited at` when present.

**Likely Files**
- `apps/react-vite/src/types/api.ts`
- `apps/react-vite/src/testing/mocks/db.ts`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts`
- `apps/react-vite/src/features/discussions/components/discussion-view.tsx`

**Detailed Implementation Plan**
1. Extend discussion type contract with `editedAt?: number | null`.
2. Add `editedAt` in mock DB model and defaults for existing entities.
3. In discussion PATCH handler, set `editedAt = Date.now()`.
4. Ensure GET list/detail handlers include `editedAt` in response mapping.
5. In `discussion-view.tsx`, render edited timestamp conditionally.
6. Validate update flow end-to-end and refresh behavior.

---

## 10) Add Pinned Discussions
**User Story:** As an admin, I want to pin critical discussions so they stay visible at the top.

**Acceptance Criteria**
- Admin can pin/unpin a discussion from list.
- Pinned discussions render above non-pinned.
- Non-admin cannot perform pin action.

**Likely Files**
- `apps/react-vite/src/types/api.ts`
- `apps/react-vite/src/testing/mocks/db.ts`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts`
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx`
- `apps/react-vite/src/features/discussions/api/*` (new pin mutation hook)
- `apps/react-vite/src/lib/authorization.tsx`

**Detailed Implementation Plan**
1. Extend discussion model/type with `isPinned`.
2. Add handler endpoint for pin toggle (admin-only).
3. Update list handler to sort pinned first, then by creation date.
4. Create new mutation hook in discussions API folder.
5. Add pin/unpin button in table row under admin authorization guard.
6. Invalidate discussions query on successful pin toggle.
7. Add integration test for admin success + user forbidden visibility.

---

## 11) Add Team Description in Registration “Join Team” Selector
**User Story:** As a registering user, I want to see team descriptions when choosing a team so I can join the right one.

**Acceptance Criteria**
- Team selector labels include name and short description.
- Existing registration flow remains valid.

**Likely Files**
- `apps/react-vite/src/features/auth/components/register-form.tsx`
- `apps/react-vite/src/features/teams/api/get-teams.ts`

**High-Level Implementation Plan**
1. Enhance select option labels to include description.
2. Truncate description for readability.
3. Validate with teams loaded via existing query.

---

## 12) Add “Load More Discussions” Mode (Optional List Variant)
**User Story:** As a user, I want an alternative to pagination links so I can load more discussions inline.

**Acceptance Criteria**
- A toggle allows switching between pagination and load-more mode.
- Load-more mode appends additional discussions.

**Likely Files**
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx`
- `apps/react-vite/src/features/discussions/api/get-discussions.ts`

**High-Level Implementation Plan**
1. Introduce a list mode state.
2. Add infinite query variant for discussions.
3. Render appended rows in load-more mode while preserving existing pagination mode.

---

## 13) Add Protected “Team Overview” Route
**User Story:** As a logged-in user, I want a team overview page so I can see team-level summary info.

**Acceptance Criteria**
- New protected route under `/app`.
- Route appears in sidebar navigation.
- Route uses existing layout shell.

**Likely Files**
- `apps/react-vite/src/config/paths.ts`
- `apps/react-vite/src/app/router.tsx`
- `apps/react-vite/src/components/layouts/dashboard-layout.tsx`
- `apps/react-vite/src/app/routes/app/*` (new route file)

**High-Level Implementation Plan**
1. Add path constant.
2. Add lazy route in router children.
3. Add nav item in dashboard layout.
4. Implement route UI with `ContentLayout`.

---

## 14) Add Discussion Search by Title
**User Story:** As a user, I want to search discussions by title so I can find threads quickly.

**Acceptance Criteria**
- Search input filters discussions by title text.
- Filter applies without full page reload.

**Likely Files**
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx`
- `apps/react-vite/src/features/discussions/api/get-discussions.ts`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts`

**High-Level Implementation Plan**
1. Add search query param to discussions fetcher.
2. Update handler to filter by title.
3. Add search field in list UI and sync with query state.

---

## 15) Add Markdown Preview Toggle in Comment Composer
**User Story:** As a user, I want to preview my markdown comment before submitting so formatting is predictable.

**Acceptance Criteria**
- Composer supports edit/preview toggle.
- Preview uses existing markdown preview component.

**Likely Files**
- `apps/react-vite/src/features/comments/components/create-comment.tsx`
- `apps/react-vite/src/components/ui/md-preview/md-preview.tsx`

**High-Level Implementation Plan**
1. Add local preview mode toggle.
2. Show textarea in edit mode and `MDPreview` in preview mode.
3. Keep form submission bound to body content.

---

## 16) Add Role Badge Styling in Users Table
**User Story:** As an admin, I want visually distinct role badges so I can quickly distinguish admins from users.

**Acceptance Criteria**
- Role column displays styled badges.
- Badge styling differs for admin/user.

**Likely Files**
- `apps/react-vite/src/features/users/components/users-list.tsx`
- `apps/react-vite/src/components/ui/*` (reuse existing style patterns)

**High-Level Implementation Plan**
1. Replace raw role text with a small badge component inline.
2. Map role to style variants.
3. Verify readability on mobile and desktop.

---

## 17) Add “Duplicate Discussion” Action (Admin)
**User Story:** As an admin, I want to duplicate a discussion so I can reuse structure for recurring topics.

**Acceptance Criteria**
- Admin can trigger duplicate action from discussion detail.
- New discussion preserves title/body with suffix.

**Likely Files**
- `apps/react-vite/src/features/discussions/components/discussion-view.tsx`
- `apps/react-vite/src/features/discussions/api/create-discussion.ts`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts`

**High-Level Implementation Plan**
1. Add admin-visible duplicate button.
2. Create helper that calls create-discussion mutation using current discussion data.
3. Invalidate discussions list and show success notification.

---

## 18) Add “Last Active” to Discussions List
**User Story:** As a user, I want to see when a discussion was last active so I can prioritize active threads.

**Acceptance Criteria**
- List displays last activity time per discussion.
- Value updates when new comments are added.

**Likely Files**
- `apps/react-vite/src/types/api.ts`
- `apps/react-vite/src/testing/mocks/handlers/discussions.ts`
- `apps/react-vite/src/testing/mocks/handlers/comments.ts`
- `apps/react-vite/src/features/discussions/components/discussions-list.tsx`

**High-Level Implementation Plan**
1. Add `lastActiveAt` concept to discussion response model.
2. Update comments create path to mutate discussion’s last active timestamp.
3. Render new column in discussions table.

---

## 19) Add Soft Delete for Comments
**User Story:** As an admin, I want comment deletion to be soft-delete so audit history can be preserved.

**Acceptance Criteria**
- Delete marks comment as removed, does not physically remove record.
- UI shows “Deleted comment” placeholder text.
- Delete permission rules remain unchanged.

**Likely Files**
- `apps/react-vite/src/testing/mocks/db.ts`
- `apps/react-vite/src/testing/mocks/handlers/comments.ts`
- `apps/react-vite/src/features/comments/components/comments-list.tsx`
- `apps/react-vite/src/features/comments/api/delete-comment.ts`

**High-Level Implementation Plan**
1. Add soft-delete field on comment model.
2. Update delete handler to mark deleted state.
3. Update comments list renderer to show placeholder for deleted rows.
4. Keep query invalidation behavior as-is.

---

## 20) Add Optimistic UI for Comment Creation
**User Story:** As a user, I want comments to appear instantly after submit so the app feels fast.

**Acceptance Criteria**
- New comment appears immediately before server confirmation.
- Failed request rolls back optimistic comment and shows error notification.

**Likely Files**
- `apps/react-vite/src/features/comments/api/create-comment.ts`
- `apps/react-vite/src/features/comments/api/get-comments.ts`
- `apps/react-vite/src/features/comments/components/comments-list.tsx`
- `apps/react-vite/src/components/ui/notifications/notifications-store.ts`

**High-Level Implementation Plan**
1. Add `onMutate` optimistic cache update for comment list.
2. Add rollback in `onError` using cached snapshot.
3. Keep `onSettled` invalidate to sync server truth.

