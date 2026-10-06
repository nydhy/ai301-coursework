# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

Where it lives: in an eval bundle, the plan's "Diagnosis" section (or equivalent opening paragraph), read against every line of the "Repro evidence" block — especially its control runs, which are the facts that can rule a cause out. In live mode, the student's own posted Unit 2 repro comment (or the house repro pack), read the same way.

What good looks like: the stated cause is consistent with every artifact the repro evidence shows, not just the main failing run. A diagnosis that repeats a theory from the issue or thread without checking it against the repro's own controls is exactly what this family exists to catch — a control run that already shows the behavior without the blamed mechanism present (or shows it absent when the blamed mechanism is present) rules that diagnosis out, no matter how confidently it's stated.

## Scope

Where it lives: the plan's "Scope" (in-scope / not-in-scope) statement, read together with its "Files" and "Approach" sections as one combined list of everything the plan proposes to touch.

What one bounded change looks like next to a drive-by rewrite: a bounded plan maps its proposed changes directly onto the issue's trigger and explicitly defers anything else, naming a reason ("not in scope: the deeper crate-level fix, which the maintainer already called hard"). A drive-by rewrite reads as several independent projects under one plan — a redesign, a migration, a new abstraction, a version bump unrelated to the bug, a CI matrix — regardless of whether one item among them is the correct fix.

## Executability

Where it lives: the plan's "Files" and "Approach" sections, read for concreteness this time rather than for scope.

What it means for a stranger to start without asking: every file or code location is named, not described generically ("somewhere in the input layer"), and the approach is an ordered sequence of concrete steps, not an open question ("investigate whether X or Y is the better layer"). A plan that defers its real decision to build time ("fix it in whichever of upstream or vendored turns out easier") is not executable yet, however reasonable the surrounding prose sounds.

## Test plan

Where it lives: the plan's "Test plan" section, read against the repro evidence's own steps and artifacts (same source as Diagnosis, different question).

What a decisive test plan names that a vague one does not: a specific, observable result tied to the repro — re-running the exact repro command/control and naming the expected concrete output (an exit code, a value, a named artifact) that a second reader could check and agree on. "Run the full test suite" or "should feel fast" names an activity, not an outcome, and is not decisive even when everything else about the plan is strong.

## Honesty

Where it lives: the plan's stated risks/unknowns (or their absence), and the confidence language in both the plan and the plan comment.

How to tell a stated unknown from false confidence: an honest plan says what it does not yet know ("I have not measured the per-print cost of this check; if it shows up in benchmarks I will move it") without that uncertainty undermining the diagnosis itself, which should still be grounded per the Diagnosis family above. False confidence asserts a result ("guaranteed", "this is definitely the root cause") that the stated evidence does not support, or omits risk entirely while the rest of the plan reads as more certain than its own grounding allows.

## Comms

Where it lives: the plan comment's own sentences, read against two separate things — the thread highlights (or the live thread) for any maintainer comment that already states a diagnosis, a fix location, or a rejected approach, and the repo-facts block's stated AI-use/disclosure policy.

What thread-aware looks like next to boilerplate: a plan comment that engages a maintainer's already-stated direction explicitly, either adopting it or saying why it diverges, rather than silently pursuing an unrelated angle (a maintainer naming the exact buggy function and posting a patch to test is a direction; a docs-only plan that never mentions it is routing around it). Where the repo's policy requires disclosing AI assistance — unconditionally or conditional on AI having been used — the comment needs one explicit sentence naming the tool and that a human reviewed the work; an excellent plan that skips this still fails here.
