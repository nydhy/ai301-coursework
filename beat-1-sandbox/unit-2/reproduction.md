# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

nydhy

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/67#issuecomment-5882525024

Hi, I'd like to take this on as a first contribution. I've read through the relevant code: `POST /reviews` (`api/routes/reviews.py`) passes both `data.profile_id` and `current_user.id` into `create_review()`, but `create_review()` in `core/services/review_service.py` only uses `profile_id` to build the `Review` row — the `user_id` argument it receives is never checked against the profile's owner. That's inconsistent with `get_review()` and `list_reviews()` in the same file, which both join through `Profile.user_id == user_id` before returning anything.

Next I want to set up the sandbox environment, exercise `POST /reviews` with a `profile_id` that belongs to a different user, and record what happens, before writing up a reproduction here.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/67#issuecomment-5882872048

Environment: my fork of `codepath/pathreview-ai301-fa26-s3`, commit `2f4e82f` (`main`), Python 3.12.13, SQLAlchemy 2.1.1, macOS 15.7.9 (arm64). I didn't have Docker available, so this reproduces the bug directly at the service layer — the same mocked-async-session pattern `tests/unit/test_review_service.py` already uses — rather than through a live `POST /reviews` HTTP call. Calling that out explicitly: this shows the missing check at its source, not the full request path.

Steps: from a fresh clone, `pip install -e ".[dev]"` (no Docker/Postgres needed for this), then run the script committed here: https://github.com/nydhy/pathreview-ai301-fa26-s3/blob/61eb42e4975aee7d20e3dd09b69d974ffe4dfca5/repro_issue_67.py

It calls `create_review(db, profile_id=<a profile belonging to user_b>, user_id=<user_a>)`, then as a control calls `get_review()` on the same session.

Output:

```
=== create_review(profile_id=profile_of_b, user_id=user_a) ===
Review constructed with profile_id=f33003bb-102e-4ab0-a835-385113e05333
db.execute call count during create_review: 0
-> Review created for user_b's profile while acting as user_a,
   with ZERO queries against the profiles table to check ownership.

=== get_review(review_id, user_id=user_a) [control] ===
db.execute call count during get_review: 1
Compiled SQL WHERE clause references Profile.user_id:
  True
SELECT reviews.id, reviews.profile_id, ... FROM reviews JOIN profiles ON profiles.id = reviews.profile_id
WHERE reviews.id = :id_1 AND profiles.user_id = :user_id_1
```

Expected: acting as `user_a` with a `profile_id` that belongs to `user_b`, `create_review()` should either reject the request or route it through the same `Profile.user_id` ownership check `get_review()`/`list_reviews()` already use.

Actual: `create_review()` constructs and commits a `Review` for `user_b`'s profile while the `user_id` argument it receives is never used — zero queries run against `profiles`, so no ownership check ever happens. The control shows `get_review()`, in the same file, issuing a real `profiles.user_id`-scoped query — confirming the check exists elsewhere and is simply missing here. Matches the issue exactly.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. `--limit 4` smoke run (pkg-01 through pkg-04): 4/4
2. Full confirming run, `--save-run eval-run.txt`: **18/20 scored items (bar: 18/20: PASS)**, category floor clear on all five categories (`clear-accept 6/8`, `disclosure 1/1`, `no-evidence 4/4`, `unfollowable-comms 3/3`, `wrong-target 4/4`) — this is the run committed in `eval-run.txt`. I ran the full 20-package grade twice back to back before saving (once unsaved, once with `--save-run`) and got the identical 18/20 result and the identical two disagreements both times, so no revision cycle was needed between them.

**Package analysis**

`pkg-05`. Gold label: `accept` (a conda `EnvironmentSectionNotValid` message printed to stdout instead of stderr, minimal `env.yml` repro with a JSON-parse-failure artifact). My rubric's verdict: `reject`, failing `steps-followable` and `control-run-present`. The report's steps say "wrote a minimal `env.yml` containing a valid `dependencies:` list plus a `category:` section (the section conda does not recognize)" but never pastes the literal YAML content, only describes it. My rubric's `steps-followable` pass condition requires "every step is a literal, copyable command/snippet/action with enough starting state given to run it from scratch (files created inline, config shown, no unstated setup)" — a description of a file's shape is not the same as showing the file, so my grader read this as an unstated setup step rather than a literal one, even though the described contents are unambiguous to a human reader.

**Check rationale**

From `rubric.md`, the `steps-followable` row's evidence column, quoted exactly as it currently reads: "The repro report's steps, read as literal commands, code, or UI actions a stranger with no access to the reporter's machine could carry out." I wrote this check to catch packages like `pkg-18` (golangci-lint), whose reproduction "lives entirely in a private monorepo with an unshared config" — a stranger literally cannot re-run it. Requiring the evidence itself (not a paraphrase of it) is what makes that check catch a private, unshareable resource; the same wording is what then reads `pkg-05`'s described-but-not-shown `env.yml` as falling short of "literal," even though nothing about `pkg-05`'s setup is actually private or unshareable.

**Trade-offs**

Loosening `steps-followable` to accept a step that accurately *describes* a config's shape (not just one that pastes it) would flip `pkg-05` and `pkg-12` to agree with gold, but it would also loosen the same check on `pkg-18`, whose repro report likewise describes what it did ("reproduction lives entirely in a private monorepo") without showing the config — the two cases differ only in whether the described resource happens to be shareable, and my rubric's wording can't currently tell "described because it's simple" from "described because you can't share it." I left the stricter wording in place and accepted the two false rejects rather than risk a false accept on the one-package case the check exists to catch; I did not have to re-run a canary for this, since I made this choice before the confirming run rather than as a later loosening of an already-passing check.
