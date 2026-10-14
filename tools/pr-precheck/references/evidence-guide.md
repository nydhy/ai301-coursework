# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

Where it lives: in an eval bundle, the "Plan context" block's scope/not-in-scope statement, Files list, and any deviation note, read against the candidate PR's "Diff" (every changed file, every hunk) and "Description" sections. In live mode, your `plan.md` (including its `## Deviations` heading) read against the literal output of `git diff main...HEAD` and your draft description.

What good looks like: every file the diff touches, and every behavior change inside it, maps onto something the plan's Files/Scope list named, or onto a change the plan's own deviation note explicitly discloses. A description's claim about what the PR does is read against the diff itself, not taken at its word: "implements the plan exactly" or "no other behavior changes" over a diff that also rewrites an unrelated function, adds an unscoped config option, or touches a file the plan never named is silent drift — in the more-than-planned direction. A description that claims an addition the diff doesn't actually contain (e.g. "docs now document the limitation" with no doc-file hunk anywhere in the diff) is the same family of drift in the less-than-claimed direction. A disclosed, named deferral (stated in both the plan's deviation note and the description, and consistent with what the diff actually omits) is not drift — it's honest scoping, graded under `honest-shortfall` instead.

## Test evidence (harness category: not-tested)

Where it lives: the candidate PR's "Test evidence" section, read against the plan's named test plan (the specific repro/scenario it said to re-run) and the issue's own reproduction steps/artifacts, plus whatever the repo's PR template or CONTRIBUTING docs name as required checks (lint, typecheck, unit tests, integration tests).

What decisive looks like next to "tests pass": the evidence names the exact scenario the plan said to re-run (not an adjacent one — a GET request when the fix was for POST, a single-file case when the plan named a two-file scenario, the control run alone with the actual trigger never re-run) and states a concrete before/after result for it (an exit code, an output value, a passing/failing assertion), plus the repo's own required checks reported as actually run with their outcome. "Verified working on my machine" or "tests pass" with nothing named is not decisive, however confident it reads. When a plan's test plan names more than one failure mode or scenario, decisive evidence covers all of them, or discloses which it didn't (graded under `honest-shortfall`).

## Diff quality (harness category: unreviewable)

Where it lives: the full unified diff, every hunk, and the commit list — read for cleanliness this time, independent of whether each hunk is in scope.

What a reviewable change looks like next to debris: every hunk is something a reviewer would need to read to understand the fix. The debris tells to look for: commented-out code (including a commented-out first attempt kept "for reference"), debug output (`eprintln!`, `console.log`, a `DBG` print) left in, a dead or unused function/variable that exists only to probe the bug during development, pure reformatting or re-indentation hunks that change no behavior, and commit messages that are only "wip" / "fix" / "fmt + cleanup" with no real description anywhere in the series. A correct, well-evidenced fix still fails this family when debris rides along with it — the fix being right does not make the extra hunks reviewable.

## Standards and comms (harness category: standards-wall)

Where it lives: the repo-facts block's stated PR template sections and its contribution/AI-use policy, read against the candidate PR's description.

What compliant looks like next to boilerplate: every section the repo's PR template asks for is filled with real, specific content — a genuine `Closes #123` link, a checklist with boxes actually ticked for things that are actually true, a changelog entry in the repo's own format if the template asks for one — not the template's own placeholder text left untouched or a section skipped. Where the repo's policy requires disclosing AI assistance (unconditionally, or conditional on AI having been used), the description needs one explicit sentence naming the tool and that a human reviewed the work; an otherwise-excellent PR that skips this still fails here, the same way it did for the claim and plan comments in earlier weeks. (Whether the description's own claims match what the diff actually contains is `plan-fidelity`, not this family — this family is only about the repo's stated asks, not internal consistency.)
