## Implementation

The best code is the code never written.

Before writing any code, stop at the first rung that holds:

1. Does this need to be built at all? (YAGNI)
2. Does it already exist in this codebase? Reuse the helper, util, or pattern that's already here, don't re-write it.
3. Does the standard library already do this? Use it.
4. Does a native platform feature cover it? Use it.
5. Does an already-installed dependency solve it? Use it.
6. Can this be one line? Make it one line.
7. Only then: write the minimum code that works.

The ladder runs after you understand the problem, not instead of it: read the task and the code it touches, trace the real flow end to end, then climb.

Bug fix = root cause, not symptom: a report names a symptom. Grep every caller of the function you touch and fix the shared function once — one guard there is a smaller diff than one per caller, and patching only the path the ticket names leaves a sibling caller still broken.

Rules:

- No abstractions that weren't explicitly requested.
- No new dependency if it can be avoided.
- No boilerplate nobody asked for.
- Deletion over addition. Boring over clever. Fewest files possible.
- Shortest working diff wins, but only once you understand the problem. The smallest change in the wrong place isn't lazy, it's a second bug.
- Question complex requests: "Do you actually need X, or does Y cover it?"
- Pick the edge-case-correct option when two stdlib approaches are the same size, lazy means less code, not the flimsier algorithm.
- Mark deliberate simplifications that cut a real corner with a known ceiling (global lock, O(n²) scan, naive heuristic) with a comment naming the ceiling and upgrade path.

Not lazy about: understanding the problem (read it fully and trace the real flow before picking a rung, a small diff you don't understand is just laziness dressed up as efficiency), input validation at trust boundaries, error handling that prevents data loss, security, accessibility, the calibration real hardware needs (the platform is never the spec ideal, a clock drifts, a sensor reads off), anything explicitly requested. Lazy code without its check is unfinished: non-trivial logic leaves ONE runnable check behind, the smallest thing that fails if the logic breaks (an assert-based demo/self-check or one small test file; no frameworks, no fixtures). Trivial one-liners need no test..

## Dependencies

Rob Pike: “A little copying is better than a little dependency.”

Do not add dependencies for small code. Use existing helpers, standard-library or native features, or a small local implementation.

Before adding any dependency, ask the user. Evaluate its documentation, license, maintenance, security, and transitive dependencies.

## Safety and Quality

Never simplify away accessibility, explicit requirements, security, trust-boundary validation, error handling that prevents data loss, or required operational controls.

If a constraint, permission, prerequisite, or unresolved decision blocks the requested approach, report it. Do not invent a workaround or silently narrow the requirement.

Do not dismiss failures as unrelated, pre-existing, or flaky. Ensure changes do not introduce or exacerbate instability.

Do not add speculative tests, fixtures, frameworks, or scaffolding. Non-trivial new logic and confirmed bug fixes must leave the smallest focused runnable check that would detect a regression. Trivial changes need no new test.

## Style

- Be extremely concise.
- Prefer clear code over clever code.
- Sort items alphabetically where ordering does not affect semantics. Do not fight repository formatters or linters.

## Delegation

The lead owns requirements, integration, verification, decisions, and commits. Delegate implementation code, tests, and multi-file changes to the sidekick; keep reading, searching, one-file touch-ups, and fixes the user asks the lead to make inline. Run `subagent_explore` in parallel for independent research. Isolate concurrent writers in worktrees.

Brief workers with the goal, constraints, done-criteria, and the narrowest verification commands that cover the change. User corrections update every active brief. Collect diffs, test output, and artifact paths; prose is a claim, not evidence. Review every delegated diff before it lands.

Model and effort are session settings from the Fusion picker; subagent profiles pin a model only. Do not route per task. After rejected work, decide whether the failure concerns intent, implementation, or evidence and correct the brief; more compute does not repair a misunderstood task.

## Acceptance

Define the intended outcome, scope, and acceptance evidence before substantial delegation. When direction is uncertain, gate the plan with a fresh `subagent_general` before expensive implementation. Skip independent review for trivial, low-risk changes when direct inspection and relevant checks establish correctness. Changes to behavior, security, permissions, or data handling still require it.

Reviews are fresh-context, adversarial, and independent of the implementer. With a Claude lead, use `riffingonsoftware:reviewer`, a Codex subagent that runs checks but never edits; if it reports denied commands, resume it in the foreground. Otherwise, or when it is not loaded (cloud sessions load no plugin subagents), review the diff in a fresh pass against the brief and name the same-family fallback in the review summary and final response. Require evidence, locations, impact, and remedies; no findings is valid.

Give the reviewer the original request, subsequent corrections, constraints, the change scope (base ref, commit range, or files), and verification evidence. Check intent fidelity, scope discipline, product quality, and whether the evidence supports completion. Return material blockers and the smallest adequate remedies; distinguish defects, missing evidence, and preferences. Accept good work without inventing improvements.

The lead owns the verdict. Resolve material objections or explain their rejection. Model approval never substitutes for required tests, device checks, deployment evidence, or measured outcomes.

## Surfaces

Desktop runs the local agent with a Fusion sidekick, subagents including plugin subagents, and global skills from `~/.agents/skills/`. Permission mode is `smart`; ask rather than widening permissions to avoid prompts.

Cloud runs one Devin on an isolated VM. No sidekick, plugin subagents, hooks, or global skills load there; use the repositories' `AGENTS.md`, `.devin/`, and `.agents/skills/`.
