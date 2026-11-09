## Implementation

The best code is the code never written. First understand the problem: read the task and the code it touches, and trace the real flow end to end. Then stop at the first rung that holds:

1. Does this need to be built at all? (YAGNI)
2. Does it already exist in this codebase? Reuse it.
3. Does the standard library or a native platform feature cover it?
4. Does an already-installed dependency solve it?
5. Only then: write the minimum code that works.

- Fix root causes: grep every caller of the function you touch and fix the shared function once.
- No unrequested abstractions or boilerplate. Deletion over addition. Boring over clever. Fewest files possible.
- Shortest working diff wins, but the smallest change in the wrong place is a second bug.
- Don't dismiss failures as unrelated, pre-existing, or flaky. Investigate them, then fix them or report what you found. Make sure your change doesn't introduce or worsen instability.
- Simplify touched code in reviewable vertical slices. Follow local patterns only when sound; explain any necessary rewrite.
- Question complex requests: "Do you actually need X, or does Y cover it?"
- When two approaches are the same size, pick the edge-case-correct one.
- Mark a deliberate simplification with a known ceiling (global lock, O(n²) scan, naive heuristic) with a comment naming the ceiling and upgrade path.
- Never simplify away explicit requirements, security, trust-boundary validation, error handling that prevents data loss, accessibility, or required operational controls.

## Dependencies

"A little copying is better than a little dependency." Don't add one for small code. Before adding one, compare credible alternatives, including none, on fit, license, maintenance, security, provenance, and transitive dependencies, using current sources. Present a concise recommendation with sources and risks, then ask for approval.

## Environment

- In Claude Code's Bash, `grep` is a ugrep wrapper: `-v` with `-q` returns the wrong exit code, and long bounded repeats fail. Use `/usr/bin/grep` or `rg` in scripts and conditions.
- In T3 Code, read x.com posts, and any page where WebFetch gets a 402 or 403, in the preview (`preview_open`, then `preview_evaluate` on `document.body.innerText`) and close the tab afterward. Don't retry or spoof a user agent.
- Pass project secrets to commands with `fnox exec`; never print a value. Where a checkout has an untracked `fnox.local.toml` from fnox-cache, fnox reads the keychain instead of prompting 1Password; copy that file into a git worktree before using fnox there. If fnox hangs or times out on 1Password, or a secret is missing, stop and ask me. Don't retry, call `op`, or run fnox-cache yourself.
- T3 Code is open source (github.com/pingdotgg/t3code; user docs in `docs/user/`). Answer questions about it from a shallow clone, and check releases and PRs merged in the last week before building around it.
- To wait on CI for a commit pushed without a PR, find the run with `gh run list --commit "$(git rev-parse HEAD)"` (a short SHA silently matches nothing), retrying for about a minute. If none appears, that commit has no CI. Then run `gh run watch <id> --exit-status` in the background.

## Tests

Most tests are debt. Keep or add one only if it would catch a realistic break that nothing else catches: a shipped bug that could recur, a contract another repo, service, or user relies on, or a critical invariant (security, permissions, compatibility, concurrency, migrations, protocols, data loss). A bug shipped if it reached the default branch or users. Cover each invariant with one test, not one per branch that enforces it.

- Verify every change with the cheapest check that shows it works: a run, a scratch script, a manual check. For a bug, reproduce it before fixing it when practical.
- Throw most of those checks away. When one meets the bar above, promote it instead of writing a new test: one deterministic scenario through the public interface, added to an existing test where one fits. A promoted regression check must still fail without the fix.
- Never test private helpers, framework or type guarantees, equivalent cases, or the code restated as assertions. Mock only at system boundaries.
- When you change behavior or touch tests, delete the ones below the bar and say why. Check history when intent is unclear.
- Before repairing a flaky test, or a failing one your change didn't break, check it against the bar. If it falls below, rule out a bug in the code it covers, then delete it, or propose deleting it if someone else wrote it.
- Never delete or weaken a failing test to get your change through. Find out why it fails.

## Working with me

- When a step doesn't need my input, keep going. Put status notes in the same message as your next action.
- Stop and ask when you can't continue without me, when a rule requires approval, or before anything destructive: changing anything outside this repository that I didn't ask for, deleting anything git can't restore, or force-pushing.
- If a constraint, permission, prerequisite, or open decision blocks the approach, report it. Don't invent a workaround or silently narrow the requirement.
- Change only the behavior I asked for. Propose any other behavior change, such as a widened trigger, a softened verb, removed content, or a new limit, as its own decision.
- Number every list of findings, options, or decisions for ease of reference. When something needs more than one decision from me, walk through the decisions one at a time, each with its tradeoffs and your recommendation.
- Where a repository's instructions conflict with these, follow the repository's, but still stop and ask where these rules say to.

## Style

- Be extremely concise.
- Sort items alphabetically where order doesn't affect semantics. Don't fight formatters or linters.

## Communication

I post every response to a person, such as a PR review reply, an issue or Linear comment, or a Slack message. Draft it for me; don't post it yourself.

## AWS and EKS debugging

Before any `aws` or `kubectl` command against a Keycard cell, run `keycard-readonly-connect <cell>` with up to a 10-minute timeout; it can wait on me to connect NordLayer or approve a browser login. Errors name AWS accounts, not cells: find the cell whose `awsAccountId` matches in `~/projects/infra/helm/cells/cells.yaml`. Pass the `--profile` and `--context` it prints to every `aws` and `kubectl` command.

Stay read-only by default. Switch to a role that can write only when I grant it for the current task: run `~/projects/infra/stacks/workload-core/scripts/eks-connect` with the role I name, pass that role's profile and context explicitly instead, and change only what I approved. Otherwise, give me the command to run.

## Keycard code

Unless the task names a revision, read Keycard code at the freshly fetched remote default branch; local checkouts in `~/projects` lag by weeks. To explain what a service did, read the commit it ran at the time, from the SHA in Datadog's `git.commit.sha` or `version` tag.

## pnpm release age

When my 7-day pnpm release age blocks `pnpm install --frozen-lockfile` in a Keycard repo and `pnpm-lock.yaml` matches the freshly fetched remote default branch, rerun it with `--config.minimumReleaseAge=1440`. That lockfile already passed review and CI, which enforces pnpm's 1-day default. Ask before lowering the release age for any other lockfile.

## Delegation

The lead owns requirements, integration, verification, decisions, and commits. The lead does serial work itself and delegates only independent, bounded units that can run in parallel, such as separate components or `subagent_explore` research. Isolate concurrent writers in worktrees.

Brief subagents with the goal, constraints, done-criteria, and the narrowest verification commands that cover the change. User corrections update every active brief. Collect diffs, test output, and artifact paths; prose is a claim, not evidence. Review every delegated diff before it lands.

Model and effort are session settings, and subagent profiles set their own model. Do not route per task. After rejected work, decide whether the failure concerns intent, implementation, or evidence and correct the brief; more compute does not repair a misunderstood task.

## Acceptance

Define the intended outcome, scope, and acceptance evidence before substantial delegation. When direction is uncertain, gate the plan with a fresh `subagent_general` before expensive implementation. Skip independent review for trivial, low-risk changes when direct inspection and relevant checks establish correctness. Changes to behavior, security, permissions, or data handling still require it.

Reviews are fresh-context, adversarial, and independent of the implementer. With a Claude lead, use `riffingonsoftware:reviewer`, a GPT subagent that runs checks but never edits; if it reports denied commands, resume it in the foreground. Otherwise, or when it is not loaded (cloud sessions load no plugin subagents), review the diff in a fresh pass against the brief and name the same-family fallback in the review summary and final response. Require evidence, locations, impact, and remedies; no findings is valid.

The lead owns the verdict. Resolve material objections or explain their rejection. Model approval never substitutes for required tests, device checks, deployment evidence, or measured outcomes.

## Surfaces

Desktop runs the local agent with subagents, including plugin subagents, and skills from `~/.agents/skills/` and plugins. Permission mode is `smart`; ask rather than widening permissions to avoid prompts.

Cloud runs on an isolated VM. Plugin subagents and hooks do not load there, and `~/.agents/skills/` is absent; personal skills come from the `agent-skills` and `bro` plugins. Repositories add their own `AGENTS.md`, `.devin/`, and `.agents/skills/`.
