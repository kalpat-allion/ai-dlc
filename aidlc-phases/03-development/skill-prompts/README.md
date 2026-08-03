# Phase 3 Development Skill Prompts

This directory holds the **`SKILL.md` templates** for the Phase 3 development skills referenced in [`../PROCESS.md`](../PROCESS.md). They are templates, not active skills — Claude Code only auto-discovers folders under `.claude/skills/`.

Each skill encodes **one ordered procedure ending at a human gate**. Unlike the [Phase 3 specialist subagents](../subagent-prompts/), which a developer invokes by name, **a skill auto-triggers from its description alone.** That makes the `description:` field the entire routing surface — read [Routing](#routing) before editing one, because `open-pull-request` shipped into a **collision with two subagents in the directory next door**, closed by a coordinated edit that any future description change can re-open.

The two are deliberately asymmetric about writes. **`open-pull-request` writes nothing to the tracker** — it hands the finished PR title and body to `linear-task-agent` and stops at that agent's confirm-then-`go`. **`run-sprint-planning` performs its tracker writes itself**, because `linear-task-agent` refuses sprint planning in three places and cannot be routed into. Both terminate at a human, not at the next artifact.

## Files

| Folder | Procedure | Gate | Wraps |
|--------|-----------|------|-------|
| [`open-pull-request/`](./open-pull-request/SKILL.md) | Reconcile the branch identifier against the named story, run the suite and the pre-PR self-review, fix **and re-review** every Critical/High, walk the Definition of Done in three tokens, rebase and re-review any resolved hunk, write the title and body, hand them verbatim to the tracker agent | [Gate 2 (PR Merge)](../QUALITY-GATES.md#gate-2-pr-merge), pre-merge half — the per-story DoD block, the rebase line, and the "PR links story, describes approach, notes AI-generated sections" line | [`self-review-before-pr`](../PROMPTS.md#self-review-before-pr), [`pr-description`](../PROMPTS.md#pr-description) — plus the rebase half of [PROCESS 4.1](../PROCESS.md#step-4-code-review), which has no prompt |
| [`run-sprint-planning/`](./run-sprint-planning/SKILL.md) | Pull the candidates read-only, drop anything outside Backlog, refuse to size a story with < 3 AC, size against the repo, post estimates behind a literal `go`, decompose XXL into sub-issues, return a three-token commitment-readiness report | [Gate 1 (Sprint Commitment)](../QUALITY-GATES.md#gate-1-sprint-commitment) — three of its five criteria; the other two are reported as unanswerable | [`linear-sprint-pull`](../PROMPTS.md#linear-sprint-pull), [`estimation`](../PROMPTS.md#estimation), [`estimates-to-linear`](../PROMPTS.md#estimates-to-linear), [`story-decomposition`](../PROMPTS.md#story-decomposition) |

The **Wraps** column is this directory's purpose. Because shipped templates must stand alone in a consuming repo, each skill **inlines** its procedure with `{{PLACEHOLDERS}}` rather than linking back to `PROMPTS.md` — so the table above is the only record of where a procedure came from. Keep it current, or the round-trip is lost.

> **Note — three defects in the wrapped prompts, recorded rather than fixed.** No prompt text was changed; the paste path stays valid. The skills work around all three.
>
> 1. **`estimation` and `story-decomposition` disagree on the AC floor, and the gap is a deadlock.** `estimation` refuses below **3** AC bullets; `story-decomposition` refuses below **5**. A story with 3-4 bullets can therefore be sized XXL and then cannot be decomposed — while [Gate 1](../QUALITY-GATES.md#gate-1-sprint-commitment) requires *"story decomposed into sub-issues if AI-estimated XXL"*. The gate line becomes unsatisfiable by any legal path. The skill neither invents seams nor waives the line: it records the line as `not met` and sends the story back for AC.
> 2. **`estimates-to-linear` stores only half the estimate Gate 1 asks for.** The gate wants *"T-shirt + Fibonacci"*; only the Fibonacci value lands in a field. The T-shirt size is prose inside a comment, so the gate line cannot be checked mechanically. The skill pins the T-shirt size to the comment's **first line** so it is at least greppable — which is a mitigation, not a fix.
> 3. **`pr-description` and `linear-task-agent`'s pr-open flow specify different PR bodies.** The prompt's shape is Summary / Closes / Approach / AC checklist / AI-generated sections / Notes for reviewer; the shipped agent drafts Summary / AC coverage / Test plan / Out of scope / Reviewer notes. The agent's shape carries **neither** the closing keyword nor the AI-generated notation — two separate Gate 2 lines. Hand-off integrity is why `open-pull-request` hands the **finished body verbatim, to post unmodified**, rather than handing a payload to draft from.

> **Note — `open-pull-request` requires the Phase 3 subagent set.** It is the only shipped skill that names sibling slugs in its body (`code-reviewer`, `frontend-engineer`, `backend-engineer`, `conflict-resolver`, `linear-task-agent`) rather than describing them by work type. That is a deliberate trade: the same-phase slugs never dangle if the directories are installed together, and the hand-off has to be exact. **Install [`../subagent-prompts/`](../subagent-prompts/) first.** `run-sprint-planning` names no slug and stands alone.

## How to instantiate per repo

1. Copy the chosen **folder** into `.claude/skills/<skill-name>/` at the consuming repo root, preserving the folder name. **The folder name must equal the `name:` field** — Claude Code uses the folder name as the invocation slug, and a mismatch means the skill silently never loads. Use `cp -r`; globbing the Markdown files flattens the structure and produces zero working skills.
2. Replace the placeholders with the repo's values:
   - `{{DEFAULT_BRANCH}}` — the branch PRs target and the branch you rebase onto, e.g. `main`, `master`, `develop` (open-pull-request)
   - `{{TEST_RUNNER}}` — how the suite is run, e.g. `vitest`, `pytest`; **the same value the implementation specialists carry** (open-pull-request)
   - `{{PR_TITLE_FORMAT}}` — e.g. `[ENG-XXX] <imperative summary>`; **the same value `linear-task-agent` carries**, or the title it composes and the title this skill composes will differ (open-pull-request)
   - `{{LINEAR_PROJECT}}` — the project the backlog sits on, e.g. `TimeSync v2 — Scheduling` (run-sprint-planning)
   - `{{TEAM_PREFIX}}` — the team key, e.g. `ENG` (run-sprint-planning)
   - `{{TEAM_STACK}}` — language and framework, e.g. `Next.js 14 + TypeScript`; estimates sized against the wrong stack are confidently wrong (run-sprint-planning)
3. **The Linear MCP server must be connected at project scope before `run-sprint-planning` runs** — see [`../PROCESS.md` → Step 0](../PROCESS.md#step-0-one-time-setup--connect-claude-code-to-linear-via-mcp). It writes directly; the connecting user's Linear permissions are the ceiling. `open-pull-request` needs no MCP server of its own, only the git host CLI its tracker agent uses.
4. Commit `.claude/skills/<skill-name>/SKILL.md` to the repo — skills at project scope are shared team infrastructure; treat edits as code changes requiring review.
5. **Verify. `/agents` does not list skills** — skills are a different surface and are verified differently. Run all five checks:
   - **Discovery.** Open a Claude Code session; the skill appears in the available-skills list under its slug. If not, the folder name and the `name:` field are out of alignment.
   - **Explicit invocation.** Type `/<skill-name>` with a representative input. The procedure should run top-to-bottom and stop at its human gate — including `run-sprint-planning`'s confirm-before-write stop, which must wait for a literal `go`, and `open-pull-request`'s hand-off, which must stop at the tracker agent's own `go` rather than opening anything itself.
   - **Auto-trigger, in messy language.** This is the check that matters, because the description is the whole routing surface:
     - `open-pull-request` — *"i think that's it for eng-247, can you get it up for review"*
     - `run-sprint-planning` — *"we're planning wednesday, can you size what's in the backlog and stick the numbers on the tickets"*
   - **Refusal.** Drive each skill into its sharpest refusal and confirm it redirects rather than complies:
     - `open-pull-request` — *"there's two Highs left but the reviewer will catch them, just raise it"* and *"mark the DoD done, I checked it manually last week"*
     - `run-sprint-planning` — *"it's only got two AC bullets, just give me a rough number"* and *"re-estimate ENG-247, it's turning out bigger than we thought"*
   - **Hand-off integrity** — the check most often skipped, and the one that matters most here. Confirm the receiving subagent accepts exactly what the skill hands it. For `open-pull-request`, `linear-task-agent` must post the supplied title and body **unmodified**: if it re-drafts from them, the closing keyword and the AI-generated-sections notation go missing, and those are two separate [Gate 2](../QUALITY-GATES.md#gate-2-pr-merge) lines. A skill that hands off the wrong shape is worse than no skill — it produces a confident-looking failure.
6. **Negative-routing check, once, after installing more than one.** Ask for ordinary work these skills must not claim — *"review my diff"*, *"implement the checkout endpoint"*, *"move ENG-247 to In Progress"*, *"resolve these merge conflicts"*, *"create the issues for the booking epic"* — and confirm neither loads. **The first of those five is the one to watch** — it is the phrase the closed `code-reviewer` collision ran through, so it is where a re-opening would surface first; see below.

## Routing

Skills auto-trigger; subagents do not. A skill whose trigger phrases reuse a verb a specialist subagent already claims **wins that route by default and hijacks ordinary work.** The cross-phase version of this table lives in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholder names that recur under other spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

| Skill | Owns | Explicitly does not own |
|---|---|---|
| `open-pull-request` | The ordered pre-PR gate: self-review → verified fixes → the DoD walk → rebase → the composed title and body | The diff review itself, the fixes, the conflict resolution, the tracker write, the merge, anything after the PR is open |
| `run-sprint-planning` | The candidate pull, the sizing, the estimate writes, the XXL decomposition, the readiness report | Every write around a story already in flight; backlog construction from a requirements document; velocity calibration; owner assignment; opening the cycle |

**Neither is scoped to a stage of the project.** A PR opened in month nine runs the same gate; a cycle planned in month nine is planned the same way.

### `open-pull-request` shipped into a collision — now closed

Two shipped subagents claimed the same utterance, **verbatim in their descriptions**:

| Colliding phrase | Claimed by | Consequence if the incumbent wins |
|---|---|---|
| "open the PR for this branch" | `linear-task-agent` | The PR opens with **no** self-review, **no** DoD walk and **no** rebase. The entire gate is bypassed and the PR looks normal |
| "anything to fix before I PR" · "review my diff before I PR" | `code-reviewer` | The developer gets the review — step 2 of seven — and reasonably reads a clean report as *ready*. Steps 3-7 never run |

Both are **closed**, by a coordinated edit that shipped in the same change as this skill:

- `linear-task-agent` — its request-classification step now treats the pre-PR gate as a **precondition** of `pr-open` rather than a part of it, mirrored in its description, operating boundaries, escalation list and the pr-open flow itself. The classification step is the load-bearing touch: that agent classifies before any boundary is consulted, so a boundaries-only edit is one the classifier has already run past.
- `code-reviewer` — its Do-NOT list gains the full pre-PR gate, naming `open-pull-request`, **which hands the diff review back here as its step 2**. Its own step-5 output format was also corrected: it previously recommended `linear-task-agent` to open the PR, which wrote the bypass into the agent's output rather than its routing, where no description could reach it.

Re-opening either collision costs one careless description edit, in either direction. **What was done in this skill's own description**, and what must survive editing: it never claims a bare *"review my diff"* — a diff review with no PR at the end of it is excluded by name, which leaves `code-reviewer` its own lane in the reciprocal direction. And every hand-off in the body names the subagent that owns it, so a mis-fire costs an extra hop rather than a stolen job.

`run-sprint-planning` collides with nothing. Its lane was reserved for it in advance: [`../../01-requirement-gathering/skill-prompts/README.md`](../../01-requirement-gathering/skill-prompts/README.md) already lists *"estimate these stories for the sprint"* as work the requirement skills must not take, and `linear-task-agent` refuses sprint planning in its description, its operating boundaries and its escalation list. **Do not add a bare tracker verb to it** — "create an issue", "pick up my next task", "move this to In Progress" — and do not restore the trigger *"is this ready to commit"*: unqualified, it reads as a question about a working tree, and it would pull a cycle-planning procedure into an ordinary git turn. Every trigger names the sprint, the cycle or the backlog for exactly that reason.
