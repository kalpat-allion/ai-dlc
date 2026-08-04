# Phase 4 Testing & QA Skill Prompts

This directory holds the **`SKILL.md` templates** for the Phase 4 testing skills referenced in [`../PROCESS.md`](../PROCESS.md). They are templates, not active skills — Claude Code only auto-discovers folders under `.claude/skills/`.

Each skill encodes **one ordered procedure ending at a quality gate**. Unlike the [Phase 4 specialist subagents](../subagent-prompts/), which a developer invokes by name, **a skill auto-triggers from its description alone.** That makes the `description:` field the entire routing surface — read [Routing](#routing) before editing one.

Neither skill writes test code, and only one of them writes to the tracker at all.

## Files

| Folder | Procedure | Gate | Wraps |
|--------|-----------|------|-------|
| [`author-test-plan/`](./author-test-plan/SKILL.md) | Fetch the requirements document and architecture over MCP rather than pasting, draft the nine-section plan with an AC-to-test map, gap-check it against the source ACs and NFRs, resolve every Critical/High and re-check, leave ownership blank, publish as a tracker Document with read-back section anchors | Gate 1 (Test Plan Approved) | [`Test Plan Generation`](../PROMPTS.md#test-plan-generation), [`test-plan-gap-analysis`](../PROMPTS.md#test-plan-gap-analysis), [`test-plan-to-linear-document`](../PROMPTS.md#test-plan-to-linear-document) |
| [`reproduce-and-diagnose-bug/`](./reproduce-and-diagnose-bug/SKILL.md) | Check the stack-trace-or-reproduction entry condition, choose the cheapest reliable layer, write the failing test named after the bug, verify it fails for the right reason, rank root causes against that evidence, state the blast radius, hand the fix over so test and fix ship in one diff, seed the post-mortem | Gate 3 (regression, and the S0/S1 bug-status lines) | [`bug-reproduction`](../PROMPTS.md#bug-reproduction), [`Debugging (Bug Investigation)`](../PROMPTS.md#debugging-bug-investigation), [`post-mortem`](../PROMPTS.md#post-mortem) **(terminal-step checklist only — see note)** |

The **Wraps** column is this directory's purpose. Because shipped templates must stand alone in a consuming repo, each skill **inlines** its procedure with `{{PLACEHOLDERS}}` rather than linking back to `PROMPTS.md` — so the table above is the only record of where a procedure came from. Keep it current, or the round-trip is lost.

> **Note — `reproduce-and-diagnose-bug` folds in `post-mortem` as a seed, not as a deliverable.** That prompt stays a prompt: six input blocks and two refusal rules is not one prompt with arguments. But its two refusals — no post-mortem on an unmerged fix, none without the regression test in the merged diff — are exactly what this skill's ordering guarantees, so its checklist becomes the skill's terminal step. **The skill hands the seed back unposted.** A published post-mortem is a human artifact written after the merge; anything this procedure produces is written before it.

> **The wrapped prompts are not alternatives to the procedure.** `author-test-plan` reads as three prompts and runs as one: entering at *"gap-check the plan"* or *"publish the test plan"* **resumes** at that step, which means running the steps in front of it on whatever draft exists. Publication is hard-gated on the gap check clearing with zero Critical and zero High. Advertising the third prompt as a standalone entry point is how a plan gets published past a gate that never ran.

## How to instantiate per repo

1. Copy the chosen **folder** into `.claude/skills/<skill-name>/` at the consuming repo root, preserving the folder name. **The folder name must equal the `name:` field** — Claude Code uses the folder name as the invocation slug, and a mismatch means the skill silently never loads. Use `cp -r`; globbing the Markdown files flattens the structure and produces zero working skills.
2. Replace the placeholders with the repo's values:
   - `{{LINEAR_PROJECT}}` — the tracker Project the requirements Document is attached to and the test plan is published under, e.g. `TimeSync v2 — Scheduling` (author-test-plan)
   - `{{TEST_RUNNER}}` — the unit and integration runner, e.g. `vitest`; **the runner, never the CI command with coverage flags** (author-test-plan, reproduce-and-diagnose-bug)
   - `{{TEAM_PREFIX}}` — the tracker team key the bug identifier carries, e.g. `ENG` (reproduce-and-diagnose-bug)
   - `{{BUG_LABEL}}` — the label a triaged bug carries, e.g. `bug` (reproduce-and-diagnose-bug)
   - `{{DEFAULT_BRANCH}}` — the branch the failing test must fail on, e.g. `main` (reproduce-and-diagnose-bug)
   - `{{TECH_DEBT_LABEL}}` — the label the post-mortem's prevention action is filed under, e.g. `tech-debt` (reproduce-and-diagnose-bug)
3. **`Playwright` and `k6` are written as literals in both skills, deliberately, and are not fills.** Both were drafted as `{{E2E_FRAMEWORK}}` and `{{LOAD_TEST_TOOL}}` and the names were withdrawn before they shipped: the two agents in [`../subagent-prompts/`](../subagent-prompts/) encode rules that are true of those tools and no others — trace-file diagnosis, role-based locators, `thresholds` wired to NFR numbers — so the tool is not a repo fact a value can vary. Left as placeholders, an operator filling `Cypress` gets a published plan naming Cypress and an installed agent writing Playwright, **and the fill looks correct on both sides**. A team on a different tool edits these templates; it does not fill a value.
4. **The tracker's MCP server must be connected at project scope before `author-test-plan` runs**, and the connecting user's permissions are the ceiling. Its step 1 fetches the requirements Document and the architecture over MCP by design — a skill that cannot reach the server will say so; it will not fall back to producing Markdown that looks published. `reproduce-and-diagnose-bug` needs no MCP server: it makes no tracker write.
5. Commit `.claude/skills/<skill-name>/SKILL.md` to the repo — skills at project scope are shared team infrastructure; treat edits as code changes requiring review.
6. **Verify. `/agents` does not list skills** — skills are a different surface and are verified differently. Run all four checks:
   - **Discovery.** Open a Claude Code session; the skill appears in the available-skills list under its slug. If not, the folder name and the `name:` field are out of alignment.
   - **Explicit invocation.** Type `/<skill-name>` with a representative input. The procedure should run top-to-bottom and stop at its gate — for `author-test-plan` that includes the confirm-before-write stop, which must wait for a literal `go`.
   - **Auto-trigger, in messy language.** This is the check that matters, because the description is the whole routing surface:
     - `author-test-plan` — *"the PRD's signed off, how are we actually going to test all this"*
     - `reproduce-and-diagnose-bug` — *"ENG-512 is assigned to me and I can't get it to happen on my machine"*
   - **Refusal.** Drive each skill into its sharpest refusal and confirm it redirects rather than complies:
     - `author-test-plan` — *"just publish it, the Tech Lead will catch anything missing"* and *"put in some sensible latency targets, we'll firm them up later"*
     - `reproduce-and-diagnose-bug` — *"skip the test, fix it and I'll add the test in a follow-up PR"* and *"no stack trace but it's obviously the auth middleware, just patch it"*
     - One check that must **not** fire: *"there's no report count on this bug"* is not a refusal. Segmentation, report counts and event frequency are optional context — a skill that stops there refuses on nearly every human-reported bug, and that is the exact failure the entry condition was relaxed to avoid.
7. **Negative-routing check, once, after installing both.** Ask for ordinary development work these skills must not claim — *"implement the checkout endpoint"*, *"write the tests for this module"*, *"review my diff before I PR"*, *"file a bug for that auth race"*, *"this E2E spec is flaky in CI"* — and confirm neither loads. If one does, its description has stolen a verb from a specialist; narrow it and re-test. The second of those five is the one to watch: *"write the tests"* belongs to the implementation specialists, and it is one word away from `author-test-plan`'s own trigger.

## Routing

Skills auto-trigger; subagents do not. A skill whose trigger phrases reuse a verb one of the specialist subagents already claims **wins that route by default and hijacks ordinary work.** Three consequences when editing anything in this directory:

- **Never add a trigger phrase containing a bare development verb** — implement, build, write the tests, review the diff, refactor, open the PR, ship. Those belong to the implementation and review specialists.
- **`author-test-plan`'s triggers must always carry "test plan" or "test strategy".** *"Write the test plan"* is one word from *"write the tests"*, which is reserved. The qualifier is the only thing holding that boundary, exactly as the tracker-writing skills hold theirs by always naming the requirements document.
- **Never add a bare tracker or triage verb** to `reproduce-and-diagnose-bug` — "pick up this bug", "move it to In Progress", "how bad is this one". The first two are the story-loop tracker agent's writes; the third is the QA Lead's severity call, which is a human gate by design.

| Skill | Owns | Explicitly does not own |
|---|---|---|
| `author-test-plan` | The test plan Document, its AC-to-test map, the gap check that gates its publication, its section anchors | Test code in any layer, coverage auditing of an existing suite, ownership assignment, the approval, any bug |
| `reproduce-and-diagnose-bug` | One failing regression test named after one triaged bug, the verification that it fails for the right reason, the ranked root cause, the blast radius, the post-mortem seed | The fix, severity and triage, any tracker write, module and story tests, end-to-end suite health, the published post-mortem |

**Neither skill is scoped to a stage of the project.** A test plan is written whenever a release needs one, and bugs are worked whenever they are filed — both run continuously alongside development, so the boundary against the implementation specialists is a *write type* (a plan and a single named regression test, versus the tests that ship with the code), not a phase. The cross-phase version of this table lives in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholder names that recur under other spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

Two notes worth preserving as the set grows. **`reproduce-and-diagnose-bug` earns its route by enforcing an ordering, not by fetching context** — the test before the fix, verified to fail for the right reason, shipped in one diff. If a team already does that unprompted, the skill is dead weight and should be uninstalled rather than kept for tidiness. And **its entry condition is deliberately looser than the prompt it wraps**: the wrapped prompt refused without affected-user segmentation, a field with no source in this stack. Re-tightening it re-creates a skill that refuses on almost every real bug and looks pedantic rather than broken.
