# Procedure: how this tool grades a PR package

## Read order

1. Read the issue context first (title, body, labels) and note the bounded symptom in one line.
2. Read the plan in full, including its `## Deviations` note if present. List, in writing: its in-scope statement, its explicit not-in-scope line(s), every file it names, its approach steps, its named test plan (the exact repro/scenario it says to re-run), and any deviation it discloses. This list is what every side-by-side check below compares against — read it before the diff, never after, so the diff is judged against what was promised rather than the promise being read backward from the diff.
3. Read the repo-facts block. Note the PR template's required sections verbatim, and the stated AI-use/disclosure policy verbatim.
4. Read the candidate PR's title and description in full.
5. Read the full diff: every changed file, every hunk, in order. For each file, note whether it appears on the plan's Files list from step 2. For each hunk, note whether it implements a step from the plan's approach, is covered by a disclosed deviation, or is neither.
6. Read the commit list.
7. Read the Test evidence section last. Note, for each piece of evidence, which specific repro/scenario/check it exercises.
8. Before grading anything, write down six one-line notes: which (if any) diff files/hunks fall outside the plan-plus-deviations list; whether the description's claims about scope match what step 5 found; whether the test evidence matches the plan's named test plan and the repo's required checks; whether any hunk or commit is debris (debug output, dead code, pure churn, unrelated edits); whether every required template section is filled with real content; whether disclosure is required and, if so, present.

## Evidence gathering

For each check, pull evidence only from the source named here:

- `plan-fidelity`: the diff's file list and hunks from step 5, against the plan's Files/Scope/not-in-scope list and any deviation note from step 2; the description's own scope claims from step 4.
- `test-evidence-decisive`: the Test evidence section from step 7, against the plan's named test plan from step 2 and the repo's required checks from step 3.
- `diff-reviewable`: the same diff read in step 5, this time scanning every hunk and commit message for debris or unrelated content, not for scope.
- `standards-and-disclosure`: the description from step 4, against the repo-facts block's required template sections and AI policy from step 3.
- `honest-shortfall` (preferred): any deviation note from step 2 and any shortfall language in the description from step 4, against what the diff (step 5) and test evidence (step 7) actually show was built.

Live mode: the plan is the student's own `plan.md` (deviations included); the diff is the literal output of `git diff main...HEAD` run from the branch's working copy; the draft title/description come from the student's own draft file; the issue context, PR template, and AI policy come from the live repo via `gh`, the API, or the web. A house-chain student has no plan of their own: read the house plan and house repro pack in its place, and grade the same checks against those.

## Check execution

1. Execute checks in this fixed order: `plan-fidelity`, `test-evidence-decisive`, `diff-reviewable`, `standards-and-disclosure`, then `honest-shortfall`.
2. Grade each check only against the evidence named for it above. A later check never reverses an earlier one, even where the same hunk is relevant to both (a hunk can be in-scope per `plan-fidelity` and still be debris per `diff-reviewable`; grade both, independently).
3. If the evidence a check needs is genuinely absent (no Test evidence section at all, no repo-facts policy line, no plan to compare against), grade that check `unclear` and name exactly what was missing.
4. The six one-line notes from Read order step 8 are enough to grade every check provisionally; before finalizing any `fail` or `unclear` grade, re-read the exact diff hunk, description sentence, or plan line so the output quotes the real text, not a paraphrase.

## Verdict assembly

1. Accept only if every required check (`plan-fidelity`, `test-evidence-decisive`, `diff-reviewable`, `standards-and-disclosure`) grades `pass`.
2. Any required check graded `fail` or `unclear` sets the verdict to `reject`. `unclear` is always treated as `fail`.
3. `honest-shortfall` is preferred: report its grade, never let it change the verdict from steps 1-2.
4. When more than one required check fails, the output's summary names all of them, but the single check whose evidence line is quoted as "deciding" the verdict is the first one that failed in the Check execution order above (`plan-fidelity` before `test-evidence-decisive` before `diff-reviewable` before `standards-and-disclosure`) — this makes the same package produce the same "deciding check" every time, regardless of which failure an executor happened to notice first.
