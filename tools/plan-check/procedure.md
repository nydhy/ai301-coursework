# Procedure: how this skill grades a plan package

## Read order

1. Read the issue context first (title, body, labels) and note the bounded symptom the plan must answer to in one line.
2. Read the repro-evidence block in full. Note every control run and its outcome — what did and did not reproduce, and under what condition — since these are the facts that can rule a diagnosis out. In live mode, this is the student's own posted Unit 2 repro comment, or the house repro pack for a house issue.
3. Read the repo-facts block. Note the stated AI-use/disclosure policy verbatim, and any contribution convention relevant to a plan comment.
4. Read the thread highlights (or the live thread). Note any comment from a maintainer (OWNER/MEMBER/COLLABORATOR author association) that states a diagnosis, names a fix location, rejects an approach, or asks for something specific. If none exists, note that explicitly rather than leaving it blank.
5. Read the candidate plan in full: Diagnosis, Scope, Files, Approach, Test plan, Risks/unknowns, in that order.
6. Read the candidate plan comment last.
7. Before grading anything, write down six one-line notes: the plan's stated cause; its in-scope and not-in-scope lines; its named files and approach steps; its test plan's expected observable outcome; whether the plan comment engages any maintainer direction found in step 4 (and how); and whether it discloses AI use if the repo's policy (step 3) requires it.

## Evidence gathering

For each check, pull evidence only from the source named here — never substitute a different section because it seems close enough:

- `diagnosis-grounded`: the plan's Diagnosis section, against every control run and artifact noted in step 2.
- `scope-bounded`: the plan's Scope, Files, and Approach sections read together as one list of proposed changes, against the issue's stated trigger from step 1.
- `plan-executable`: the same Files/Approach sections, read this time for concreteness (named locations plus an ordered sequence) rather than for scope.
- `test-plan-decisive`: the plan's Test plan section, against the repro evidence's own steps and artifacts from step 2.
- `thread-and-conventions`: the plan comment's own sentences, against the maintainer-direction note from step 4 and the AI-policy line from step 3.
- `honest-unknowns` (preferred): the plan's Risk/unknowns language and the plan comment's confidence language, against what steps 2 and 5 actually established.

Live mode: issue context, thread highlights, and the repo-facts/AI-policy block come from `gh`, the GitHub API, or the web, per `references/evidence-guide.md`. Repro evidence comes from the student's own posted Unit 2 repro comment (or the house repro pack). The plan and plan comment are the student's own draft files, read exactly as written — never substitute other files in the student's working directory.

## Check execution

1. Execute checks in this fixed order: `diagnosis-grounded`, `scope-bounded`, `plan-executable`, `test-plan-decisive`, `thread-and-conventions`, then `honest-unknowns`.
2. Grade each check only against the evidence named for it above. A later check never reverses an earlier check's grade, even if it surfaces evidence that feels relevant to both.
3. If the evidence a check needs is genuinely absent from the package (no Risk section at all, no repro-evidence block, no thread highlights to check against), grade that check `unclear` and name exactly what was missing. Do not infer missing evidence from an adjacent section.
4. The six one-line notes from Read order step 7 are enough to grade every check `pass` or `fail` provisionally; before finalizing any `fail` or `unclear` grade, re-read the exact source section so the output quotes the real line, not a paraphrase of the note.

## Verdict assembly

1. Accept only if every required check (`diagnosis-grounded`, `scope-bounded`, `plan-executable`, `test-plan-decisive`, `thread-and-conventions`) grades `pass`.
2. Any required check graded `fail` or `unclear` sets the verdict to `reject`. `unclear` is always treated as `fail` here, with no exception.
3. `honest-unknowns` is preferred: report its grade in the checks list, but it never changes the verdict computed in step 1-2.
4. For the output JSON, every check gets its own one-line evidence quote (the real line or fact that decided it, gathered in Check execution step 4) — not only the deciding one.
