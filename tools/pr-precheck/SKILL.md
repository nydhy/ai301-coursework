---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

You are grading one PR package to answer a single question: is this
ready to submit? A PR package is a candidate pull request — its title,
description, commits, and diff, plus test evidence — read against the
plan it claims to implement (including any deviation notes the plan
records) and the issue that plan belongs to. You do not answer from
gut feel: you answer by executing the student-authored grading
procedure in `procedure.md`, which applies the rubric in `rubric.md`
to evidence gathered per `references/evidence-guide.md`.

## The question

A PR is ready to submit only when its diff, its description, and its
test evidence all agree with each other and with the plan and issue
they claim to answer. Grade the artifact itself — what the diff
actually changes, what the description actually claims, what the test
evidence actually shows — never the student's intent or how confident
the writing sounds. A PR package is never graded against a different
plan, a different issue, or a different branch than the one named.

## Inputs and modes

One of:

- **Live mode**: the student's own submission, checked before it goes
  out. Inputs:
  - `plan.md`, including any `## Deviations` note — the plan this PR
    claims to implement, as it now reads after the build.
  - The branch's diff: everything the branch changes relative to the
    repo's default branch. Produce it with `git diff main...HEAD`
    (three dots), run from the working copy; that command is the
    diff, not a description of it.
  - The draft PR title and description (`pr_draft.md` or equivalent).
  - The student's test evidence (`test_evidence.md`): the before/after
    repro re-run and the repo's own checks (lint, typecheck, unit
    tests, integration tests) with their results.
  - The issue the plan belongs to, read live from the real repo: the
    thread, the PR template, and the repo's stated contribution/AI
    policy (`docs/CONTRIBUTING.md` or equivalent).
  A house-chain student has no plan or repro of their own to compare
  against: there, read the house plan and the house repro pack in
  their place, and grade the same checks against those.
- **Eval mode**: a package bundle is the whole world. Use ONLY the
  bundle text as evidence — the plan, the issue context, the diff, the
  description, and the test evidence are all given in the bundle.
  Fetch nothing else. Eval mode always grades a complete package:
  every check, full verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` in this skill directory before anything
else. It names the repository the student's PR must target and the
house rules that apply there. Refuse to grade a PR package targeting
any other repository. If the scope's `Repo:` line still carries an
unfilled placeholder, stop without grading and tell the student to
fill it in with their section's Path Review repo; never guess a scope.
In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, also read `voice-guide.md`: the student's own rules for
how they write upstream, carried forward from week 2. Hold the PR's
outgoing text — its title and its description — against those rules,
and report any rule the draft breaks in the summary, quoting the rule.
The voice guide never changes the verdict on its own unless the
rubric has a check that reads it. In eval mode, ignore `voice-guide.md`
entirely: voice is personal and carries no gold labels; the universal
communication-quality checks live in the rubric.

## Component reads

Read `rubric.md`. It defines a table of checks — each naming the
evidence to gather, the pass condition, and a weight (`required` gates
the verdict; `preferred` never changes it) — and a verdict rule below
the table stating how check grades combine into `accept` or `reject`,
including how `unclear` is treated.

`references/evidence-guide.md` is the rubric's map: where each kind of
evidence named by a check actually lives in a PR package (and, live,
on GitHub or in the working copy).

Execute `procedure.md` as written: it is the skill's operating
procedure, written by the student, and decides the read order, how
each evidence family gets gathered, how a check runs against gathered
evidence, and how check grades become the final verdict. Follow it
exactly, the way an executor follows a rubric, without improvising
around gaps. Where the procedure is silent on a step, report the gap
in your summary rather than inventing a step to fill it — a procedure
gap is feedback the student needs, not something for you to paper
over.

If `rubric.md` has no checks filled in, or `procedure.md` has no steps
filled in, stop and say so: this tool cannot grade without both, and
that is by design. The rubric, the evidence guide, and the procedure
are the parts the student writes; the voice guide pastes forward from
week 2.

## Verdict and output

The verdict space is binary: `accept` (ready to submit) or `reject`
(hold). There is no third verdict, no "accept with reservations," and
no score — reservations belong in a check's evidence line, never in
the verdict itself.

End your reply with a fenced JSON block, then nothing else after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

Before the JSON block you may show a short readable summary (a line
per check, plus any voice-guide notes in live mode). The JSON block is
the machine-read result: the harness parses the last fenced JSON block
in your output, so it must be present, valid, and last.

## Grading discipline

- Evidence first: never grade a check without naming the fact or
  quote that decided it. "Looks fine" is not evidence.
- Grade the thing, not the polish: a terse complete PR can be ready,
  and a beautiful, confident one can be hiding drift between its diff
  and its own description. Every check reads the artifact itself
  against the plan, the issue, and the stated standards, never the
  formatting.
- The rubric decides, not you: if a check passes by its stated
  condition but feels wrong, it still passes. Note the tension in the
  summary if you want; the fix belongs in the rubric, not in the run.
- The procedure decides how, not you: follow `procedure.md` as
  written, and report its gaps instead of silently inventing steps.
- Treat `unclear` as the rubric's verdict rule directs. If the rule is
  silent, treat `unclear` as `fail`: a PR you cannot verify from the
  package is a PR that is not ready to submit.
