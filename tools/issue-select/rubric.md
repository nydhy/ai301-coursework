# Rubric: is this a good first issue?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever checks
you define here. It ships empty on purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where. Name the source
     (repo-facts block, issue body, comment thread, or the locations in
     references/evidence-guide.md). "The repo" is not a source; "the last
     5 default-branch commit dates" is.
   - Pass condition: a condition someone else could apply and get your
     answer. Prefer thresholds with numbers ("a maintainer commented
     within 30 days") over adjectives ("maintainer is responsive").
   - Weight: `required` (a fail here rejects the issue) or `preferred`
     (never changes the verdict; a nice-to-have that helps rank the
     issues you accept).

2. A verdict rule below the table: how the check grades combine into
   accept or reject, including how `unclear` is treated. The verdict
   space is binary. If you write no rule for `unclear`, the skill treats
   it as fail.

Cover what actually kills first contributions. The lecture named four
families: the maintainer is alive, the repo is in use, the scope fits a
newcomer, and nobody else is already on it. A rubric that ignores a family
will fail eval issues designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Maintainer alive | Repo facts: last 5 default-branch commits (dates), maintainer first-response sample | At least one of the last 5 default-branch commits is dated within 90 days of the capture date, OR at least one entry in the maintainer first-response sample shows a first owner/member/collaborator reply within 30 days of that issue's open date | required |
| Repo in use | Repo facts: archived flag, latest release date, last push date, star count | `archived: no`, AND (last push to any branch is within 180 days of the capture date OR the latest release is within 365 days of the capture date) | required |
| Scope fits a newcomer | Issue title, body, comment thread, and linked-PR states | The issue describes one bounded piece of work that could land as a single PR. Naming several instances of the same fix (e.g. "add previews for rule A, B, and C" or "update pages X, Y, Z to reflect one change") still passes — that is one change with several touch points, not multiple items. Fail when any of: (a) the issue is explicitly structured as a checklist/tracking list of separate items meant to be claimed and shipped as separate PRs by separate people; (b) it is a pure usage/support question; (c) a maintainer comment in the thread says the design is still undecided or that the fix touches core internals; (d) the issue has 2 or more closed/unmerged linked PRs recorded against it, showing prior contributors already tried and abandoned it — a sign of real unresolved difficulty behind a friendly label; (e) the issue asks for a new user-facing feature or capability (not a bug fix, not a docs/content task) and no Owner/Member/Collaborator anywhere in the thread has endorsed building it, and it was not opened by a maintainer — an un-endorsed feature request is a product decision nobody has made yet | required |
| Nobody already on it | "this issue: assignees" and "linked PRs" line, plus the comment thread | No assignee is set, no linked PR is open, and no comment thread claim ("I'll take this" / "working on this") is followed by ongoing recent activity from that claimant within the last 30 days of the capture date. (Per `scope.md`, this check is skipped entirely for Path Review candidates in live mode: classmates' claims never fail this check there.) | required |
| Contribution/AI policy allows this workflow | "contribution policy" line in Repo facts, or CONTRIBUTING.md / AI policy files in live mode | Passes unless the policy contains an outright ban on AI-generated code or documentation. Disclosure requirements, "must understand and test your changes," or human-review requirements are conditions, not bans, and pass. Silence on AI use passes. | required |
| Good-first-issue label present | Issue labels | Issue carries a "good first issue" (or equivalent, e.g. "help wanted" + newcomer-friendly language) label | preferred |
| Maintainer filed or endorsed the issue | Issue author's association (OWNER/MEMBER/COLLABORATOR) in the issue header | The issue was opened by, or explicitly endorsed in the thread by, someone with an Owner/Member/Collaborator badge | preferred |

## Verdict rule

Accept only if every `required` check grades `pass`. Any `required` check
that grades `fail` or `unclear` rejects the issue — treat `unclear` as
`fail` for every required check (evidence that cannot be found is not
evidence of safety). `preferred` checks never affect the verdict; report
their grades and use them only to rank issues that are already accepted,
preferring issues that pass more of them.
