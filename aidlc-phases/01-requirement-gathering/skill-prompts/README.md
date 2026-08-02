# Phase 1 Requirement Skill Prompts

This directory holds the **`SKILL.md` templates** for the Phase 1 requirement skills referenced in [`../PROCESS.md`](../PROCESS.md). They are templates, not active skills — Claude Code only auto-discovers folders under `.claude/skills/`.

Each skill encodes **one ordered procedure ending at a human gate**. Unlike the [Phase 3 specialist subagents](../../03-development/subagent-prompts/), which a developer invokes by name, **a skill auto-triggers from its description alone.** That makes the `description:` field the entire routing surface — read [Routing](#routing) before editing one.

All three write to Linear. They are the **outer-loop** writers: Documents, Projects, Milestones and Triage issues, each behind an explicit confirm-then-`go` stop. The dev-loop Linear writer (`linear-task-agent`, Phase 3) deliberately does none of this and refuses if asked.

## Files

| Folder | Procedure | Gate | Wraps |
|--------|-----------|------|-------|
| [`publish-prd-to-linear/`](./publish-prd-to-linear/SKILL.md) | Draft the eleven-section PRD from supplied context, self-review it for gaps, resolve Critical/High, create the Project in `Planned` and publish the PRD as an attached Document, return the read-back section-anchor map | Gate 1 (PRD Completeness & Published in Linear) | [`prd-generation`](../PROMPTS.md#prd-generation), [`prd-to-linear-document`](../PROMPTS.md#prd-to-linear-document), [`gap-analysis`](../PROMPTS.md#gap-analysis) **(pre-publish self-review half only — see note)** |
| [`scaffold-linear-milestones/`](./scaffold-linear-milestones/SKILL.md) | Read the approved Document over MCP, decompose into ordered epics with dependencies and complexity, check every requirement section is claimed exactly once, create one Milestone per epic, stop at the scaffold-approval gate | Gate 2 (Linear scaffolding half) | [`epic-decomposition`](../PROMPTS.md#epic-decomposition), [`prd-to-linear-scaffold`](../PROMPTS.md#prd-to-linear-scaffold-milestones) |
| [`push-linear-stories/`](./push-linear-stories/SKILL.md) | Draft stories per epic with Given/When/Then AC, run the read-only duplicate pre-flight, create Triage issues with labels and a verified PRD deep-link, stop for per-story acceptance in the AI Inbox | Gate 2 (story-content half); feeds Gate 3 | [`user-story-generation`](../PROMPTS.md#user-story-generation), [`acceptance-criteria`](../PROMPTS.md#acceptance-criteria), [`stories-to-linear-push`](../PROMPTS.md#stories-to-linear-push), [`linear-context-pull`](../PROMPTS.md#linear-context-pull) **(absorbed as the duplicate pre-flight)** |

The **Wraps** column is this directory's purpose. Because shipped templates must stand alone in a consuming repo, each skill **inlines** its procedure with `{{PLACEHOLDERS}}` rather than linking back to `PROMPTS.md` — so the table above is the only record of where a procedure came from. Keep it current, or the round-trip is lost.

> **Note — `publish-prd-to-linear` wraps only half of `gap-analysis`.** That prompt has two uses: the **pre-publish self-review** on a local draft (Step 2.3), and the **post-approval sweep** that diffs the published Document against the backlog (Step 4). Only the first is inlined here, as the skill's step 3. The second is a different procedure with a different input, a different output and a different gate, and stays a prompt until `sweep-requirements-gaps` is built.

**`sweep-requirements-gaps` is deliberately not in this directory.** It is Phase 1's fourth conversion candidate; its inputs (`gap-analysis` post-publish half, `linear-gap-sweep`) stay as prompts in the meantime.

Its job is verification, and every one of its findings reduces to parsing the `**PRD section:** [§X.Y](url#anchor)` line out of real issue descriptions and re-resolving it against a Document that has since been re-versioned. **A sweep that mis-parses that line returns clean — and a clean sweep is the phase-handoff gate's evidence.** Built from a spec rather than against real data, it would manufacture a gate pass, so it waits for data.

**Build it when either of these fires:**

1. A build-evidence note is committed here recording one real sweep against a project these three skills built — counts per category, whether the `**PRD section:**` line parsed without hand-editing, whether any deep-link failed to re-resolve after a Document version bump, and run time.
2. Any artifact ships that edits an approved PRD Document after publication — that creates orphan anchors with nothing checking them.

**Do not build it if** two recorded sweeps come back with zero Critical/High findings and no anchor failures. The prompt is sufficient at that point.

> The previous condition was "once the three skills have run on a real PRD", which nobody reading this repo can check. A hold condition that cannot be verified is the same defect as a quality-gate checkbox that cannot be falsified.

## How to instantiate per repo

1. Copy the chosen **folder** into `.claude/skills/<skill-name>/` at the consuming repo root, preserving the folder name. **The folder name must equal the `name:` field** — Claude Code uses the folder name as the invocation slug, and a mismatch means the skill silently never loads. Use `cp -r`; globbing the Markdown files flattens the structure and produces zero working skills.
2. Replace the placeholders with the repo's values:
   - `{{LINEAR_INITIATIVE}}` — the parent Initiative the Project is created under, e.g. `TimeSync v2` (publish-prd-to-linear)
   - `{{LINEAR_TEAM}}` — the Linear team that owns the work, e.g. `Engineering` (publish-prd-to-linear)
3. **The Linear MCP server must be connected at project scope before any of these run**, and the connecting user's Linear permissions are the ceiling — see [`../PROCESS.md` → Step 0](../PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp). A skill that cannot reach the MCP server will say so; it will not fall back to producing Markdown that looks like it was published.
4. Commit `.claude/skills/<skill-name>/SKILL.md` to the repo — skills at project scope are shared team infrastructure; treat edits as code changes requiring review.
5. **Verify. `/agents` does not list skills** — skills are a different surface and are verified differently. Run all four checks:
   - **Discovery.** Open a Claude Code session; the skill appears in the available-skills list under its slug. If not, the folder name and the `name:` field are out of alignment.
   - **Explicit invocation.** Type `/<skill-name>` with a representative input. The procedure should run top-to-bottom and stop at its gate — including the confirm-before-write stop, which must wait for a literal `go`.
   - **Auto-trigger, in messy language.** This is the check that matters, because the description is the whole routing surface:
     - `publish-prd-to-linear` — *"i've got notes from three stakeholder calls, can you turn them into a proper requirements doc and get it into linear"*
     - `scaffold-linear-milestones` — *"the PRD's signed off, can you carve it up into chunks of work"*
     - `push-linear-stories` — *"we need tickets for the booking epic with proper acceptance criteria"*
   - **Refusal.** Drive each skill into its sharpest refusal and confirm it redirects rather than complies:
     - `publish-prd-to-linear` — *"just fill in some sensible non-functional requirements, we'll firm them up later"*
     - `scaffold-linear-milestones` — *"the PRD isn't approved yet but set the milestones up anyway"*
     - `push-linear-stories` — *"skip the duplicate check, this project's empty"* and *"just accept the stories yourself, the inbox review is a formality"*
6. **Negative-routing check, once, after installing more than one.** Ask for ordinary work these skills must not claim — *"implement the checkout endpoint"*, *"review my diff before I PR"*, *"pick up my next Linear task"*, *"estimate these stories for the sprint"* — and confirm none of the three loads. If one does, its description has stolen a verb from a specialist subagent; narrow it and re-test. The last of those four is the one to watch: sprint pull and estimation belong to the outer-loop *development* planning skill, not to these.

## Routing

Skills auto-trigger; subagents do not. A skill whose trigger phrases reuse a verb one of the specialist subagents already claims **wins that route by default and hijacks ordinary work.** Three consequences when editing anything in this directory:

- **Never add a trigger phrase containing a bare development verb** — implement, build, write the tests, review the diff, refactor, open the PR, ship. Those belong to the Phase 3 specialists.
- **Never add a bare tracker verb** — "create an issue", "file a bug", "pick up my next task", "move this to In Progress". Those are `linear-task-agent`'s and `file-followup-bug`'s. These three skills always write Linear in the context of a *PRD*, and every trigger phrase should carry that context.
- **The three skills must exclude each other by name.** They run in sequence on the same Project, and the boundary between them is *which Linear primitive each owns*, not which topic each covers:

| Skill | Owns | Explicitly does not own |
|---|---|---|
| `publish-prd-to-linear` | the Project, the PRD Document, the section-anchor map | Milestones, Issues, the approval itself, the post-approval gap sweep |
| `scaffold-linear-milestones` | Milestones and their order and dates | the Project, the Document, Issues, the scaffold approval |
| `push-linear-stories` | Triage issues, their AC, labels and deep-links | Milestones, the Document, any state move past Triage, clearing `needs-human-review` |

**These three are not scoped to a stage of the project.** Requirements work happens whenever it happens — a mid-project change request or follow-on scope uses the same three skills. The boundary against the story-loop tracker agent is a *write type* (constructing the backlog vs. operating on a story already in it), not a phase. The cross-phase version of this table lives in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholder names that recur under other spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

Two notes on why the collision surface is small, worth preserving as the set grows. **Every trigger phrase names the PRD or the backlog structure**, never a bare Linear noun — which is what keeps them clear of the dev-loop tracker agent that shares the same MCP server. And **each skill terminates at a human gate rather than at the next skill**, so a mis-fire costs a confirmation, not a backlog.
