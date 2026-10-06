# Plan: issue #67 — Review creation does not verify profile ownership

## Diagnosis

`create_review()` in `core/services/review_service.py` receives both `profile_id` and `user_id`, but only uses `profile_id` to build the `Review` row — `user_id` is accepted and never read again. This is grounded directly in my Unit 2 reproduction (posted at https://github.com/codepath/pathreview-ai301-fa26-s3/issues/67#issuecomment-5882872048): calling `create_review(db, profile_id=<a profile belonging to a different user>, user_id=<caller>)` constructs and commits a `Review` with **zero** `db.execute` calls — meaning no query against the `profiles` table ever runs, so there is nothing in the function that could check ownership even in principle. As a control in that same reproduction, `get_review()` in the same file issues a real `SELECT ... FROM reviews JOIN profiles ON profiles.id = reviews.profile_id WHERE ... AND profiles.user_id = :user_id_1` — proving the ownership-check pattern exists, works, and is simply absent from `create_review()`.

## Scope

One bounded change: add the same ownership check `get_review()`/`list_reviews()` already perform, to `create_review()`, before it constructs a `Review`.

In scope:
- `core/services/review_service.py`'s `create_review()` — verify the given `profile_id` belongs to `user_id` before creating a review.
- `api/routes/reviews.py`'s `create_review_endpoint()` — translate "profile not found or not owned" into an HTTP error response, the same way `get_review_endpoint()` already does for a missing/not-owned review.

Not in scope:
- `get_review()` and `list_reviews()` — already correct, not touched.
- `process_review()` (the background task) — it only runs after a `Review` row already exists; once `create_review()` is fixed, a `Review` can no longer exist for a profile the caller doesn't own, so there's nothing left for this background path to additionally check. Not re-auditing it here.
- Any broader authorization framework/refactor — this is one missing check in one function, not a design problem.

## Files

- `core/services/review_service.py` (`create_review`)
- `api/routes/reviews.py` (`create_review_endpoint`)
- `tests/unit/test_review_service.py` (new regression test for the mismatched-owner case)

## Approach

1. In `create_review()`, before constructing the `Review`, query for the profile: `select(Profile).where(Profile.id == profile_id, Profile.user_id == user_id)`, and check whether a row comes back — the same shape of check `get_review()` already uses via its join, just as a standalone lookup since there's no `Review` row yet to join through.
2. If no matching profile is found (wrong owner, or `profile_id` simply doesn't exist), `create_review()` returns `None` instead of creating a `Review` — mirroring `get_review()`'s existing convention of returning `None` for "not found or not yours," rather than raising inside the service layer.
3. In `create_review_endpoint()`, if `create_review()` returns `None`, raise `HTTPException(status.HTTP_404_NOT_FOUND, detail="Profile not found")` — matching `get_review_endpoint()`'s existing 404-for-not-found/not-owned convention (not a 403; the codebase's existing pattern already treats "not yours" the same as "doesn't exist," which avoids confirming to a caller that a given profile ID exists under someone else's account).
4. Add a unit test to `tests/unit/test_review_service.py`: `create_review()` called with a `profile_id`/`user_id` pair where the mocked profile lookup returns no row returns `None` and never constructs a `Review`; a matching pair still constructs and commits one as before.

## Test plan

Re-run my Unit 2 reproduction script (`repro_issue_67.py`, committed on my `repro/67-review-ownership` branch) against the fixed code:

- **Before** (already posted in Unit 2): `create_review(db, profile_id=profile_of_b, user_id=user_a)` → `db.execute` called 0 times, `Review` constructed with `profile_id=profile_of_b` anyway.
- **After** (expected): the same call → `db.execute` called at least once (the new ownership lookup), and the function returns `None` — no `Review` constructed for a profile `user_a` doesn't own.
- **Control, unchanged**: `create_review(db, profile_id=<user_a's own profile>, user_id=user_a)` still returns a constructed, committed `Review`, exactly as before the fix — the fix must not break the legitimate case.

## Risks and unknowns

- `create_review()` is called from exactly one place in the codebase (`api/routes/reviews.py`'s `create_review_endpoint`, confirmed by grep) plus its own test file, so changing its return contract (now `Review | None`) has one call site to update. I will double check this again at build time in case anything changed since this plan was written.
- Choosing 404 over 403 for the ownership-mismatch case is a convention-matching decision (following `get_review_endpoint()`'s existing precedent), not something the issue itself specifies — I'll flag this explicitly in the PR in case a maintainer prefers otherwise.
- No maintainer has commented on this issue yet (as of this plan), so there's no existing direction to reconcile this approach against; if one appears before I open the PR, I'll revisit this plan and note any deviation below.

## Deviations

The build matched the plan closely: `create_review()` now runs the exact `select(Profile).where(Profile.id == profile_id, Profile.user_id == user_id)` lookup described above and returns `None` on a miss; `create_review_endpoint()` raises the planned 404. One thing the plan didn't anticipate: adding the ownership check meant `create_review()` now calls `db.execute()` for the first time, which six pre-existing unit tests (testing unrelated things like "does it call `db.commit()`") didn't account for in their shared mock fixture — they'd relied on `create_review()` never touching the database read path. Rather than editing each of the six individually, I updated the shared `mock_db_session` fixture in `tests/unit/test_review_service.py` once, so it returns a result with a truthy `scalars().first()` by default (profile found/owned), which those six tests now pass through unaffected. My two new regression tests (for the not-owned and owned cases) override this default explicitly, so they aren't affected by it. I verified this doesn't touch the file's existing `xfail(strict=True)` tests for the separate, already-known issue #65 (they each override `mock_db_session.execute` themselves inside their own test body), and confirmed by running the full unit suite: 377 passed, 53 xfailed, same xfail count as before my change.
