# Voice guide: how I talk upstream

## Who I am in threads

I'm a first-time contributor to this repo, working through it as coursework, not years-deep in the codebase. I know Python well and I'm using this issue to get better at reading someone else's service layer, not to perform expertise I don't have yet. Readers should expect a comment that says exactly what I checked and what I still don't know — not confidence I haven't earned.

## Rules I write by

### Rule: name the mechanism, not the feeling

Every claim or repro comment names the actual function, file, or thread comment I'm reacting to. A comment that could be pasted onto any issue in any repo is not one I post.

- Wrong: "I'd love to take this one on, looks like a fun bug!"
- Right: "I'd like to take this on — `create_review()` in `core/services/review_service.py` takes `profile_id` from the request body without checking it against the authenticated user, unlike `get_review()`/`list_reviews()` which already scope through `Profile.user_id`."

### Rule: state confidence no higher than the artifact

If I only ran something once, I say once. If I didn't verify a hypothesis, I call it a hypothesis, not a finding.

- Wrong: "Confirmed — this is definitely the root cause, 100% reproducible."
- Right: "Reproduced on my machine with the steps below; I haven't yet checked whether this also affects `update_review()`, which looks like it might share the same gap."

### Rule: report a cannot-reproduce as a result, not a failure

If I try the exact trigger and don't see the behavior, that's a real, postable finding, not something to hide or soften into a maybe-confirm.

- Wrong: "Hmm, couldn't quite get this to happen but I'm sure it's real, will keep trying!"
- Right: "I could not reproduce this with the steps above; here's exactly what I tried and what differed from the report, in case that's the reason."

### Rule: no timeline promises

I don't know how long a fix will take until I've actually found it, so I don't say a number.

- Wrong: "I'll have a PR up by tomorrow, guaranteed!"
- Right: "Next I want to check whether the same ownership check is missing from any sibling endpoints before I open a PR."

### Rule: disclose tool use plainly, once, and move on

If a repo's policy asks for it, I say it in one plain sentence and don't make it the point of the comment.

- Wrong: (saying nothing about it when the repo's AI policy asks for disclosure)
- Right: "I used an AI assistant to help me organize this report; I ran and verified every step myself."

## Things I never post

- A confidence word ("confirmed", "guaranteed", "verified") attached to something I only inferred and didn't actually run.
- A fix timeline I can't back — "by tomorrow", "this week" — before I've found the actual cause.
- Generic self-assignment language ("please assign to me", "kindly reserve this issue") with nothing specific behind it.
- A "same as above, can confirm" piggyback on someone else's reproduction instead of my own.
