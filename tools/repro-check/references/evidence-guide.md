# Evidence guide: where proof lives in a reproduction package

## Environment

Where it lives: in an eval bundle, the first paragraph of "Candidate repro report" (the line usually starting "Environment:"), plus the issue's own header/body for its stated target version. In live mode, the same line in the student's repro draft, read against the issue thread's stated version (often in the issue body or its bug-report template fields) and, if relevant, the repo's release page.

What good looks like: a specific tool version or commit/build (not "latest"), the OS/platform, and install method where it plausibly matters. If the version used differs from the issue's target, the report says so in the same breath ("filed against 1.9.4/1.10.0; still present on 1.11.7") rather than leaving the reader to notice the mismatch themselves.

## Steps

Where it lives: the numbered "Steps" section of the repro report (eval bundles), or the equivalent section of a live draft. Compare against the issue's own "Steps to reproduce" for what the trigger actually requires.

What good looks like: every step is something a stranger could paste or click with nothing else assumed — inline file contents (`printf` / heredoc / shown config) rather than "use my test file", and no step that silently requires a private resource (an internal monorepo, an unshared playground link, credentials). A step like "set up the project" or "run the usual build" with no commands shown fails this regardless of how confident the surrounding prose is.

## Behavior shown

Where it lives: the code block(s) or described output under "Steps"/"Reproduction" in the repro report — the actual command output, log lines, error text, or exit code. Read it against the issue's own quoted output or described symptom, not against the issue's title or the student's paraphrase of it.

What good looks like: the same trigger (same syntax, same kind of input) produces the same class of failure the issue names — same error type or exit code, not a different error that happens to also be non-zero; a claimed crash needs a crash shown, not a clean validation message. An honest cannot-reproduce still counts here if the attempt used the issue's real trigger and the artifact shown is what that attempt actually produced, with the reporter naming what plausibly differed (environment, data shape, timing) rather than asserting the bug is real from vibes.

## Honesty

Where it lives: the "Expected" vs "Actual" (or equivalent) lines at the end of the repro report, read against the artifact just above them, and any confidence language in the claim or repro comment ("guaranteed", "definitely", "verified the root cause", "ran it N times").

What good looks like: the conclusion says only what the artifact shown supports. A report that shows a graceful error but concludes "confirmed the crash" is not honest even if every other section looks thorough. A report that says "I could not reproduce, here is what differed" is a full pass here — cannot-reproduce is not a lesser outcome, it is the correct one when that is what happened. A root-cause diagnosis stated as fact with no artifact demonstrating it fails here regardless of how plausible the diagnosis sounds.

## Comms

Where it lives: the "Candidate claim comment" text itself (eval bundles) or the student's claim draft (live mode), read against the repo-facts contribution policy / AI-use policy field, and against the issue's own mechanism (a named function, file, or thread comment).

What good looks like: the claim comment names something specific from the issue (a function, a linked comment's hypothesis, a concrete next step) rather than generic enthusiasm or a request to be assigned with nothing behind it. Where the repo's stated policy requires disclosing AI assistance — unconditionally, or conditionally on AI having actually been used — at least one comment contains an explicit sentence naming the tool and that a human reviewed/verified the work; "no stated AI policy" or an explicitly permissive one needs no such sentence. Boilerplate self-assignment language ("please assign to me", "I will fix this in N days guaranteed", excessive flattery with no substance) fails here even when the attached repro report is otherwise strong.
