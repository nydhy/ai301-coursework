# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

nydhy

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/67#issuecomment-6023438247

Here's my plan, building on the reproduction I posted above: `create_review()` in `core/services/review_service.py` never checks the given `profile_id` against `user_id` before creating a `Review` — my repro showed zero DB queries run during that call, while `get_review()` in the same file does run a real `profiles.user_id`-scoped query.

Proposed approach: add the same ownership lookup to `create_review()` (a `select(Profile).where(Profile.id == profile_id, Profile.user_id == user_id)` check before constructing the row), returning `None` when it doesn't match — mirroring `get_review()`'s existing "not found or not yours" convention — and have `create_review_endpoint()` turn that into a 404, matching `get_review_endpoint()`'s existing pattern. One open question I'd like feedback on: whether 404 (vs. 403) is the right call here, or just following precedent from the rest of this file.

Test plan: re-run my reproduction script before/after the fix — before, 0 ownership queries and a review gets created anyway; after, the query runs and a mismatched-owner call returns nothing, while a legitimate same-owner call still works unchanged.

Scope-wise, I'm keeping this to `create_review()` and its route handler only — `get_review()`/`list_reviews()` are already correct, and I'm not touching anything else. Open to any direction before I start building.

---

## Your branch

**Branch**

fix/67-profile-ownership-check

**Evidence**

My original Unit 2 reproduction fully mocked the database session, so it couldn't run unmodified against the real fixed code (it would just hit an unconfigured mock and crash, not demonstrate the fix). Per the assignment's guidance for that case, I turned its inputs into a real check that runs through the actual `create_review()` code: the two regression tests added in this build (`test_create_review_returns_none_when_profile_not_owned`, `test_create_review_creates_review_when_profile_owned`), run once against the pre-fix code and once against the post-fix code.

**Before** (pre-fix code, via a temporary `git worktree` at the commit prior to my fix):

```
$ python -m pytest tests/unit/test_review_service.py -k "not_owned or creates_review_when_profile_owned" -v
tests/unit/test_review_service.py::TestReviewService::test_create_review_returns_none_when_profile_not_owned FAILED
tests/unit/test_review_service.py::TestReviewService::test_create_review_creates_review_when_profile_owned PASSED

FAILED tests/unit/test_review_service.py::TestReviewService::test_create_review_returns_none_when_profile_not_owned
AssertionError: assert <MagicMock name='Review()' id='4638913552'> is None
1 failed, 1 passed, 19 deselected, 1 warning in 0.60s
```

(The failure itself is the bug: `create_review()` constructs a `Review` for a mismatched-owner `profile_id` instead of returning `None`.)

**After** (current code, on `fix/67-profile-ownership-check`):

```
$ python -m pytest tests/unit/test_review_service.py -k "not_owned or creates_review_when_profile_owned" -v
tests/unit/test_review_service.py::TestReviewService::test_create_review_returns_none_when_profile_not_owned PASSED
tests/unit/test_review_service.py::TestReviewService::test_create_review_creates_review_when_profile_owned PASSED

2 passed, 19 deselected, 1 warning in 0.43s
```

I also ran the full unit suite on the fixed code to confirm no regressions: `377 passed, 53 xfailed` — the same `xfail` count as before my change (those are pre-existing, unrelated to issue #67), with 0 unexpected failures.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. `--limit 4` smoke run (pkg-01 through pkg-04): 4/4
2. Full confirming run, `--save-run eval-run.txt`: **19/20 scored items (bar: 18/20: PASS)**, category floor clear on all five categories (`clear-accept 6/7`, `scope-creep 4/4`, `thread-convention 2/2`, `unbuildable 3/3`, `wrong-cause 4/4`) — this is the run committed in `eval-run.txt`.

**Package analysis**

`pkg-14`. Gold label: `accept` ("honestly scoped-down: reattach handshake fix with a regression-window repro; defers the untestable Windows variant and says so; arguable on the deferral, ready as scoped"). My rubric's verdict: `reject`, failing `plan-executable`. The plan's Files section names the right areas — "the client attach/reattach path in `zellij-server` (session connection handling) and `zellij-client`'s terminal query issuance" — and a concrete approach (drain pending OSC query responses before pane input is wired, bounded by pattern rather than a time window), but says "exact functions to be pinned in the PR after tracing the query issuance with debug logs" instead of naming them outright. My rubric's `plan-executable` pass condition requires the plan to name concrete files or code locations such that a stranger could start executing immediately, and read "exact functions to be pinned... after tracing" as a deferred decision rather than a committed one, even though the plan already says the tracing is done and working ("which I have working") and names the two modules precisely.

**Check rationale**

From `rubric.md`, the `plan-executable` row's pass condition, quoted exactly as it currently reads: "Pass if the plan names concrete files or code locations and a concrete, ordered sequence of steps that a stranger could start executing immediately without asking the author anything. Fail if a real decision is deferred to build time ('somewhere', 'upstream or vendored, whichever is easier', no chosen files or layer named, 'investigate X' standing in for an actual approach)." I wrote this wording to catch plans like `pkg-17` and `pkg-18` (unbuildable), where the entire approach is an open question ("gocui? tcell? not sure", "recover() somewhere, fix upstream or vendored, whichever is easier") with no files committed at all. The same "no real decision deferred" standard is what then reads `pkg-14`'s one still-open detail (exact function names, as opposed to the already-named files and already-decided approach) as falling short, even though nothing else about `pkg-14`'s plan is actually vague.

**Trade-offs**

Loosening `plan-executable` to accept "named modules plus a committed approach" even when exact function names are left for the PR would flip `pkg-14` to agree with gold. But the same loosening would also have to let through a plan that names a module and an approach while still hiding its real indecision one level down inside that module — the check currently can't distinguish "I know exactly where this goes and I'm naming the precise spot during implementation" from "I'm naming a general area because I haven't actually decided." I left the stricter wording in place and accepted the one false reject rather than risk loosening a check whose job is specifically to catch plans that look executable but defer their one real decision; I made this choice before the confirming run rather than as a later loosening of an already-passing check, so no canary re-run was needed.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
