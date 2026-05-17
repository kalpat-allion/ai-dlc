# Phase 3: Development — Process Definition

## Overview

This document defines the AI-assisted workflow for the Development phase, using the consolidated AI-DLC tool stack. Linear is the spine: every coding task is opened, started, progressed, and closed against a Linear issue, and Claude Code drives those transitions through the official Linear MCP server.

**Phase Duration:** Sprint-based (typically 2–8 weeks for MVP)
**Phase Owner:** Tech Lead / Engineering Manager
**Tools Used:** **Cursor** (IDE), **Claude Code** (terminal agent), **CodeRabbit** (AI PR review), **SonarQube** (static analysis), **Linear** (project management spine, integrated via MCP), **GitHub / GitLab / Bitbucket** (source control)

> **Tool Philosophy:** Cursor is the IDE — built-in AI autocomplete (Supermaven engine) plus Composer for multi-file edits. Claude Code is the terminal agent for complex refactors, long-running autonomous tasks, and **all** Linear MCP work. CodeRabbit reviews every PR. SonarQube enforces static-analysis gates in CI. **Linear is where every story starts and every PR is closed**: Claude Code reads the next issue from Linear, moves it to *In Progress*, checks out the Linear-suggested branch, opens the PR with `Closes ENG-123` so Linear's git integration auto-transitions on merge, and posts progress as comments on the issue. Five tools, zero overlap.

> **Single-vendor alternative:** The recommended baseline is Cursor Pro plus a Claude Pro/Team plan, as assumed throughout this document. Teams that prefer simpler procurement can drop Cursor entirely and run **Claude Code Max 5x (~$100/dev/mo, approximate, early 2026 — verify before committing budget)** as the sole AI coding tool: one vendor, one billing line, no Cursor licence to manage, and a substantially higher Claude usage quota for heavy users. The trade-off is loss of inline autocomplete-as-you-type — Claude Code is request-response, not ghost-text on the cursor — which can be recovered with **Supermaven standalone** (free tier or ~$10/mo Pro) or any IDE-native completion, though many developers code productively without inline AI completion. On this path, alt-path readers should apply three substitutions wherever the per-step prose names Cursor: **Cursor autocomplete / Supermaven** (Steps 3.2 and 6.1, the last row of the *Decision rule — which agent for which work* table in Step 0, and the Daily Developer Workflow box) becomes **Supermaven standalone or your IDE's native completion**; **Cursor Composer for multi-file scaffolding or localised refactors** (Steps 3.1 and 5.3, the "When to use which tool" decision table, and the Tool attribute rows in Steps 5 and 6) becomes **Claude Code with the relevant specialist subagent** (`frontend-engineer`, `backend-engineer`, or `refactor-specialist` as appropriate); and **Cursor Chat for quick codebase questions** (the corresponding row of the decision table) becomes **Claude Code in the IDE extension or terminal**. The decision rules, gates, and quality bars are unchanged — alt-path readers swap the tools, not the process.

---

## Process Steps

### Step 0: One-Time Setup — Connect Claude Code to Linear via MCP

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Linear workspace OAuth (already enabled in Phase 1 — see [`01-requirement-gathering/PROCESS.md` Step 0](../01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp)), Claude Code installed locally, Git provider OAuth on Linear |
| **Tool** | **Linear MCP server** (`https://mcp.linear.app/mcp`) — official, hosted by Linear, OAuth 2.1 |
| **Output** | Claude Code can read & write Linear issues, sub-issues, comments, projects, and milestones; Linear's git integration auto-transitions issues on PR open/merge |
| **Human** | Each developer authorises Claude Code once via OAuth; Tech Lead expands tool scopes for dev (state changes + sub-issue creation) |

The Linear MCP server set up in Phase 1 is the same server. Phase 3 only adds **two changes** on top: (1) every developer connects Claude Code (not just chat) to it, and (2) the Anthropic Connectors policy is widened to allow `update issue state`, `assign issue`, and `create sub-issue` — operations the PM kept disabled in Phase 1 to prevent scope creep on requirements.

**In this step:**

1. Setup paths — [Path A (Claude Code, primary)](#path-a--claude-code-developer--cli-use-primary) and [Path B (Cursor, optional)](#path-b--cursor-in-ide-linear-browsing-optional), plus [Phase 3 scope expansion](#phase-3-scope-expansion-tech-lead-teamenterprise-only)
2. [Bundle the recurring workflow into `linear-task-agent`](#bundle-the-recurring-workflow-into-a-local-claude-code-subagent-recommended)
3. [Specialist subagents for Phase 3 roles](#specialist-subagents-for-phase-3-roles-recommended) — `software-architect`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`, `conflict-resolver`
4. [Build-your-own subagent recipe](#creating-your-own-claude-code-subagent--step-by-step)
5. [Build-your-own skill recipe](#creating-your-own-claude-code-skill--step-by-step)
6. [Verification checklist](#verification-checklist)
7. [Linear Workspace Setup (Phase-3 additions)](#linear-workspace-setup-phase-3-additions)

#### Path A — Claude Code (developer / CLI use, primary)

This is the primary surface for Phase 3. Every developer runs through it once.

1. **Add the server** at user scope (so it works in every repo):
   ```bash
   claude mcp add --transport http --scope user linear https://mcp.linear.app/mcp
   ```
   Use `--scope project` instead if the team wants the server committed to the repo's `.mcp.json`. CI runners that cannot do interactive OAuth should not push to Linear directly — gate Linear writes behind interactive Claude Code sessions.
2. **Authenticate:** open Claude Code (`claude`) → `/mcp` → select **linear** → approve OAuth in the browser.
3. **Verify available tools.** In a session, prompt: `Via Linear MCP, list_teams and then search_issues assigned to me with state Backlog ordered by priority.` Claude Code should return your teams and your top-priority backlog. If it fails, run `claude mcp list` — `linear: connected` must appear.
4. **Enable the git integration on Linear's side.** In Linear → **Settings → Integrations → GitHub / GitLab / Bitbucket**, connect the provider and enable: (a) **Open PRs auto-link by branch name** (matches `feature/eng-123-…` to `ENG-123`), (b) **Auto-update issue status to "In Review" on PR open**, (c) **Auto-update issue status to "Done" on PR merge**. With this on, Claude Code only needs to set *In Progress* — the PR lifecycle drives the rest.
5. **Revoke when done** (e.g., off-boarding): `claude mcp remove linear` and revoke the OAuth grant in Linear under **Settings → Security & access → OAuth applications**.

#### Path B — Cursor (in-IDE Linear browsing, optional)

Cursor 0.45+ supports MCP servers. Adding the same Linear server inside Cursor lets developers `@`-mention Linear issues from inline chat. Add it in **Cursor Settings → MCP → Add Server** with the same URL and OAuth as Path A. This is convenience-only — do not duplicate writes from both Cursor and Claude Code; pick one writer per task.

#### Phase 3 scope expansion (Tech Lead, Team/Enterprise only)

In the Anthropic admin console → **Connectors → Linear → Tool Permissions**, **enable** the following (kept disabled in Phase 1):

- `update_issue` (state changes — required for "Move to In Progress")
- `assign_issue` (so Claude can self-assign or reassign during pickup)
- `create_issue` with `parentId` (sub-issue creation for story decomposition)
- `create_comment` (already on from Phase 1 — unchanged)

Keep `delete_issue` **disabled** workspace-wide. If a story is wrong, a human cancels it in Linear; Claude does not delete.

#### Bundle the recurring workflow into a local Claude Code subagent (recommended)

Path A wires the MCP connection; this sub-section wraps the **daily** Linear operations — fetch next story, transition state, check out `branchName`, post kickoff and progress comments, open the PR with the right title — into a single-purpose Claude Code subagent. Developers stop retyping the multi-step prompt sequences and the team stops drifting on Linear-write conventions (state transitions, PR title format, comment signature).

The subagent is a markdown file with YAML frontmatter under `.claude/agents/`. Claude Code auto-discovers any file in that directory. Committing it to the repo means every developer (and every fresh clone) gets the same workflow with no per-machine configuration.

1. **Create the directory** at the repo root if it does not exist:
   ```bash
   mkdir -p .claude/agents
   ```
2. **Author the subagent file** at `.claude/agents/linear-task-agent.md`. The reference template at [`./subagent-prompts/linear-task-agent.md`](./subagent-prompts/linear-task-agent.md) is the starting point — copy it into your repo and fill the placeholders (`{{TEAM_PREFIX}}`, `{{LINEAR_PROJECT}}`, `{{PR_TITLE_FORMAT}}`, `{{LABEL_CONVENTIONS}}`, `{{TECH_DEBT_LABEL}}`, `{{BUG_LABEL}}`) plus any team-specific guardrails (e.g., "always read from the Cycle Active saved view, never the global backlog"). At minimum the frontmatter must declare:
   ```yaml
   ---
   name: "linear-task-agent"
   description: "Use for any Linear-driven story workflow: pulling the next assigned issue, transitioning to In Progress, creating the local branch from Linear's branchName, posting kickoff/progress comments, opening a PR with [ENG-XXX] / Closes ENG-XXX format."
   model: sonnet
   ---
   ```
3. **Verify Claude Code discovers it.** In a session, run `/agents` — `linear-task-agent` should appear with its description. Then smoke-test: `Use the linear-task-agent to fetch my next Todo issue from the active cycle and print its branchName — do not transition state yet.` It should return a real issue with a `branchName` field and stop without writing.
4. **Commit the file.** The subagent is shared infrastructure; team-wide consistency on Linear writes depends on every developer using the same definition. Treat edits to `.claude/agents/linear-task-agent.md` as a code change requiring review.
5. **Daily use.** From Step 2 onwards, developers invoke the subagent instead of typing the [`linear-next-task`](./PROMPTS.md#linear-next-task), [`linear-progress-comment`](./PROMPTS.md#linear-progress-comment), or [`pr-description`](./PROMPTS.md#pr-description) prompts directly. PROMPTS.md remains the source of truth — the subagent's system prompt embeds those prompts and must be kept in sync when prompt templates change.

> **Why a subagent and not a slash command?** A slash command fires one prompt; the Linear workflow is a multi-step conversation (fetch → confirm AC → transition → branch → comment) that needs context retention across tool calls. A subagent gives each story its own scoped session with a focused system prompt — which is also why it is safe to grant the `update_issue` and `assign_issue` scopes: responsibilities are bounded.

> **Scope inheritance:** the subagent runs as the developer and inherits their Linear OAuth — it cannot write anything the developer cannot. Off-boarding is unchanged: revoke the OAuth grant in Linear and the subagent loses access automatically.

#### Specialist subagents for Phase 3 roles (recommended)

The `linear-task-agent` above is a **workflow orchestrator** — it drives the Linear writes, branch creation, and PR opening that wrap every story. It does not write production code. The six subagents in this sub-section are **role implementers**: each one is scoped to a specific Phase 3 contributor role (per-story architecture design, frontend, backend, pre-PR review, refactoring, and on-demand merge/rebase conflict resolution) and embeds the relevant PROMPTS.md templates so a developer can drop into a focused, role-aware Claude Code session in one invocation. The two layers are complementary — the typical day is `linear-task-agent` to fetch and start a story, then (for non-trivial stories) `software-architect` to design the approach before any code is touched, then `frontend-engineer` / `backend-engineer` for the implementation, then `linear-task-agent` again to open the PR, with `conflict-resolver` invoked on demand whenever a rebase or merge produces conflicts (typically when rebasing onto main before opening or merging the PR).

Each specialist follows the same packaging pattern as `linear-task-agent`: a markdown file under `.claude/agents/` with YAML frontmatter (`name`, `description`, `model`) and a system-prompt body. Claude Code auto-discovers anything in that directory, so committing the file makes the role definition shared infrastructure — every developer (and every fresh clone) gets the same scoped prompt. Treat edits to these files as code changes requiring review, just like `linear-task-agent.md`.

**At-a-glance comparison:**

| Agent | Suggested model | Read/write surface | When to use | System-prompt template |
|-------|-----------------|--------------------|-------------|------------------------|
| [`software-architect`](#software-architect) | `opus` | Read-only (any file, dry-run linter/type-check, WebSearch/WebFetch) | Step 3.0 per-story design for non-trivial stories | [`./subagent-prompts/software-architect.md`](./subagent-prompts/software-architect.md) |
| [`frontend-engineer`](#frontend-engineer) | Team default (`opus` for complex state machines) | Read any file; write inside `{{FRONTEND_ROOT}}`; run tests/lint/format/type-check | Steps 3.1–3.3 UI implementation | [`./subagent-prompts/frontend-engineer.md`](./subagent-prompts/frontend-engineer.md) |
| [`backend-engineer`](#backend-engineer) | Team default (`opus` for migrations / concurrency) | Read any file; write inside `{{BACKEND_ROOT}}` + `{{MIGRATIONS_PATH}}`; run tests/lint/format/type-check | Steps 3.1–3.3 server-side implementation | [`./subagent-prompts/backend-engineer.md`](./subagent-prompts/backend-engineer.md) |
| [`code-reviewer`](#code-reviewer) | `opus` | Read-only (any file, dry-run linter/type-check, git history) | Step 3.5 pre-PR self-review | [`./subagent-prompts/code-reviewer.md`](./subagent-prompts/code-reviewer.md) |
| [`refactor-specialist`](#refactor-specialist) | `opus` | Read/write within the refactor scope; full test suite after every edit | Step 5 refactoring against a `tech-debt` Linear issue | [`./subagent-prompts/refactor-specialist.md`](./subagent-prompts/refactor-specialist.md) |
| [`conflict-resolver`](#conflict-resolver) | `opus` | Modifies the working tree only to complete a git operation (resolve conflicts, `git add`, `--continue`); never `push`; never `--abort` without instruction | On demand during rebase / merge / cherry-pick conflicts | [`./subagent-prompts/conflict-resolver.md`](./subagent-prompts/conflict-resolver.md) |

**Decision rule — which agent for which work:**

| Situation | Use |
|-----------|-----|
| Fetch next story, transition state, branch, kickoff/checkpoint/PR-opened comments, open PR | `linear-task-agent` |
| Non-trivial story needs a design before scaffolding — schema change, new endpoint surface, new integration, cross-module touch, no clear in-pattern reference module | `software-architect` |
| Implement a UI story — components, state, API integration, accessibility, component tests | `frontend-engineer` |
| Implement a server story — endpoints, services, migrations, integrations, integration tests | `backend-engineer` |
| Pre-PR self-review of the diff (Step 3.5) before invoking `linear-task-agent` to open the PR | `code-reviewer` |
| Step 5 refactoring work against a `tech-debt` Linear issue | `refactor-specialist` |
| Resolve git merge / rebase / cherry-pick conflicts on the working tree, complete the operation, and report what was decided | `conflict-resolver` |
| Inline single-file edits, autocomplete, "rename this variable", small intra-file tweaks | Cursor autocomplete (Supermaven) — no subagent needed |

> **Credentials & Linear-write boundary (all six specialists).** Each subagent inherits the developer's local credentials and repo permissions; it cannot escalate. None of them may transition Linear state, post Linear comments, or open PRs — those writes belong exclusively to `linear-task-agent` so the audit trail stays single-sourced. None of the six may push, force-push, or merge — only the developer pushes after review. [`conflict-resolver`](#conflict-resolver)'s operating boundaries are stricter — see its subsection below.

> **Specialist classification.** Each specialist falls into one of three categories. **Implementation** (`frontend-engineer`, `backend-engineer`, `refactor-specialist`) — may read any file, run tests / lint / format / type-check, and write code on the current branch. **Analysis** (`software-architect`, `code-reviewer`) — read-only; may read any file and run dry-run checks but may not edit, stage, or push. **Utility** (`conflict-resolver`) — modifies the working tree, but only to complete an already-started git operation; see its subsection for the full boundary set.

> **Escalation to human architect.** If an implementation specialist hits an architectural question (new pattern, cross-service contract, technology choice), it must surface the question to the developer and recommend a `software-architect` pass for per-story design, or — for system-wide / new-tech / new-service-boundary decisions — a Phase 2 human-architect review. `software-architect`'s mirror responsibility runs in the same direction: per-story design lives inside the agent, but system-wide decisions escalate to human-architect review. Both routes lead to the same destination — architectural decisions that exceed per-story scope are humans-only.

#### `software-architect`

**Purpose / when to invoke.** Drives Step 3.0 — a read-only, per-story design pass run between Step 2.4 (context pull) and Step 3.1 (scaffold) for non-trivial stories: cross-module touch, schema change, new endpoint surface, new external integration, or any story without a clear in-pattern reference module. Produces a markdown architecture plan in the developer's session — interfaces, data flow, file/module touch list, test strategy, risks, and an explicit "scaffold-ready" verdict — using the [`architecture-design`](./PROMPTS.md#architecture-design) prompt. The plan is the authoritative input to Step 3.1; implementation specialists must not be invoked until the developer explicitly approves it. Trigger phrases: "design the architecture for ENG-XXX", "scope the technical plan before I scaffold", "what's the design for this story", "design before I touch code".

**When NOT to invoke.** In-pattern stories that follow an existing reference module → confirm the pattern in the kickoff comment and go straight to Step 3.1. Behaviour-preserving refactors → `refactor-specialist`. Pre-PR self-review of a finished diff → `code-reviewer`. Post-open PR review → CodeRabbit + human reviewer in Step 4. Linear writes / branch / PR open → `linear-task-agent`. System-wide architecture decisions, new-tech evaluation, or new service-boundary choices → escalate to a Phase 2 human-architect review; per-story design is this agent's remit, system-wide design is not.

**Operating boundaries.**
- **Read-only.** Must not edit any file, stage any change, run any mutating command, or run any command that touches a production-like service. Dry-run linter / type-checker and `git diff` / `git log` are allowed.
- May read any file in the repo, the linked Linear issue, the ADRs, and the OpenAPI spec.
- May use **WebSearch** and **WebFetch** to consult current library / framework documentation when the design depends on external API shape — cite the source in the plan.
- Must produce the plan in the format the [`architecture-design`](./PROMPTS.md#architecture-design) prompt specifies — interfaces, data flow, file/module touch list, test strategy, risks, scaffold-ready verdict.
- Must explicitly cite the in-repo reference module(s) or ADR(s) the plan follows. If none exists, must say so and route the question to a human architect rather than invent one.
- Must end the plan with a developer-approval gate: the implementation specialists may not be invoked until the developer says approved (explicitly), so the boundary is visible in the report itself.
- May recommend a 3–5 line plan summary as a Linear kickoff comment, but must not call Linear MCP — `linear-task-agent` posts the comment.
- Must never call Linear MCP, push, open a PR, or transition state.

#### `frontend-engineer`

**Purpose / when to invoke.** Drives the implementation half of UI stories in Steps 3.1–3.3 — component scaffolding, state wiring, API integration, accessibility (WCAG 2.2 AA), and component-level tests written in the same commit. The agent embeds the [`feature-scaffolding`](./PROMPTS.md#feature-scaffolding-for-cursor-composer-or-claude-code) and [`test-generation`](./PROMPTS.md#test-generation) prompts and is calibrated to the repo's component library and test runner via placeholders. Trigger phrases: "build the X screen", "wire up the new dashboard tile", "add the form for ENG-247", "make this component accessible", "write the component tests for X".

**When NOT to invoke.** Server endpoints, migrations, or anything outside the frontend tree → `backend-engineer`. Pre-PR review of the diff → `code-reviewer`. Behaviour-preserving cleanups against a `tech-debt` issue → `refactor-specialist`. Linear writes, branching, or PR open → `linear-task-agent`. New design-system primitives or visual-language decisions → escalate to a Phase 2 design-system review; do not invent.

**Operating boundaries.**
- May read any file; may write only inside the frontend tree (`{{FRONTEND_ROOT}}`).
- May run `{{TEST_RUNNER}}`, the linter, the formatter, and the type-checker; must run them after every meaningful edit.
- Must follow the existing component library at `{{COMPONENT_LIBRARY_PATH}}` — no new primitives without escalation.
- Must produce accessible markup (semantic HTML, ARIA where needed, keyboard reachability) and add at least one accessibility test per new interactive component.
- Must never call Linear MCP, never push, never open a PR.

#### `backend-engineer`

**Purpose / when to invoke.** Drives the implementation half of server-side stories in Steps 3.1–3.3 — endpoint handlers, service-layer business logic, repository / data-access code, schema migrations, third-party integrations, and integration tests. Embeds the [`feature-scaffolding`](./PROMPTS.md#feature-scaffolding-for-cursor-composer-or-claude-code) and [`test-generation`](./PROMPTS.md#test-generation) prompts and is calibrated to the repo's framework, ORM, and test runner via placeholders. Trigger phrases: "implement the OAuth callback handler", "add the endpoint for ENG-247", "wire up the migration for X", "build the integration with Stripe", "write the integration tests for the orders service".

**When NOT to invoke.** Anything inside the frontend tree → `frontend-engineer`. Pre-PR review of the diff → `code-reviewer`. Behaviour-preserving cleanups against a `tech-debt` issue → `refactor-specialist`. Linear writes, branching, or PR open → `linear-task-agent`. New service boundaries, datastore choices, or cross-service contracts → escalate to a Phase 2 system-architect-style review.

**Operating boundaries.**
- May read any file; may write only inside the backend tree (`{{BACKEND_ROOT}}`) and the migrations directory (`{{MIGRATIONS_PATH}}`).
- Must run `{{TEST_RUNNER}}`, the linter, the formatter, and the type-checker after every meaningful edit.
- Must keep business logic in the service layer; controllers stay thin; data access only through the ORM. No raw SQL outside migrations and explicitly approved repository methods.
- Migrations are forward-only and reversible. Never edit a migration that has shipped to any environment.
- Must never call Linear MCP, push, open a PR, or commit secrets / connection strings.

#### `code-reviewer`

**Purpose / when to invoke.** Performs the pre-PR self-review in Step 3.5 — produces a severity-ranked issue list across correctness, security, performance, error handling, edge cases, naming, tests, and docs. **Read-only**: it reports, it does not patch. Complements CodeRabbit (which runs *after* PR open) by catching issues before review bandwidth is spent. Embeds the [`self-review`](./PROMPTS.md#self-review-before-pr) prompt. Trigger phrases: "review my diff before I open the PR", "self-review this branch", "what's wrong with this change", "anything to fix before I PR".

**When NOT to invoke.** Implementing fixes the review surfaces — that's `frontend-engineer` or `backend-engineer`. Reviewing already-open PRs — that's CodeRabbit + the human reviewer in Step 4. Architectural review of new patterns — escalate to a Phase 2 system-architect-style review.

**Operating boundaries.**
- **Read-only.** Must not edit any file, run any test that mutates state, or stage any change. If the developer asks for a fix, refuse and redirect to `frontend-engineer` / `backend-engineer`.
- May read any file in the repo, run the linter and type-checker in dry-run mode, and inspect git history.
- Must rank every finding as Critical / High / Medium / Low with the exact rule from the [`self-review`](./PROMPTS.md#self-review-before-pr) prompt: Critical and High **must** be fixed before PR; Medium/Low may be dismissed with a one-line reason.
- Must propose the fix inline (per the prompt) but **must not apply it**.
- Must check the diff against the AC of the linked Linear issue if the developer provides the identifier — flag any AC bullet without a matching code change or test.
- Must never call Linear MCP, push, or open a PR.

#### `refactor-specialist`

**Purpose / when to invoke.** Drives Step 5 refactoring against a `tech-debt`-labelled Linear issue: behaviour-preserving structural improvements (extract service, rename pattern across codebase, simplify long methods, dependency upgrades with API change). Embeds the [`refactor-candidates`](./PROMPTS.md#refactoring) prompt and the [`test-generation`](./PROMPTS.md#test-generation) prompt for any new structural seams. Trigger phrases: "refactor the orders service", "extract X into its own module", "clean up the long method in Y", "execute tech-debt issue ENG-XXX".

**When NOT to invoke.** Feature work — that's `frontend-engineer` / `backend-engineer`. Pre-PR review — that's `code-reviewer`. Linear writes / PR open — `linear-task-agent`. **Anything that changes externally observable behaviour** — split into a feature story instead. Identifying refactor candidates with no Linear issue yet — first run the `refactor-candidates` prompt directly to scope and file the issue, then come back here.

**Operating boundaries.**
- May only operate against an open Linear issue with the `tech-debt` label. If the developer cannot supply the identifier, refuse and redirect to file one first.
- Must read the existing test suite for the affected module **before** editing. If coverage is below the threshold for the area being refactored, stop and tell the developer to add characterisation tests first — refactoring without tests is gambling.
- Must run the full relevant test suite after every meaningful edit. Any pre-existing test that flips from green to red is the signal that behaviour drifted; revert and re-plan.
- Must produce one PR against one `tech-debt` issue — never bundle with feature work, never bundle multiple `tech-debt` issues into one diff.
- Must never add new functionality, fix unrelated bugs, or change formatting outside the refactor scope. If a bug is found, surface it as a separate Linear issue and continue with the refactor.
- Must never call Linear MCP (filing the issue is `linear-task-agent`'s job), push, or open a PR.

#### `conflict-resolver`

**Purpose / when to invoke.** Utility specialist invoked **on demand** whenever a git operation produces conflicts that block forward motion — there is no fixed step it owns. The four most frequent touchpoints, in order: (1) Step 4.1, when the developer rebases the feature branch onto main before opening the PR and the rebase stops on conflicts; (2) Step 4.5, when main has moved during review and the merge button is blocked by conflicts; (3) mid-Step-3, when a long-running feature branch needs to pull the latest main; (4) Step 5, when refactor branches rebase against main. The agent walks each conflict hunk, applies a documented selection-or-combination rule (never blind "keep ours" / "keep theirs"), stages the resolved files, runs the appropriate `--continue`, and produces a per-hunk report of what was decided so the developer can spot a wrong call before it ships. Trigger phrases: "resolve these merge conflicts", "fix the conflicts on this rebase", "I'm stuck in a rebase, walk me through it", "continue the rebase", "merge main into my branch", "the cherry-pick is conflicting".

**When NOT to invoke.** Authoring new feature code → `frontend-engineer` / `backend-engineer`. Reviewing a clean (non-conflicted) diff → `code-reviewer`. Behaviour-preserving cleanups → `refactor-specialist`. Pre-implementation design → `software-architect`. Opening PRs, transitioning Linear state, or any Linear write → `linear-task-agent`. **Aborting a rebase / merge / cherry-pick without explicit user instruction** — this is the agent's hard refusal; reflexive `--abort` is what loses already-done resolution work and is the single largest failure mode the agent exists to prevent.

**Operating boundaries.**
- **Modifies the working tree, but only for git-operation completion.** May resolve conflict markers in files that are *currently* in a conflicted state, run `git add` on resolved files, and run `git merge --continue` / `git rebase --continue` / `git cherry-pick --continue`. May not edit files that are not currently conflicted; may not refactor adjacent code "while it's open"; may not add TODOs, emojis, comments, or any content not present on at least one side of the conflict. Resolution is *selection-or-combination*, not authoring.
- **Never runs `git push` in any form** — including `--force` and `--force-with-lease`. The developer pushes after reviewing the per-hunk report.
- **Never runs `git rebase --abort`, `git merge --abort`, or `git cherry-pick --abort` without an explicit developer instruction.** If a hunk is ambiguous, the agent stops on that hunk and surfaces the choice to the developer rather than discarding the work already done across earlier hunks.
- **Never blindly keeps one side.** Each hunk is analysed against documented rules (e.g., schema migrations are forward-only; lockfiles regenerate, not merge; tests merge by union of cases unless the same case is rewritten on both sides) and the chosen rule is named in the per-hunk report.
- **Never calls Linear MCP, opens PRs, transitions issue state, or comments on issues** — those writes belong exclusively to `linear-task-agent` so the audit trail stays single-sourced.
- May run `git status`, `git diff`, `git log`, `git show`, `git ls-files --unmerged`, and the test / lint / type-check commands after `--continue` to verify the working tree is sane before handing back to the developer.
- Inherits the developer's local credentials and repo permissions; cannot escalate.

##### Choosing between a specialist and Cursor inline

> **Rule of thumb:** if the work is **cross-file or multi-step** (scaffolds a new feature, refactors across modules, generates tests for a whole service, runs a structured pre-PR review), use a specialist subagent — it carries the right system prompt and runs in its own scoped session. If the work is **intra-file and incremental** (rename a variable, accept the next autocomplete, add one missing branch in a conditional, tweak a JSX prop), Cursor's Supermaven autocomplete is faster and lower-friction. The specialist is the framing tool; Cursor is the typing tool. Most stories use both — open the specialist for the scaffold and the tests, drop into Cursor for the small edits in between.

#### Creating your own Claude Code subagent — step-by-step

The seven subagents above (`linear-task-agent`, `software-architect`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`, `conflict-resolver`) cover the Phase 3 defaults. Teams routinely need more — a `db-migration-reviewer`, a `release-notes-writer`, a `flaky-test-detective`, a `dependency-upgrader`. This sub-section explains, in plain terms, how a developer creates a brand-new subagent and gets the team using it. **No special tooling required — a subagent is just a markdown file Claude Code reads on startup.**

> **What a subagent actually is.** A markdown file with a YAML header at the top and a system prompt as the body. Claude Code scans `.claude/agents/` (project scope) and `~/.claude/agents/` (user scope) on every session start and registers each file as an invokable role. When you say *"Use the db-migration-reviewer to check this PR's migration"*, Claude Code spins up a fresh sub-session with that file's system prompt loaded. The sub-session has its own context window, inherits your local credentials and MCP connections, and reports its result back to your main session when done.

> **Subagent vs. skill — when to reach for which.** A **subagent** is a *scoped sub-session* — it runs in its own context window, owns a multi-turn conversation, can call tools, and reports back. Reach for it when the work is conversational (review my diff, fetch the next story, implement this feature) and you want isolation from your main thread. A **skill** is a *procedure that runs in your current session* — Claude loads its instructions when triggered and follows them right where you are. Reach for it when the work is a deterministic recipe you want every developer (including future-you) to follow the same way (open a PR, write a plan, audit CLAUDE.md). The how-to for skills is in the next sub-section.

##### Step-by-step: build a new subagent

The fastest path is the built-in `/agents` interactive command — it scaffolds the file for you. The hand-written path is a few extra minutes but gives you full control and is the right call when you want to commit the file straight to the repo.

**Path 1 — Interactive scaffold via `/agents` (recommended for first-timers).**

1. Open Claude Code in the repo root: `claude`.
2. Type `/agents` and pick **Create new agent** from the menu.
3. Pick the scope: **Project** (writes to `.claude/agents/`, shared with the team via git) or **Personal** (writes to `~/.claude/agents/`, only yours). For team workflows, almost always pick Project.
4. Type a short kebab-case name — e.g., `db-migration-reviewer`.
5. Either describe the role in plain English and let Claude generate the system prompt for you, or paste your own prompt. Generated prompts are a fine starting point; refine them before committing.
6. Pick the model (default Sonnet for most coding work, Opus for analysis-heavy or high-stakes roles like reviewers and refactorers).
7. Optionally restrict the tool list. By default the subagent inherits every tool the parent session has. For read-only roles (reviewers, auditors) explicitly narrow to read-only tools so an accidental "go fix it" prompt cannot mutate the repo.
8. Save. Claude Code writes the file and lists it on `/agents` immediately — no restart needed.

**Path 2 — Hand-write the file (full control).**

1. Create the directory if it does not exist: `mkdir -p .claude/agents`.
2. Create `.claude/agents/<name>.md` (or `~/.claude/agents/<name>.md` for personal scope). The name in the filename, the `name:` in the frontmatter, and the slug used to invoke the subagent must match exactly.
3. Write the frontmatter and body. Minimum viable shape:
   ```markdown
   ---
   name: "db-migration-reviewer"
   description: "Use this agent to review a database migration before it is merged: checks reversibility, locking risk on large tables, index strategy, and forward-compatibility with currently-running app code. Read-only. Trigger phrases: 'review this migration', 'is this migration safe', 'check the migration in this PR'. Do NOT invoke for: writing the migration (backend-engineer), reviewing non-migration code (code-reviewer), or applying fixes."
   model: opus
   tools:
     - Read
     - Grep
     - Bash
   ---

   You are the Database Migration Reviewer subagent for Phase 3 (Development) in the AI-DLC framework. Your single responsibility is to audit a pending migration for safety, reversibility, and operational risk before it ships.

   ## Operating boundaries
   - Read-only. Never edit a file, never run a destructive command, never connect to a production datastore.
   - You inherit the developer's local credentials. You cannot escalate.
   - You may run the migration tool in dry-run / plan mode only.

   ## How you produce a review
   1. Identify the migration file(s) in the diff. If none, refuse and tell the developer.
   2. For each migration, score risk across: reversibility, table-lock duration, index creation strategy (online vs. blocking), data-backfill safety, and forward-compatibility with the previous app version (the rollback case).
   3. Output a severity-ranked report: Critical / High / Medium / Low. Critical = data loss or extended outage risk; High = degraded availability under load; Medium / Low = style / convention.
   4. End with an explicit "safe to merge" verdict (yes only if zero Critical and zero High) and the recommended next subagent (`backend-engineer` to apply fixes).

   ## Hand-offs you must escalate, never resolve yourself
   - The migration drops a column or table → flag Critical, request confirmation that no live app version reads it.
   - The migration adds a non-null column without a default → flag High, recommend the two-step pattern (add nullable, backfill, set non-null in a follow-up).
   - The developer asks you to apply a fix → refuse and redirect to `backend-engineer`.
   ```
4. Save the file. Claude Code picks it up on the next prompt — no restart needed.

##### Frontmatter cheat-sheet

| Field | Required | What it does | Tips |
|-------|----------|--------------|------|
| `name` | Yes | The slug used to invoke the agent (`Use the <name> subagent to …`). Must match the filename stem. | kebab-case, role-descriptive (`db-migration-reviewer`), not author-specific (`alices-agent`). |
| `description` | Yes | The router. Claude Code reads this to decide when this subagent applies. | **Lead with when to invoke**, then trigger phrases, then explicit "Do NOT invoke for" cases. A vague description means the subagent never gets picked up automatically. |
| `model` | No | Override the session model. `sonnet` for high-throughput coding; `opus` for review/refactor/audit work; `haiku` for very narrow lookups. | If omitted, inherits the parent session's model. |
| `tools` | No | Allow-list of tools the subagent can call. | Omit to inherit the parent's full toolset. Narrow it explicitly for read-only roles (`Read`, `Grep`, `Bash` only — no `Edit`, `Write`). |

##### Writing a good system prompt

The body of the file is the system prompt for every invocation of this subagent. The four existing specialists in [`./subagent-prompts/`](./subagent-prompts/) are the reference template. The pattern is:

1. **One-sentence role statement** — *"You are the X subagent for Phase 3 (Development). Your single responsibility is to …"*. Single responsibility, single sentence. No second sentence.
2. **Operating boundaries** — a bulleted list of what you may and may not do. Always include: credential inheritance ("you cannot escalate"), explicit out-of-scope writes (must never call Linear MCP / push / open a PR — those belong to `linear-task-agent`), and any read-only constraints.
3. **How you produce the output** — a numbered procedure. Each step should be a concrete action a session can take. Reference the relevant prompt template in [`PROMPTS.md`](./PROMPTS.md) and embed it by reference, not by copy — keeping PROMPTS.md as the source of truth.
4. **Hand-offs you must escalate** — a bulleted list of situations the subagent must surface to the developer instead of resolving. Architectural questions, disabled tests, workflow-infrastructure edits (`.claude/agents/*`), and any pressure to break a boundary all belong here.

> **The single responsibility rule.** If the description has the word "and" in its first clause ("Use this agent to review **and** apply fixes …"), split into two subagents. Mixed-responsibility agents are the single biggest cause of "the subagent helpfully did something I did not want" — typically because the operator typed a prompt that activated the wrong half of the role.

##### Test the new subagent before committing

1. **Discovery.** `/agents` in Claude Code lists the new role with its description. If it does not appear, the file is malformed (most often: missing `---` fence around the frontmatter, or `name:` differs from the filename).
2. **No-write smoke test.** Run a prompt that exercises the agent's core procedure without writing anything: *"Use the db-migration-reviewer to dry-run a review of the migration in this branch and print the severity-ranked list — do not edit any file."* Confirm the output structure matches what the system prompt promised.
3. **Boundary test.** Ask the subagent to do something out of scope: *"Use the db-migration-reviewer to fix the unsafe migration and commit the change."* It must refuse and redirect. If it complies, the boundaries in the system prompt are not strong enough — tighten the operating-boundaries section and re-test.
4. **Negative-routing test.** Ask Claude Code in plain English to do something the new subagent should **not** handle — e.g., *"Write a new migration for the orders table."* — and confirm it does not auto-route to `db-migration-reviewer`. If it does, the description's "Do NOT invoke for" list is too thin; expand it.

##### Commit and roll out

- Project-scope subagents at `.claude/agents/*` are **shared team infrastructure**. Treat edits as code changes — open a PR, get it reviewed by another developer, and update the [`subagent-prompts/`](./subagent-prompts/) reference directory if the new role is broadly reusable (so other repos can copy the template).
- Personal-scope subagents at `~/.claude/agents/*` are private to the developer. Useful for experiments before promotion, or for roles that are genuinely individual (e.g., a personal `learning-journal` agent). They never get to the team's audit trail — do not put any Linear-writing or PR-opening role at user scope.
- Announce the new role in the team channel with one line on **what triggers it**, **what it does**, and **what it explicitly will not do**. Without this, the subagent exists but no one invokes it.

> **Iteration.** Subagents drift the same way prompts drift — a few weeks of edge cases reveal gaps in the boundaries and the procedure. Treat the system-prompt body as living documentation: when the team agrees on a new rule ("migrations adding indexes must always use the concurrent / online variant"), update the relevant subagent's prompt and ship it as a PR. The next session everyone opens picks up the new rule automatically.

#### Creating your own Claude Code skill — step-by-step

Skills are Claude Code's second extensibility surface. Where a **subagent** spins up a scoped sub-session for a multi-turn conversation, a **skill** is a *procedure that loads into your current session when triggered* — Claude reads the skill's instructions and follows them right here, in the conversation you are already having. Skills are the right shape for deterministic recipes the whole team should follow the same way: "how we open a PR", "how we file a follow-up bug", "how we run the pre-merge checklist", "how we generate release notes from Linear".

> **What a skill actually is.** A folder under `.claude/skills/<skill-name>/` (project scope) or `~/.claude/skills/<skill-name>/` (user scope) containing a `SKILL.md` file. The `SKILL.md` has a YAML header (`name`, `description`) and a body with the procedure — step-by-step instructions, checklists, templates, gotchas. The folder may also contain supporting files (templates, reference docs, example outputs) which the skill body references by relative path. Claude Code scans both scope paths on startup and registers each skill as invokable via `/<skill-name>` and as auto-triggerable when a user's prompt matches the `description`.

##### When to build a skill vs. a subagent vs. a slash command

| Need | Reach for | Why |
|------|-----------|-----|
| A multi-turn conversation in its own scoped context (review a diff, drive a story end-to-end, implement a feature) | **Subagent** | Owns a sub-session with its own context window. Can take many tool calls, branch on observations, and report back. |
| A deterministic procedure that should run the same way every time (file a follow-up bug, open a PR, write a Phase plan, run a CLAUDE.md audit) | **Skill** | Loads its instructions into the *current* session. The user invokes it explicitly (`/file-followup`) or Claude auto-triggers when the prompt matches. |
| A one-shot prompt with no procedure (e.g., "summarise this file") | **Slash command** (`.claude/commands/<name>.md`) | Fires one prompt with arguments. No procedure, no checklist, no supporting files. |

> **A useful test.** If you find yourself writing the phrase *"first, then, then, then, finally"* in plain English to teach the team how to do something — that is a skill. If you find yourself writing *"you are the X agent, your responsibility is …"* — that is a subagent.

##### Step-by-step: build a new skill

1. **Pick a name and scope.** Kebab-case, action-oriented (`open-pull-request`, `file-followup-bug`, `audit-claude-md`), not noun-only (`pull-requests` is too vague). Use project scope (`.claude/skills/`) for anything the team should share; user scope (`~/.claude/skills/`) for personal recipes you are still experimenting with.
2. **Create the folder.** `mkdir -p .claude/skills/<skill-name>`. The folder name **must** match the `name:` in the frontmatter — Claude Code uses the folder name as the invocation slug.
3. **Author `SKILL.md`.** Minimum viable shape:
   ```markdown
   ---
   name: file-followup-bug
   description: Use when a developer wants to file a follow-up bug discovered during Phase 3 implementation — typically a bug the current story is not scoped to fix. Triggers on phrases like "file a bug for X", "follow-up issue for the auth race", "track this for later", "log a tech-debt item". Creates a Linear issue with the right project, labels, parent linkage, and reproduction steps formatted to PROMPTS.md → bug-report. Do NOT use for: bugs the current story IS scoped to fix (those go straight into the current branch), or for filing brand-new feature ideas (use the product backlog instead).
   ---

   # File a follow-up bug

   You are guiding the developer through filing a follow-up bug discovered mid-story. The canonical procedure lives in [PROCESS.md → Step 3 / follow-ups](../../aidlc-phases/03-development/PROCESS.md). The Linear write is performed exclusively by the `linear-task-agent` subagent — this skill produces the structured payload it consumes.

   ## Procedure

   1. Confirm the bug is genuinely out of scope for the current story. Ask the developer one line: *"Is this in the current story's AC? (yes / no)"* If yes, refuse — the bug must be fixed in the current branch, not deferred.
   2. Gather the minimum payload:
      - One-line title (imperative, no ticket prefix — Linear adds it).
      - Reproduction steps (numbered).
      - Expected vs. actual behaviour (two lines max each).
      - Affected component / file path.
      - Suggested severity (Critical / High / Medium / Low) with one-line justification.
      - Optional: a link to the line(s) in the current branch that surfaced it.
   3. Format the payload using the [`bug-report`](../../aidlc-phases/03-development/PROMPTS.md#bug-report) template from PROMPTS.md. Do not deviate from the template; the team filters Linear by its structure.
   4. Hand the payload to `linear-task-agent` with: *"File this as a new bug in {{LINEAR_PROJECT}}, label `bug`, parent = the current story, assignee = unassigned. Confirm the new issue ID and post it back to me."*
   5. Once `linear-task-agent` returns the new issue ID, paste it back into the current branch as a code comment near the surfacing line: `// follow-up: ENG-1234`.

   ## Refusal cases

   - Bug is in the current story's AC → refuse, fix in the current branch.
   - No reproduction steps → refuse, ask the developer to produce them first; never fabricate.
   - Developer asks you to call Linear MCP directly → refuse, hand to `linear-task-agent` (sole Linear writer).
   ```
4. **Optional: add supporting files in the folder.** A skill folder can hold templates, example outputs, or reference docs the skill body links to. Common patterns:
   - `templates/issue-body.md` — a fill-in-the-blank Markdown template.
   - `examples/good-bug.md`, `examples/bad-bug.md` — known-good and known-bad references the skill cites.
   - `reference/severity-rubric.md` — long-form rubric the skill body summarises and links.
   Reference these files from `SKILL.md` by **relative path** so they resolve regardless of where the repo is checked out.

##### Frontmatter cheat-sheet

| Field | Required | What it does | Tips |
|-------|----------|--------------|------|
| `name` | Yes | The invocation slug. `/file-followup-bug` runs the skill explicitly. Must match the folder name exactly. | kebab-case, action-verb-led. |
| `description` | Yes | **The trigger.** Claude Code reads this on every prompt and decides whether to auto-load the skill. A weak description means the skill is never auto-triggered and only fires when a developer remembers the slash command. | **Lead with "Use when …"**. List trigger phrases. Add explicit "Do NOT use for …" cases. One paragraph, not a sentence. |

> **The description is the entire routing surface.** Unlike subagents (which a developer usually invokes by name), skills are designed to auto-trigger from user intent. If a developer asks *"Can you file a bug for that auth race?"*, the `file-followup-bug` skill should load automatically — but only if its description contains language close enough to "file a bug" for Claude to match. Write descriptions imagining the messiest plausible user phrasing, not the cleanest.

##### Writing a good skill body

1. **Open with the role and the source of truth.** One sentence — *"You are guiding the developer through X. The canonical procedure lives in [PROCESS.md → Step Y]."* This anchors the skill in the AI-DLC docs and makes it auditable.
2. **Procedure as a numbered list.** Each step should be one concrete action — confirm scope, gather payload, format with template, hand off. If a step branches, write the branches inline (*"If yes, refuse. If no, continue."*). Avoid open-ended prose.
3. **Refusal / escalation cases.** A bulleted list at the end. This is the equivalent of a subagent's "operating boundaries" — it stops the skill from doing something the team did not authorise.
4. **Reference, do not duplicate.** Link to PROMPTS.md, PROCESS.md, and template files by relative path. The whole point of a skill is to keep the team's procedure in one place — duplicating it inside the skill body creates two places to maintain and inevitably one falls behind.
5. **Keep it short.** A skill body that runs past two screens is doing too much. Split into two skills or move detail into supporting files that the body links to.

##### Test the new skill before committing

1. **Discovery.** Open Claude Code in the repo. The skill should appear in the available-skills list under the slug `<skill-name>`. If it does not, the folder name, the `name:` field, and the invocation slug are out of alignment — fix and reload the session.
2. **Explicit-invocation test.** Type `/<skill-name>` and pass a representative input. The skill should run its procedure top-to-bottom and end at the documented hand-off. Confirm the output format matches the body's promise.
3. **Auto-trigger test.** Phrase a request the description claims to handle, in messy human language — *"hey can you log a follow-up thing for that timing bug we just noticed"*. Claude should auto-load the skill. If it does not, the description's "Use when …" clause or trigger-phrase list is too narrow.
4. **Refusal test.** Drive the skill into one of its refusal cases — *"file a bug for the missing AC bullet on the current story"*. The skill must refuse and redirect, not file the bug. If it complies, tighten the refusal list and re-test.
5. **Hand-off integrity test.** If the skill hands off to a subagent (most do — e.g., `file-followup-bug` hands off to `linear-task-agent`), confirm the payload format is exactly what the receiving subagent expects. A skill that hands off the wrong shape is worse than no skill — it produces a confident-looking failure.

##### Commit and roll out

- Project-scope skills at `.claude/skills/*` are shared team infrastructure. Treat edits as code changes; PR-review them. Cross-link the new skill from the relevant PROCESS.md step (so a developer reading the phase doc discovers the skill at the moment of need).
- Personal-scope skills at `~/.claude/skills/*` are private. Useful for experiments. Promote to project scope once the skill has been used a handful of times and the procedure is stable.
- A skill that hands off to a subagent (the common pattern in the AI-DLC dev loop — skills *gather* and *format*; subagents *act*) should name the receiving subagent in its description so the chain is discoverable: *"… hands the payload to `linear-task-agent` to perform the Linear write."*
- Announce the new skill in the team channel with three lines: **what triggers it**, **what it produces**, and **which subagent it hands off to**. Without this, skills exist but never auto-trigger because no one phrases their prompts to match.

> **The single-procedure rule.** A skill encodes one procedure. If you find yourself writing *"and then, if it's a frontend bug, do X, but if it's a backend bug, do Y"* — that is two skills (`file-followup-frontend-bug`, `file-followup-backend-bug`) or one skill that hands off to two different subagents at step 4. Branching procedures are skills that will silently take the wrong branch under pressure.

#### Verification checklist

- [ ] `claude mcp list` shows `linear: connected` for every developer
- [ ] Smoke test: `Via Linear MCP, fetch my next In-Progress issue and print its branchName.` returns a real issue with a `branchName` field
- [ ] Linear → Settings → Integrations shows GitHub/GitLab/Bitbucket connected with PR auto-link on
- [ ] Anthropic admin policy: `update_issue`, `assign_issue`, `create_issue` enabled; `delete_issue` disabled
- [ ] `.claude/agents/linear-task-agent.md` committed to the repo; `/agents` in Claude Code lists `linear-task-agent`; the subagent smoke test returns a real `branchName` without writing state
- [ ] Each Phase 3 specialist subagent (`software-architect`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`, `conflict-resolver`) listed by `/agents` and committed under `.claude/agents/`; team has agreed on the model + tool-permission defaults for each
- [ ] Team understands the **new-subagent recipe** (`/agents` interactive flow, frontmatter shape, system-prompt structure, four pre-commit tests: discovery, no-write smoke, boundary, negative-routing) and the single-responsibility rule before committing any new role
- [ ] Team understands the **new-skill recipe** (folder + `SKILL.md` layout, description-as-router rule, four pre-commit tests: discovery, explicit invocation, auto-trigger, refusal) and the single-procedure rule before committing any new skill
- [ ] Audit log on in Linear (Workspace settings → Security → Audit log)

> **Permission inheritance:** Claude Code inherits the connecting developer's Linear permissions — no privilege escalation.

#### Linear Workspace Setup (Phase-3 additions)

The Phase 1 workspace setup (Initiative → Project → Milestone → Issue, plus the `phase:*`, `ai-generated`, `needs-human-review` labels) carries over. Phase 3 adds:

**Labels:**

| Label | Meaning |
|-------|---------|
| `tech-debt` | Refactoring task — no new features allowed in the PR |
| `bug` | Bug filed during development (Phase 4 owns the bulk; some leak in here) |
| `blocked` | Cannot progress until a dependency clears (paired with a Linear blocker link) |
| `agent-authored` | PR was opened by a Linear Agent or Claude Code background agent — extra reviewer scrutiny |

**Saved views the Tech Lead should configure:**

1. **Cycle Active** — `cycle = current AND state != Done` (the daily standup view)
2. **My In Progress** — `assignee = me AND state = In Progress` (the developer's own view)
3. **In Review** — `state = In Review` (the review queue)
4. **Tech Debt** — `label = tech-debt AND state != Done`
5. **Blocked** — `label = blocked` (escalation queue)

**Cycle hygiene:**
- One issue, one PR. If a story needs multiple PRs, decompose into sub-issues before starting work — each sub-issue gets its own PR and `Closes`.
- No story stays in `In Progress` longer than the cycle. If it does, comment why and split it.
- Cycle close = retro. AI-estimate variance, defect escape rate, and PR cycle time get reviewed every cycle.

---

### Step 1: Sprint Planning & Estimation

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-sprint-planning)

| Attribute | Detail |
|-----------|--------|
| **Input** | Phase 1 backlog (Linear Project + Milestones + accepted Issues), Phase 2 architecture, ADRs, OpenAPI spec, sprint goal, team capacity |
| **Tool** | **Claude Code** with Linear MCP — pulls stories, generates estimates, posts back as comments |
| **Output** | Sprint backlog committed in Linear (Cycle), each story carrying an AI estimate and a developer-calibrated estimate |
| **Human** | Tech Lead sets the cycle; team calibrates AI estimates; PM confirms scope |

**Workflow:**

**1.1 — Pull the candidate backlog.** In Claude Code, run the [`linear-sprint-pull`](./PROMPTS.md#linear-sprint-pull) prompt. Claude Code calls `search_issues` filtered by the Phase 1 Project, `state=Backlog`, ordered by priority. The result is a structured list of stories with their AC, milestone, labels, and **`branchName`** (Linear computes this — it is what we will use in Step 2).

**1.2 — Generate estimates per story.** For each candidate, run the [`estimation`](./PROMPTS.md#estimation) prompt with the story body, AC, ADRs, and similar shipped code as context. Claude returns T-shirt size (S/M/L/XL/XXL), Fibonacci points, subtask breakdown, risks, and dependencies.

**1.3 — Post estimates back to Linear.** Run the [`estimates-to-linear`](./PROMPTS.md#estimates-to-linear) prompt. Claude Code uses `update_issue` to set the **estimate** field and `create_comment` to attach the subtask breakdown + risks as a single thread on the issue. Comments are signed `(via Claude Code MCP)` so reviewers know the source.

**1.4 — Team calibration.** During sprint planning, the team walks the AI estimates. Anywhere the team-calibrated estimate diverges from Claude's by > 30%, capture the *reason* in a comment — over time this trains the team's prompt context with codebase-specific friction (legacy modules, test brittleness, infra lag).

**1.5 — Decompose XXL stories.** Any story Claude flags as XXL (> 5 days) **must** be split before commitment. Run the [`story-decomposition`](./PROMPTS.md#story-decomposition) prompt. Claude proposes child stories; run `create_issue` with `parentId` set to the original to create them as **sub-issues**. The parent stays as the tracking story; sub-issues carry the AC slices.

**1.6 — Commit to the cycle.** Tech Lead creates the Linear Cycle (sprint), drags in committed stories, sets the cycle dates. Velocity from prior cycles drives capacity, not Claude's estimates — AI estimates inform, they do not commit.

> **[Gate 1: Sprint Commitment](./QUALITY-GATES.md#gate-1-sprint-commitment) must pass before development begins.** Every committed story has: AC ≥ 3, an estimate, no dependencies blocking, a clear owner.

---

### Step 2: Pick a Story & Start Work — Linear-driven

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-pick--start-a-story)

| Attribute | Detail |
|-----------|--------|
| **Input** | Active Linear Cycle, developer's next assigned issue |
| **Tool** | **Claude Code** with Linear MCP — fetches issue, transitions state, checks out the Linear-suggested branch |
| **Output** | A local branch named after Linear's `branchName`, the issue moved to **In Progress** with the developer assigned, a kickoff comment posted |
| **Human** | Developer reviews AC and architecture references before code is touched |

This step is the daily entry point. It replaces the manual "open Linear → copy ticket ID → invent a branch name → forget to set status" choreography that wastes 5–10 minutes per story.

**Workflow:**

**2.1 — Get the next story.** From the repo root, run Claude Code with the [`linear-next-task`](./PROMPTS.md#linear-next-task) prompt. Claude Code calls `search_issues` with `assignee=me` (Linear MCP self-assigns via the `me` keyword), `cycle=current`, `state=Todo`, ordered by priority. It returns the top one and prints: identifier (e.g., `ENG-247`), title, AC, milestone, **branchName** (e.g., `david/eng-247-oauth2-callback-handler`), PRD section deep-link, related ADRs.

**2.2 — Confirm and transition.** Developer reads the AC and confirms. If the AC is wrong or under-specified, **stop** — comment on the issue and bounce back to the PM. If clear, the same Claude Code session calls `update_issue` to set state to **In Progress** and self-assigns via `me`. A `create_comment` posts: `Started by <user> via Claude Code. Plan: <2-3 line plan>.` This single comment is the audit trail.

**2.3 — Create the local branch from Linear's `branchName`.** Claude Code reads the `branchName` field from the issue object (Linear computes it from team prefix + identifier + slugified title — e.g., `david/eng-247-oauth2-callback-handler`). Claude runs `git checkout -b <branchName>` in the working directory. Linear's git integration will recognise this name on PR open and auto-link the issue without any developer action. **Do not invent your own branch name** — drift between branch and issue breaks the integration.

**2.4 — Pull dependent context.** Run the [`task-context`](./PROMPTS.md#task-context) prompt. Claude reads, in order: the PRD section the issue cites, the relevant ADR(s), the OpenAPI section for any endpoint touched, the existing tests in the affected module. This becomes the working context for Step 3 — no further Linear reads happen until the PR. For **non-trivial stories** (cross-module touch, schema change, new endpoint surface, new external integration, or no clear in-pattern reference module), the next stop is **Step 3.0** (`software-architect` design) before Step 3.1 (scaffold); in-pattern stories that follow an existing reference module skip Step 3.0 and go directly to Step 3.1.

> **Convention:** the *only* Linear write between *In Progress* and PR open should be progress comments. Do not flip the issue back to Todo or Backlog mid-flight; if blocked, comment with the blocker and tag the blocker — Linear's blocker UI handles the rest.

---

### Step 3: Feature Development

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-feature-development)

| Attribute | Detail |
|-----------|--------|
| **Input** | Working branch, story AC, PRD section, ADRs, OpenAPI spec, data model |
| **Tool** | **Cursor** (daily coding) + **Claude Code** (complex multi-file work, MCP-driven integrations) |
| **Output** | Implementation code, unit tests, inline docs, commits on the feature branch |
| **Human** | Architecture alignment, business-logic correctness, code ownership |

**When to use which tool:**

| Task | Tool | Why |
|------|------|-----|
| Non-trivial story design before scaffolding | **Claude Code** with `software-architect` | Read-only design pass; gates the implementation specialists — no scaffold until the developer approves the plan |
| New feature with familiar pattern | **Cursor** Composer | Multi-file scaffolding in the IDE; autocomplete as you code |
| Incremental coding, small edits | **Cursor** autocomplete (Supermaven) | Inline suggestions — fast and low-friction |
| Complex multi-file refactoring or feature | **Claude Code** | Reads entire codebase, runs tests, iterates autonomously |
| Debugging across modules | **Claude Code** | Traces data flows, checks logs, proposes and verifies fixes |
| MCP integration (Linear, Figma, GitHub, Sentry, etc.) | **Claude Code** | Deepest MCP support — built by the team that created MCP |
| Autonomous story implementation | **Claude Code** (background agent or [Linear Agent](#linear-agent-optional-autonomous-pickup) — see below) | For well-scoped stories with clear AC, assign and review output |
| Cross-layer story splittable into parallel slices, multiple sibling stories in flight, or adversarial review/debug | **Multi-agent** — see [Multi-agent development patterns](#multi-agent-development-patterns) | Subagents (in-process), Agent Teams (peer messaging, experimental), git worktrees, or Conductor — picks the right surface for the parallelism shape |
| Quick question about the codebase | **Cursor** Chat | Ask inline without leaving the editor |

**Workflow:**

#### 3.0 Design and approve architecture

(Non-trivial stories only.) Before any code is touched, invoke `software-architect` in Claude Code with the story body, AC, the context pulled in Step 2.4, and the relevant ADR(s). The agent runs the [`architecture-design`](./PROMPTS.md#architecture-design) prompt and returns a markdown plan in the developer's session — interfaces, data flow, file/module touch list, test strategy, risks, and a scaffold-ready verdict — citing the in-repo reference module(s) and ADR(s) the design follows. **User-approval gate:** the developer reviews the plan and must explicitly approve before any implementation specialist (`frontend-engineer`, `backend-engineer`) is invoked; on revisions, the agent re-runs against the developer's feedback. Once approved, the developer asks `linear-task-agent` to post a 3–5 line plan summary as a Linear comment so the audit trail captures the design intent alongside the story. **Skip rule:** in-pattern stories that follow an existing reference module skip Step 3.0 — the developer confirms the pattern in the kickoff comment (`Following the pattern at <module>`) and goes straight to Step 3.1. **Escalation:** if the agent surfaces a system-wide concern (new service boundary, new tech, new pattern that becomes the next reference), it must stop and recommend a Phase 2 human-architect review — per-story design is in scope, system-wide design is not.

**3.1 — Scaffold the feature.** Run the [`feature-scaffolding`](./PROMPTS.md#feature-scaffolding-for-cursor-composer-or-claude-code) prompt either in Cursor Composer (for in-pattern features) or Claude Code (for multi-module). Provide the story body, the relevant ADR, the reference module to follow. **For stories that ran Step 3.0**, the developer-approved architect plan is the authoritative input — the scaffold must match the plan, or the developer must explicitly re-approve a deviation before continuing.

**3.2 — Code with autocomplete.** Cursor's Supermaven engine handles inline suggestions while typing. The skill is to accept selectively — discard suggestions that hallucinate APIs, take ones that match repo patterns.

**3.3 — Write tests alongside code.** Every new function gets a test in the same commit. Use the [`test-generation`](./PROMPTS.md#test-generation) prompt with an example test from the project as style reference. Test coverage is enforced both at the Phase 3 [Gate 2: PR Merge](./QUALITY-GATES.md#gate-2-pr-merge) and downstream in Phase 4 Testing — but the discipline starts here.

**3.4 — Post progress on the Linear issue (every checkpoint).** When a sub-task lands or a non-trivial decision is made, run the [`linear-progress-comment`](./PROMPTS.md#linear-progress-comment) prompt. Claude Code summarises the diff and posts a comment on the issue via `create_comment`. This is the running narrative that PMs and reviewers read instead of pinging the developer.

#### 3.5 Self-review before PR

Run the [`self-review`](./PROMPTS.md#self-review-before-pr) prompt against the diff. Claude returns a severity-ranked issue list across correctness, security, performance, error handling, edge cases, naming, tests, docs. Resolve every Critical and High before opening the PR. **Each fix is a separate commit** — preserves rollback points and gives reviewers a clean history.

#### Linear Agent (optional, autonomous pickup)

For a clearly scoped, low-risk story (typed contract, mature module, full AC), the Tech Lead can **assign the issue directly to a Linear Agent** instead of a human developer. Linear Agents (April 2026 onwards, Business plan and above) are AI teammates that pick up the assigned issue, branch off, implement, run tests, open a PR, and request human review — same lifecycle as a human contributor, with the human assignee staying primary and the agent added as a contributor for accountability. Agent-authored PRs go through the **same** code review gate (Step 4) — there is no shortcut. Use sparingly until the team has calibration data on agent quality on your codebase.

#### Multi-agent development patterns

> **tl;dr** — Most stories: **Pattern A** (subagents, already wired). Two-to-four sibling stories in one cycle: **Pattern C** (worktrees). Three-plus stories on macOS: **Pattern D** (Conductor). Adversarial review / debate: **Pattern B** (Agent Teams, experimental). When parallel branches reach merge time, invoke `conflict-resolver` inside each affected worktree. Skip multi-agent entirely if your AC is vague — multi-agent multiplies ambiguity into divergent implementations.

Single-agent coding has a hard ceiling: one context window, one task in flight, one developer reviewing each step in real time. The patterns in this sub-section break that ceiling by running **multiple Claude Code instances in parallel**, each with its own context, scope, and (usually) its own working tree. The bet is that one engineer supervising 3–5 focused agents ships more than the same engineer driving one agent eight hours a day — provided the work decomposes cleanly and the review discipline holds. The patterns are additive to the rest of Step 3, not replacements: every PR still flows through Step 4, every PR still maps to a Linear identifier, `linear-task-agent` still owns Linear writes. What changes is the implementation half — between *In Progress* and PR open, parts of the work can run concurrently.

The industry framing for this shift, captured in O'Reilly's "Conductors to Orchestrators" (2026) and Addy Osmani's "Code Agent Orchestra" (2026), is that the developer's role moves from **conductor** (synchronous, one agent, real-time steering) to **orchestrator** (asynchronous, multiple agents, plan-up-front + verify-at-end). The conductor model is what Steps 3.1–3.5 above describe; this sub-section is the orchestrator overlay.

> **The bottleneck moves.** With one agent, the bottleneck is generation speed. With three to five agents, the bottleneck is *verification* — review bandwidth, integration friction, and the team's ability to write specs precise enough that parallel runs do not multiply ambiguity into divergent implementations. Plan for verification, not for generation throughput.

##### When multi-agent helps (and when it doesn't)

| Story shape | Use multi-agent? | Why |
|-------------|------------------|-----|
| Cross-layer feature where frontend, backend, and tests can be owned independently | **Yes** | Three teammates each own a layer; the lead synthesises at the end. Classic agent-team sweet spot. |
| Two to four sibling features in the same cycle, no shared files | **Yes** | Each feature in its own worktree; review and merge on independent schedules. |
| Adversarial debugging — competing hypotheses for the same bug | **Yes** | Three teammates each pursue a theory and challenge each other; the surviving theory is more likely the root cause than a single agent's first plausible explanation. |
| Cross-module refactor with clear file boundaries | **Yes (with care)** | One teammate per module, all gated by characterisation tests; one reviewer agent watches for behaviour drift. |
| Sequential story with tight feedback loops (research → spike → implement → test) | **No** | Coordination overhead exceeds parallelism gain. Stay in a single Claude Code session. |
| Same-file edits or many shared dependencies | **No** | Two agents editing the same file overwrite each other. Decompose first, or stay sequential. |
| Spec is vague or the AC is ambiguous | **No** | Vague specs multiply errors across N parallel runs. Fix the spec first (bounce to PM, see Step 2.2). |
| Small intra-file change | **No** | Cursor inline autocomplete is faster and cheaper. |

> **Decision rule:** if you cannot describe each agent's file scope and deliverable in one sentence each before spawning, do not spawn. Vague delegation is the single biggest cause of wasted multi-agent runs (Osmani 2026, internal team retros). Most of the value comes from the *upfront decomposition*, not the parallelism — if the decomposition is shaky, run sequentially and save the tokens.

##### Pattern A — Claude Code subagents (in-process, already adopted)

The six specialist subagents committed in [Step 0](#specialist-subagents-for-phase-3-roles-recommended) — `software-architect`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`, `conflict-resolver` — are already a multi-agent surface. Each runs in its own scoped session with its own system prompt; each reports back to the developer. They parallelise inside a single Claude Code session via the built-in Task tool: the developer (or `linear-task-agent`) can spawn `frontend-engineer` and `backend-engineer` in the same turn, and both run concurrently. `software-architect` runs read-only, so it is safe to parallelise with implementers — for example, designing the next story while the current story's implementation is still in flight. `conflict-resolver` is the exception: because git-conflict resolution needs a single writer on the working tree, it runs alone — never spawned in parallel with implementers, and never two `conflict-resolver` sessions against the same worktree. Subagents always **report results back to the spawning session** — they do not message each other.

**When to reach for this pattern.** Default for cross-layer stories where the frontend and backend slices are independent: spawn both specialists from the same Claude Code session at story-pickup time, let them work in parallel, then merge their findings in the developer's head before the self-review pass. No extra tooling, no extra terminals, lowest token overhead of the four patterns.

**Constraints.** Subagents share the parent session's working tree — if both write the same file they will conflict. Mitigate by giving each specialist an explicit file scope in the spawn prompt (the specialist system prompts already enforce a frontend / backend boundary). For genuinely independent feature streams, prefer Pattern C or D (separate worktrees) so two PRs can land independently.

**Drain in-flight specialists before invoking [`conflict-resolver`](#conflict-resolver).** When multiple implementation specialists are spawned in one session and a rebase or merge becomes necessary mid-flow, let the in-flight specialists complete and report back **first**, then spawn the resolver. Spawning `conflict-resolver` while an `Edit` / `Write` call from a sibling specialist might still land races two writers on the same working tree, and the per-hunk rule-priority analysis the resolver performs becomes invalid against a moving file. The drain is cheap (specialists already report back to the spawning session); the alternative is silent corruption of a resolved hunk.

##### Pattern B — Claude Code Agent Teams (peer messaging, experimental)

Claude Code v2.1.32+ ships an experimental **agent teams** feature: a team lead session spawns named teammates that each have their own context window, can **message each other directly** (not only the lead), and coordinate through a shared task list with file locking and dependency tracking. This is one step beyond Pattern A — teammates can challenge each other, hand off work peer-to-peer, and self-claim the next unblocked task without the lead arbitrating every transition.

**Enable** by setting `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in `~/.claude/settings.json` (`env` block) — the feature is off by default. Teammates inherit the lead's project context (CLAUDE.md, MCP servers, skills) but **not** the lead's conversation history, so the spawn prompt must carry every detail the teammate needs. Subagent definitions can be reused as teammate roles by name (`Spawn a teammate using the code-reviewer agent type to audit src/auth/`); the six specialists committed in Step 0 are immediately reusable here.

**Display.** In-process by default — Shift+Down cycles through teammates, Ctrl+T toggles the shared task list. Split-pane mode requires `tmux` or iTerm2 with the `it2` CLI and gives each teammate its own visible pane (set `teammateMode: "tmux"` in `~/.claude/settings.json`). Split-pane is materially easier when running 3+ teammates for the first time; in-process is fine once muscle memory forms.

**Quality gates via hooks.** The team feature exposes three hooks the team should wire up before adopting it broadly:

- `TaskCreated` — exit code 2 to block creation of low-quality tasks (e.g., a task with no AC).
- `TaskCompleted` — exit code 2 to block premature completion (e.g., tests not green, lint not clean).
- `TeammateIdle` — exit code 2 to send feedback and keep a teammate working when it is about to stop short.

These are the multi-agent equivalent of the per-PR CI gate: enforce them in `.claude/settings.json` so the lead cannot mark work done while CI would fail. Without them, agent teams routinely declare victory while tests are red.

**Rebase handoff to [`conflict-resolver`](#conflict-resolver).** If a teammate's task is "rebase the team's shared branch onto main", the teammate must **not** attempt the rebase in-line — it hands off via the in-process Task tool to `conflict-resolver`, or the developer manually invokes the resolver in the lead session. The shared task list's file-locking does not extend across `git rebase`'s working-tree mutations, so two teammates active on the branch during a rebase is the same single-writer violation as Pattern A. Wire `TaskCompleted` so it does **not** fire green for a rebase task until `git status` shows a clean tree — otherwise teammates self-claim the next task on top of a half-finished rebase, which is the worst possible state and the failure mode the hook exists to prevent.

**Strongest fit.** Adversarial review (one teammate per concern: security, performance, test coverage), competing-hypothesis debugging, and cross-layer features where the frontend and backend teammates need to renegotiate the contract mid-flight without round-tripping through the developer. Less compelling than Pattern A for in-pattern feature work — the peer messaging adds tokens for coordination that simpler subagents do not need.

**Constraints / known limits (as of Claude Code v2.1.32):** experimental — `/resume` and `/rewind` do not restore in-process teammates; task status occasionally lags (a teammate finishes work but does not mark the task complete, blocking dependents); shutdown is not always immediate; one team per lead session; no nested teams. Treat the feature as adoption-ready for research, review, and bounded refactors; treat it as cautious-adopt for production feature work until your team has logged a few cycles of use.

**Token cost.** Each teammate is a full Claude Code instance — token usage scales linearly with team size. Anthropic's own guidance is to start at 3 teammates and only scale up when the work genuinely benefits. A four-teammate session can easily 4–5× a single-session token bill; budget accordingly and prefer the `code-reviewer` specialist (read-only) over a fourth implementer when in doubt.

**Reference.** [Claude Code → Agent Teams](https://code.claude.com/docs/en/agent-teams).

##### Pattern C — Git worktrees for manual parallel sessions

Git worktrees let one repository have **multiple checked-out working trees** at different paths, each on its own branch. Combine that with separate Claude Code sessions and the developer can run 3–5 stories in flight in genuinely independent terminals — zero shared on-disk state, zero risk of one session's edits clobbering another's.

**Mechanics.**

```bash
# From the repo root, create a sibling working tree on a Linear-suggested branch
git worktree add ../<repo>-eng-247 -b david/eng-247-oauth2-callback-handler

# In a new terminal, cd into it and start Claude Code
cd ../<repo>-eng-247
claude
```

The new directory is a fully checked-out copy on its own branch. Multiple worktrees of the same repo share the `.git` directory, so they are cheap on disk (no full clone) and the branches are first-class — `git push`, `git status`, and the Linear ↔ git auto-link all work as if it were a normal clone. When the story merges, prune with `git worktree remove ../<repo>-eng-247`.

**Subagent worktree isolation.** Subagents in Claude Code can be configured to run in their own worktree by setting `isolation: worktree` in the subagent's frontmatter — the worktree is created at spawn, cleaned up on idle if the agent made no changes, otherwise the path and branch come back in the result. Use this for `refactor-specialist` runs (a refactor that fails should not pollute the developer's branch) and for any speculative spike where the developer wants to be able to discard the entire attempt cleanly.

**Strongest fit.** Two to four siblings stories in the same cycle that the developer is happy to drive sequentially in attention but wants to run concurrently on the wall clock. Also the simplest path to "Claude Code is busy refactoring while I write the next story's tests" — the parallelism is in *time*, not in *coordination*.

**Conflict resolution across worktrees.** Worktrees materially expand the conflict surface — each branch ages independently against a moving main — so [`conflict-resolver`](#conflict-resolver) is the load-bearing agent for this pattern.

- **Run the resolver *inside* the conflicted worktree's Claude Code session, never in a sibling worktree or in the main repo's session.** Each worktree is its own working tree on its own branch; invoking the resolver in the wrong worktree resolves nothing useful and corrupts the agent's `git status` parsing. The worktree path **is** the resolver's working directory — `cd` into it before `claude`.
- **Merge train discipline.** When 3–5 sibling worktrees all approach ready-to-land at the same time, merge serially: the first lands clean against main, each subsequent one rebases onto the new main and may produce conflicts. Realistic per-worktree rebase cost for trailing worktrees is N × (one worktree's conflict load) — budget for it in the cycle plan, not on the day. The discipline that keeps this manageable is the file-scope rule already in the existing operational guardrails: the more disjoint the per-worktree file sets at spawn time, the closer the trailing-worktree rebase cost trends to zero.
- **Subagent-isolation worktrees** (the `isolation: worktree` frontmatter used on `refactor-specialist`): if a spawned worktree picks up an upstream change that conflicts mid-run, **abandon the spawn** — the per-spawn worktree exists exactly so it can be discarded — surface the upstream change to the developer, then re-spawn against the new tip. **Do not invoke `conflict-resolver` inside an `isolation: worktree` spawn.** The discard-and-respawn flow is cleaner and the per-spawn cost is by design throwaway; resolving conflicts inside a worktree that is about to be pruned is wasted tokens.
- **Cross-worktree pattern drift.** When the resolver runs in worktree A and resolves an architectural-migration conflict in favour of the newer pattern (Rule 2 in the resolver's priority list), that decision exists in worktree A only. Worktree B still on the old pattern will hit the same conflict on its next rebase. Propagate the decision by landing A first (A's resolution becomes the new main baseline) or by manually applying the same resolution in B **before** A lands. Otherwise B's rebase reproduces the same conflict and the resolver re-derives the same answer — wasteful but not unsafe.

**Constraints.** The developer is the orchestrator — there is no team lead, no shared task list, no inter-agent messaging. Each session is fully isolated from the others, which is a feature for safety and a limit for collaboration. Practical upper bound is ~3–5 worktrees before context-switching between terminals becomes its own bottleneck.

**Reference.** [Claude Code → Common workflows → Run parallel sessions with Git worktrees](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees).

##### Pattern D — Conductor (visual multi-agent orchestration, macOS)

[Conductor](https://www.conductor.build/) is a free macOS desktop app that wraps Pattern C with a visual dashboard: it spins up multiple Claude Code (and Codex) agents in parallel, each in its own isolated git worktree, and gives the developer a single pane of glass for status, diffs, merging, and PR creation. It uses the developer's existing Claude API key / Pro / Max subscription for billing — there is no separate Conductor charge — and is built on the same `git worktree` primitive as Pattern C, so dropping out of Conductor at any point leaves a fully working git tree behind.

**What it adds over manual worktrees.**

- Single-window dashboard for 3–8 concurrent agents with at-a-glance progress, prompt, and diff.
- Diff-first review UI: every agent's changes appear side by side; merge or archive per agent.
- Automated worktree lifecycle: spawn creates the worktree and branch; archive prunes it; merge opens a PR.
- Single-workspace mode for collaborative work (one agent implements while another reviews the same files) and multi-workspace mode for genuinely independent features.
- ⌘ N to spawn a new agent on a new worktree from anywhere in the app — the friction floor for trying a parallel run is one keystroke.

**When to adopt.** Mac-only teams that already run 2+ Claude Code sessions on most stories and feel the friction of managing terminals and worktrees by hand. Sweet spot is 3–8 features in flight on the same repo with visual oversight — exactly the band where Pattern C starts to break down on terminal context-switching. Linux and Windows developers stay on Pattern C until Conductor (or an equivalent) ships cross-platform.

**Conflict resolution inside Conductor.**

- **Conductor's diff-first review UI is the early-warning system for conflicts.** Before invoking [`conflict-resolver`](#conflict-resolver) on any agent's branch, scan the diffs across agents in the dashboard — two agents touching the same file is the leading indicator. If the overlap is small or one agent's branch is redundant, **archive** that agent rather than running the resolver: archiving is free, conflict resolution costs analysis tokens. Reserve the resolver for the conflicts that actually need to land.
- **PR-time conflicts.** Conductor's PR opening flow uses its own template (see "Constraints" below). When the PR is opened and main moves before merge, `conflict-resolver` runs in the worktree-backed Claude Code session that produced the PR — exactly as in Pattern C. Conductor itself does not run the resolver; it surfaces the conflict in the dashboard and the developer invokes the agent inside the affected worktree's session.
- **`isolation: worktree` interaction with Conductor.** Conductor already gives every agent its own worktree, so `isolation: worktree` on a subagent spawned inside a Conductor session creates a worktree-inside-a-worktree. This works in current Conductor builds but is unnecessary overhead — **drop `isolation: worktree`** on subagents spawned inside Conductor sessions and let Conductor's own worktree be the isolation boundary.

**Constraints.** macOS only (Apple Silicon and Intel) as of mid-2026 — verify before committing the team. The PR opening flow is Conductor's, not `linear-task-agent`'s, so teams adopting Conductor must make sure the PR title still includes `[ENG-XXX]` and the body still includes `Closes ENG-XXX` (configurable in Conductor's PR template) — otherwise Linear's git integration will not auto-transition. Treat Conductor's PR template as code-reviewed infrastructure, same as `.claude/agents/linear-task-agent.md`.

**What it does not change.** Step 4 (Code Review) is identical — every Conductor PR runs through CodeRabbit, SonarQube, and human review like any other PR. Conductor accelerates *opening* PRs; it does not bypass *merging* gates.

##### Choosing the right pattern

| Situation | Pattern | One-line reason |
|-----------|---------|-----------------|
| Cross-layer feature inside a single story (frontend + backend + tests) | **A — Subagents** | Lowest overhead; the six specialists are already wired. |
| Adversarial code review or competing-hypothesis debugging | **B — Agent Teams** | Peer messaging is what makes the debate work — subagents cannot challenge each other. |
| New module owned end-to-end by a single sub-team, with strong file scoping | **B — Agent Teams** | Teammates self-claim from a shared task list; lead synthesises. |
| Two to four sibling stories in flight in the same cycle | **C — Worktrees** | Sessions stay independent; review and merge on their own schedules. |
| Refactor that the developer wants to be able to throw away cleanly | **C — Worktrees** + `isolation: worktree` on `refactor-specialist` | Failed attempt prunes itself; main branch is never polluted. |
| 3+ stories in flight, macOS team, friction managing terminals | **D — Conductor** | Same primitive as C, with a dashboard — adopt once C starts costing more attention than it saves. |
| Inline single-file edit, "rename this", small intra-file tweak | **None — Cursor inline** | Multi-agent overhead is wasted on intra-file work. |
| Vague AC or fuzzy story | **None — bounce to PM (Step 2.2)** | Multi-agent multiplies ambiguity. Fix the spec first. |

The patterns are stackable: a typical heavy-load day looks like Conductor (D) holding three stories in flight, each story's session running its specialists (A) in parallel, and one of those stories occasionally promoting to an agent team (B) when adversarial review is warranted. The patterns answer different questions — A is *"how do I split one story across roles?"*, B is *"how do I get those roles to talk?"*, C is *"how do I run multiple stories at once?"*, D is *"how do I see all of that on one screen?"*.

##### Operational guardrails (apply to all four patterns)

- **One PR per Linear story, always.** Multi-agent does not change PR granularity. If a multi-agent run produces work for two stories, split the diff into two PRs against two issues. The agent fan-out is internal scaffolding; the audit trail stays one issue → one PR → one merge.
- **`linear-task-agent` is still the only Linear writer.** Specialists, teammates, Conductor agents — none of them call Linear MCP. Single-source the Linear writes so audit logs do not fragment.
- **File-scope every agent before spawn.** Each agent must own a disjoint set of files. The `frontend-engineer` / `backend-engineer` boundary already enforces this for Pattern A; for Patterns B/C/D, spell out the scope in the spawn prompt or branch plan. Two agents touching the same file is the leading cause of wasted multi-agent runs.
- **Conflict resolution is a single-writer operation.** Across all four patterns, [`conflict-resolver`](#conflict-resolver) runs alone on the affected working tree — never spawned alongside an active implementation specialist (Pattern A — drain in-flight specialists first), never alongside an actively-writing teammate on the same branch (Pattern B — the rebase teammate hands off to the resolver), always inside the affected worktree's session (Patterns C and D), and never inside an `isolation: worktree` subagent spawn (the discard-and-respawn flow handles that case). The agent's per-hunk rule analysis assumes a stable working tree; concurrent writers invalidate it.
- **Plan approval for risky teammates (Pattern B).** For migrations, auth changes, and high-blast-radius refactors, spawn the teammate with a plan-approval requirement (`Spawn an architect teammate to refactor X. Require plan approval before any code changes.`). The teammate stays in plan mode until the lead approves; bad architecture is rejected before the code is written.
- **Wire `TaskCompleted` hooks (Pattern B).** Block task completion until the local test suite is green and the linter is clean. Without this, agent teams routinely close tasks while CI would fail at PR open.
- **Token budget per cycle.** A four-teammate Pattern B session can 4–5× a single-session token bill. Track per-cycle token spend in the retro alongside velocity; if multi-agent does not also lift PR throughput, it is a cost line, not a capability.
- **Cap parallelism at 5.** Across all four patterns, three to five agents is the sweet spot industry-wide (Anthropic, Conductor, Osmani 2026). Beyond five, coordination cost dominates parallelism gain — and the developer's ability to actually verify each output collapses.
- **Verification is the bottleneck — staff for it.** With three agents producing PRs in parallel, the human review queue must clear at the same rate or the Cycle stalls in *In Review*. Pair multi-agent adoption with a CodeRabbit-tuned review pipeline and a named secondary reviewer; do not adopt multi-agent and a thin review bench at the same time.

##### Pitfalls

| Pitfall | Mitigation |
|---------|------------|
| **Same-file overwrites.** Two agents edit the same file; the second silently wins. | File-scope at spawn (above). For Pattern B, the shared task list's file-locking helps — but the lead must split tasks at file granularity, not story granularity. |
| **Merge-train rebase explosion.** Five worktrees all reach "ready" together; the first lands clean, but the trailing four each hit progressively worse conflicts as main moves under them. | Land worktrees serially with [`conflict-resolver`](#conflict-resolver) invoked inside each one before its rebase. Keep file scopes disjoint at spawn time so trailing worktrees touch fewer of the just-landed changes. If trailing-worktree conflict load is consistently high, the file-scope discipline is wrong, not the resolver — re-decompose at story granularity. |
| **Unverified completion.** Agent declares "done" while tests are red or AC is unmet. | `TaskCompleted` hook on Pattern B; explicit `code-reviewer` pass on Pattern A; CI gate at Step 4 catches the rest — but pre-PR is much cheaper than post-PR. |
| **Spec amplification.** Vague AC + N agents = N divergent implementations. | Bounce ambiguous stories to PM at Step 2.2. Multi-agent demands tighter specs, not the same specs faster. |
| **Token budget surprises.** A Pattern B retro reveals the cycle spent 5× a normal Claude budget for marginal throughput gain. | Track per-cycle token spend in the cycle retro alongside velocity. Drop back to Pattern A or single-session for the next cycle if the lift is not there. |
| **Lead context fragmentation.** The team lead's context fills with status updates and loses the story plot. | Use the shared task list (Pattern B) and the dashboard (Pattern D) as the working memory; do not rely on the lead's chat scrollback. Clean up the team (`Clean up the team`) at story close so a fresh story starts with a clean lead. |
| **Conductor PR template drift.** Conductor opens PRs but the team's `[ENG-XXX]` / `Closes ENG-XXX` convention is missing. | Conductor's PR template is committed and reviewed like any other config file. Verify in the first week of adoption that auto-transition still fires. |
| **Agent-authored over-assignment.** Tech Lead routes too many stories to autonomous patterns (Linear Agent + Pattern B unattended) and exceeds review bandwidth. | Cap agent-authored PRs in flight per cycle (e.g., ≤ 3, same as the Linear Agent guardrail in the existing risk table). The `agent-authored` label triggers two human reviewers, not one. |

> **Adoption sequence (recommended).** Week 1: pilot Pattern A on cross-layer stories — every team is already configured for it. Week 2: enable Pattern B (set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, wire the three hooks) and use it for one research/review story. Week 3+: layer Pattern C worktrees for in-flight parallelism; consider Pattern D / Conductor only when the team is regularly running 3+ worktrees at once and the terminal-juggling is the visible friction. Skipping straight to Pattern D from single-session is a common first-mover mistake — the dashboard does not teach the discipline that A and B build.

**Coding standards for AI-assisted development:**
- **Always review AI-generated code line by line.** Never merge unread code.
- **AI code must pass the same standards as human code.** No exceptions.
- **Run tests after every AI edit.** AI code has 1.7x more issues per PR than human-written code (CodeRabbit / academic data, 2026).
- **Commit after each AI change.** Creates rollback points in git history.
- **Note AI-generated sections in the PR description.** Transparency for reviewers.
- **Never merge code you don't understand.** If you can't explain it, you can't maintain it.

---

### Step 4: Code Review

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-code-review)

| Attribute | Detail |
|-----------|--------|
| **Input** | Feature branch with passing local tests, linked Linear issue |
| **Tool** | **CodeRabbit** (AI PR review) + **SonarQube** (SAST + coverage gate) + **GitHub/GitLab/Bitbucket** + **Linear MCP** for status |
| **Output** | Merged PR, Linear issue auto-transitioned to **Done** by the git integration, SonarQube clean on main |
| **Human** | Architecture alignment, business-logic validation, final approval |

**Workflow:**

**4.1 — Open the PR with the right title and description.** Before opening, rebase the feature branch onto the latest main; if conflicts arise, invoke [`conflict-resolver`](#conflict-resolver) to work through them — never `--abort` without an explicit reason, since reflexive aborts have lost already-done resolution work in past incidents. Then run the [`pr-description`](./PROMPTS.md#pr-description) prompt. The title **must** include the Linear identifier (`[ENG-247] OAuth2 callback handler`), and the description **must** include the closing keyword `Closes ENG-247`. Linear's git integration uses this to auto-transition the issue to **In Review** on PR open and to **Done** on merge — no manual updates needed in Linear from this point.

**4.2 — CI runs.** The pipeline executes: lint → type check → unit tests → integration tests → SonarQube SAST → coverage gate (≥ 80% on new code). A failure here blocks CodeRabbit and human review until fixed.

**4.3 — CodeRabbit reviews automatically.** CodeRabbit posts inline comments on bugs, security issues, code quality, and test gaps. **Critical and High comments must be resolved**; Medium/Low can be dismissed with a one-line reason. Re-running CodeRabbit happens automatically on every push.

**4.4 — Human review.** A non-author teammate reviews architecture alignment, business-logic correctness, naming, cross-service impacts, and the AI-generated sections that the developer flagged in the PR description. **One human approval is the minimum**; high-blast-radius changes (auth, payments, schema migrations) require two.

**4.5 — Merge.** Required before merge: CI green + CodeRabbit comments resolved + ≥ 1 human approval + Linear issue is the same one referenced in the PR title (sanity check — wrong identifier means an unrelated issue auto-closes). If main has moved during review and the merge button is blocked by conflicts, invoke `conflict-resolver` for the resolution before retrying the merge.

**4.6 — Verify Linear auto-transition.** After merge, the Linear issue should be **Done** within ~30 seconds (Linear's webhook). If not, the developer manually closes it and files an issue against the Linear ↔ git integration.

**Code Review Checklist:** [../templates/code-review-checklist.md](../templates/code-review-checklist.md).

**PR requirements (enforced by template):**
- Title contains the Linear identifier in `[ENG-XXX]` form
- Body contains `Closes ENG-XXX` (single-issue PRs) or `Part of ENG-XXX` (multi-PR stories — manual close)
- Approach summary (3–5 lines)
- Notation of AI-generated sections
- All automated checks passing
- ≥ 1 human approval; 2 for high-blast-radius changes

---

### Step 5: Refactoring

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-refactoring)

| Attribute | Detail |
|-----------|--------|
| **Input** | SonarQube findings, tech-debt backlog (Linear issues labelled `tech-debt`) |
| **Tool** | **Claude Code** (large-scale, multi-file) + **Cursor** Composer (localised) |
| **Output** | Refactored code with maintained test coverage, no behaviour change |
| **Human** | Scope validation, behaviour verification |

**Workflow:**

**5.1 — Identify and ticket targets.** SonarQube flags code smells; Claude Code's [`refactor-candidates`](./PROMPTS.md#refactor-candidates) prompt scans for structural improvements (long methods, god classes, repeated patterns). Each accepted target becomes a Linear issue with the `tech-debt` label — refactoring is tracked work, not a back-pocket activity.

**5.2 — Scope explicitly.** In the Linear issue, write what changes and what stays the same. Refactoring is a behaviour-preserving operation; if behaviour changes, it is a feature, not a refactor — split it.

**5.3 — Execute.** Claude Code for large-scale (extract service, rename pattern across codebase, dependency upgrade with API change). Cursor Composer for localised (extract function, simplify conditional, parameterise).

**5.4 — Verify.** All existing tests must pass; if they do not, the refactor is wrong. **No new functionality** in refactoring PRs. Reviewers reject refactor PRs that introduce features — split the diff.

**Rule:** Refactoring PRs are PRs against `tech-debt` Linear issues, never against feature stories. Separate concerns.

---

### Step 6: Documentation

> Visual: [Step 6 flowchart](./FLOWCHART.md#step-6-documentation)

| Attribute | Detail |
|-----------|--------|
| **Input** | Implementation code, OpenAPI spec, ADRs |
| **Tool** | **Claude Code** (comprehensive docs, runbooks) + **Cursor** (inline docstrings) |
| **Output** | Inline docs, module READMEs, auto-generated API docs, operational runbooks |
| **Human** | Validate accuracy, ensure completeness |

**Workflow:**

**6.1 — Inline docs (continuous).** Cursor generates docstrings as code is written. Review and refine in the same commit as the code — docs added as an afterthought drift.

**6.2 — Module READMEs.** When a module crosses ~500 lines or > 3 public exports, run the [`documentation-generation`](./PROMPTS.md#documentation-generation) prompt in Claude Code. Output covers overview, architecture, key concepts, usage, configuration, testing, limitations.

**6.3 — API docs.** Auto-generated from `/docs/api/openapi.yaml` (the Phase 2 artifact) using Redoc or Swagger UI. **Never maintain hand-written API docs alongside the spec** — they will drift within two sprints.

**6.4 — Operational runbooks.** Use Claude Code with the [`runbook-generation`](./PROMPTS.md#runbook-generation) prompt to draft on-call runbooks from the codebase + observability dashboards. Reviewed by an SRE before publishing.

**Standard:** Every exported function has a docstring. Every module has a README. API docs are generated from the spec — no hand-written API reference.

---

## Daily Developer Workflow

The Linear-driven loop runs every working day:

```
Morning:
       Pull latest from main
  2.1  Claude Code → linear-next-task → next assigned issue
  2.2  Review story AC + ADRs + PRD section deep-link;
       Claude Code transitions issue to In Progress, self-assigns,
       and posts kickoff comment
  2.3  git checkout Linear branchName
  2.4  Pull dependent context (PRD, ADRs, OpenAPI, existing tests)

Design (non-trivial stories only):
  3.0  software-architect → plan → developer approves → linear-task-agent
       posts 3-5 line plan summary as a Linear comment

Coding (in Cursor, Claude Code for complex):
  3.1  Scaffold with Composer (or Claude Code for complex)
  3.2  Code with Supermaven autocomplete
  3.3  Tests alongside code
  3.4  Post progress comments on the Linear issue at checkpoints
  3.5  Self-review: Claude Code with self-review prompt

Review:
  4.1  Open PR with [ENG-XXX] title + Closes ENG-XXX → Linear auto-moves to In Review
  4.2  CI: lint + types + tests + SonarQube + coverage
  4.3  CodeRabbit AI review
  4.4  Human review (≥ 1 approval, 2 for high-blast-radius)
  4.5  Merge to main
  4.6  Linear auto-transitions to Done

End of Day:
       Commit WIP to feature branch if not merged
       Update issue with EOD comment if blocked
```

---

## Phase Handoff

When all MVP stories in the cycle are merged and [Gate 3: Phase Completion](./QUALITY-GATES.md#gate-3-phase-completion) passes, the following artefacts hand off to **Phase 4: Testing & QA**:

| Artefact | Format | Location |
|----------|--------|----------|
| Implemented codebase | Source code | Git repository (main branch) |
| Unit test suite | Test files | `/tests/` or co-located with source |
| Integration test stubs | Test files | `/tests/integration/` |
| API documentation | Auto-generated from OpenAPI | Published docs site or `/docs/api/` |
| Module READMEs | Markdown | Co-located in each module directory |
| SonarQube report | Dashboard + CI artefact | SonarQube server |
| Developer setup guide | Markdown | `/docs/setup.md` or repo README |
| Environment config docs | Markdown | `/docs/configuration.md` |
| Linear cycle report | Linear Cycle view | Linear (`phase:dev` saved view) |

**Handoff Checklist:**
- [ ] All MVP stories merged to main with passing CI
- [ ] Every story closed in Linear with the auto-transition (no manual closes — drift signal)
- [ ] Test coverage ≥ 80% on new code (SonarQube verified)
- [ ] 0 Critical / 0 High SonarQube issues on main
- [ ] API docs published and matching implementation
- [ ] Developer setup guide tested by someone other than the author
- [ ] All environment variables documented
- [ ] Health check endpoints operational
- [ ] Structured logging in place (JSON, correlation IDs)
- [ ] Linear cycle retrospective filed; AI-estimate vs actual variance recorded for next cycle's calibration

---

## Sprint Loop (end-to-end)

```
[Phase 2 handoff] Linear Project = approved PRD + ADRs + OpenAPI + wireframes
   │
[Step 1.1-1.5] Claude Code + Linear MCP → pull backlog, estimate, post comments,
               decompose XXLs into sub-issues (parentId)
   │
   ▼  GATE 1: SPRINT COMMITMENT — Tech Lead opens the Cycle in Linear
   │
[Step 2.1-2.4] linear-next-task → fetch issue → update_issue (In Progress, assignee=me)
               → git checkout <branchName> → kickoff comment → context pull
   │
[Step 3.0]     software-architect → plan → developer approves → plan summary on Linear
               (non-trivial stories only; in-pattern stories skip to 3.1)
   │
[Step 3.1-3.4] Cursor / Claude Code implement + test + checkpoint comments
[Step 3.5] self-review → fix Critical/High before PR
   │
[Step 4.1] Open PR with [ENG-XXX] title + Closes ENG-XXX
              ↳ if rebase conflicts → conflict-resolver
           → Linear auto-transitions to In Review
[Step 4.2] CI: lint + types + tests + SonarQube + coverage
[Step 4.3] CodeRabbit AI review
[Step 4.4] Human review
   │
   ▼  GATE 2: PR MERGE — PR merged
   │
   └─► Linear auto-transitions issue to Done
   │
[Step 5/6 as needed] Refactoring tracked as `tech-debt` issues; docs alongside code
   │
   ▼  Repeat for every issue in the cycle
   │
   ▼  GATE 3: PHASE COMPLETION — coverage ≥ 80%, 0 Critical/High SonarQube,
              all stories Done, retro filed
   │
[Phase 4: Testing & QA]
```

Three explicit gates ensure that **no code reaches main without CI + CodeRabbit + human approval**, and **every merged commit is traceable to a Linear issue** through the PR title and `Closes` keyword.

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **PR-to-issue drift** — developer invents a branch name, Linear's git integration fails to auto-link, the issue stays *In Progress* after merge | Step 2.3 mandates `branchName` from Linear MCP. Linear's git integration is the authority — broken links surface in the **In Progress** view as stale issues; the Tech Lead sweeps it weekly. |
| **Wrong issue closed** — PR title typo (`ENG-274` vs `ENG-247`) auto-closes an unrelated issue | Step 4.5 sanity-check: reviewer confirms the linked issue matches the diff before merging. The Linear `update_issue` audit log + git diff together identify the mistake within hours; reopen and fix forward. |
| **Silent state-change scope creep** — a Claude prompt accidentally moves backlog issues to In Progress | Anthropic admin policy keeps `update_issue` scoped to the dev team only; the audit log on Linear records every state change. Off-board old developers by revoking OAuth in Linear. |
| **Hallucinated APIs / dependencies in AI-generated code** — Claude or Cursor invents a method that does not exist | Run tests after every AI edit (Step 3.3). CI catches imports of non-existent modules. CodeRabbit flags hallucinated APIs in 70%+ of cases (CodeRabbit telemetry). |
| **Linear Agent over-assignment** — Tech Lead routes too many stories to the Linear Agent without human review-bandwidth | Cap agent-authored PRs in flight per cycle (e.g., ≤ 3). The `agent-authored` label triggers two human reviewers, not one. Agent quality is reviewed at cycle close; under-performing patterns roll back to human-only. |
| **Refactor PRs introduce features** — Claude generalises during a refactor and adds capability that nobody asked for | Step 5.4: PRs against `tech-debt` issues that introduce features are rejected at review and split. The reviewer checklist explicitly asks "is any new behaviour added?". |
| **Comment spam on Linear** — every Claude Code session posts a kickoff + checkpoint + EOD comment, drowning real discussion | Comment discipline: one kickoff, checkpoint comments only on substantive change, one EOD only if not merged. Aggregate progress in the PR description, not on the issue. |
| **AI estimate anchoring** — team accepts Claude's estimate without calibration | Step 1.4 mandates a divergence reason captured as a comment. Cycle retro reviews variance. The team's velocity (not Claude's estimates) drives capacity. |
| **Architecture-by-implementer** — `frontend-engineer` or `backend-engineer` invents a design mid-implementation, drifting from the established pattern | Step 3.0 routes non-trivial stories through `software-architect` first; implementation specialists explicitly defer architectural questions to the architect or escalate to a Phase 2 human-architect review. The developer-approval gate on the architect plan blocks scaffolding until the design is sound. |
| **Lost work on `--abort`** — developer (or AI) hits a confusing rebase / merge state and runs `git rebase --abort` or `git merge --abort`, throwing away resolution work already done across multiple hunks | `conflict-resolver` is forbidden from running `--abort` without explicit user instruction; if resolution becomes ambiguous it stops and surfaces the conflict for human decision rather than discarding work silently. Developers should also default to invoking `conflict-resolver` over `--abort` when stuck — the agent's per-hunk report makes "what would I lose?" answerable before any destructive command runs. |

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowchart →](./FLOWCHART.md)
- [Code Review Checklist →](../templates/code-review-checklist.md)
- [Phase 1 Linear MCP setup (Step 0) →](../01-requirement-gathering/PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp)

## External References

- [Linear MCP server docs](https://linear.app/docs/mcp) — server URL, OAuth, supported clients, available tool families
- [Linear MCP setup for Claude Code (Linear blog)](https://linear.app/changelog/2026-04-23-linear-agent-mcp-support) — April 2026 Agent MCP support
- [Linear Agent — Changelog](https://linear.app/changelog/2026-03-24-introducing-linear-agent) — assignable AI teammates, mentions, contributor flow
- [Linear MCP for product management — Changelog](https://linear.app/changelog/2026-02-05-linear-mcp-for-product-management) — initiatives, milestones, project updates
- [Claude × Linear integration page](https://linear.app/integrations/claude)
- [Claude Code MCP setup](https://code.claude.com/docs/en/mcp)
- [Claude Code Agent Teams (experimental, peer messaging)](https://code.claude.com/docs/en/agent-teams) — `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, v2.1.32+, shared task list, `TaskCreated` / `TaskCompleted` / `TeammateIdle` hooks
- [Claude Code subagents](https://code.claude.com/docs/en/sub-agents) — in-process specialist agents, `isolation: worktree` frontmatter for per-agent worktree
- [Claude Code parallel sessions with Git worktrees](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) — manual multi-session pattern
- [Conductor (multi-agent orchestrator, macOS)](https://www.conductor.build/) — visual dashboard, isolated worktrees, free (uses your existing Claude / Codex auth)
- [Conductors to Orchestrators (O'Reilly Radar, 2026)](https://www.oreilly.com/radar/conductors-to-orchestrators-the-future-of-agentic-coding/) — industry framing of the conductor → orchestrator shift
- [The Code Agent Orchestra (Addy Osmani, 2026)](https://addyosmani.com/blog/code-agent-orchestra/) — multi-agent roles, scope boundaries, pitfalls
- [Linear's git integration (auto-link, auto-transition)](https://linear.app/docs/github) — branch-name auto-link, `Closes` keyword
- [CodeRabbit Pro](https://www.coderabbit.ai/) — AI PR review
- [SonarQube Community](https://www.sonarsource.com/products/sonarqube/) — SAST and coverage gate
- [Phase 3 Tools Evaluation](../../docs/tools-evaluation/3.Development_Phase_Tools.md)
