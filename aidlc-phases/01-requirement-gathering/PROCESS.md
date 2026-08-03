# Phase 1: Requirement Gathering — Process Definition

## Overview

This document defines the AI-assisted workflow for the Requirement Gathering phase, using the consolidated AI-DLC tool stack.

**Phase Duration:** 1–2 weeks (varies by project size)
**Phase Owner:** Product Manager / Business Analyst
**Tools Used:** Claude Code, Fireflies.ai, **Linear** (project management spine, integrated with Claude Code via MCP)

> **Tool Philosophy:** Claude Code handles ALL AI reasoning in this phase — PRD generation, user stories, acceptance criteria, gap analysis, and cost estimation. **Linear is the single home for both the PRD and the work breakdown.** The PRD lives as a **Linear Document** attached to the Linear Project; epics become Milestones; stories become Issues. Claude Code reads from and writes to Linear directly through the official Linear MCP server (Model Context Protocol), so the PRD, the backlog, the gap analysis, and the stakeholder sign-off all live on one continuous, traceable thread inside Linear. No separate documentation tool (Notion, Confluence, repo `/docs/`) is needed for the PRD itself.

---

## Process Steps

### Step 0: One-Time Setup — Connect Claude to Linear via MCP

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Linear workspace admin access, Claude subscription that funds Claude Code (Pro / Max / Team / Enterprise), Claude Code installed |
| **Tool** | **Linear MCP server** (`https://mcp.linear.app/mcp`) — official, hosted by Linear, OAuth 2.1, wired into Claude Code at **project scope** |
| **Output** | Claude Code can read & write Linear issues, projects, comments, and initiatives from inside the repo |
| **Human** | Workspace admin confirms the MCP endpoint is reachable; each user authorises once per project via OAuth (`/mcp`) |

The Linear MCP server is **the** integration mechanism — do not build a custom Linear API wrapper. It is centrally hosted, OAuth-authenticated, and driven entirely from **Claude Code**. Register it once per repo (the registration is committed to `.mcp.json`), then each user authenticates once per project.

#### Set up the project-scoped Linear MCP server in Claude Code

All Claude interaction in the AI-DLC runs through **Claude Code**, and the Linear MCP server is wired **per project** so each repo carries its own committed MCP config. This is what lets one person work on several projects concurrently — each repo authenticates independently, and can even sign in with a **different Linear account per project**.

1. **Workspace admin (Linear):** confirm the workspace is on a plan that exposes the MCP endpoint and that no IP allow-list blocks `mcp.linear.app`. No additional install is required on Linear's side — the server is hosted by Linear.
2. **Add the server at project scope** (run from the repo root). This writes the registration to the repo's committed `.mcp.json`:
   ```bash
   claude mcp add --transport http --scope project linear https://mcp.linear.app/mcp
   ```
   Because `.mcp.json` is committed, the whole team shares one per-project MCP config — nobody re-derives the setup. If a repo needs Linear under a **different account** than another project you already have open, give it a **distinct server name** so the two registrations (and their OAuth tokens) never collide:
   ```bash
   claude mcp add --transport http --scope project linear-acmeco https://mcp.linear.app/mcp
   ```
3. **Authenticate once per project:** open Claude Code (`claude`) in the repo. Run `/mcp`, select **linear** (or the distinct name from step 2), and approve OAuth in the browser. **Review the requested scopes** — at minimum `read`, `write:issues`, `write:comments`. The token is stored against this project's server registration, so authenticating one project never disturbs another — concurrent projects each keep their own identity.
4. **Smoke test:** in a session prompt `List my Linear teams.` — Claude Code should call the Linear MCP `list_teams` tool and return your teams. Then `Create a draft issue in team <X> titled "MCP smoke test", label "ai-generated".` Confirm it appears in Linear's Triage.
5. **Restrain write scope until the team is confident.** For Phase 1, keep Claude Code to **read + create issues + create comments**, with **update / state-change actions withheld** — enforced through Claude Code's tool-permission settings and the confirm-before-write design of this phase's Linear-writing prompts (see [Risks & Guardrails](#risks--guardrails)). Promote to update / state-change scopes only when the team is confident.
6. **Revoke when done** (e.g., off-boarding): `claude mcp remove linear`, and revoke the OAuth grant in Linear under **Settings → Security & access → OAuth applications**.

#### Install the Phase 1 requirement skills

Phase 1 ships three **skills** as templates — one per Linear write step. Copy them into the consuming repo, fill the placeholders, and commit; they are project artefacts, not personal config, and they transfer with the repo at Phase 7. Install them after the MCP server is connected, because all three call it on their first step.

```bash
# From the consuming repo root
mkdir -p .claude/skills
cp -r <ai-dlc>/aidlc-phases/01-requirement-gathering/skill-prompts/*/ .claude/skills/
```

Skills are **folders**, not files — `.claude/skills/<skill-name>/SKILL.md`, where the folder name must equal the `name:` field. A mismatch means the skill silently never loads, so copy the directories with `cp -r` rather than globbing the Markdown.

| Skill | Owns | Gate |
|-------|------|------|
| [`publish-prd-to-linear`](./skill-prompts/publish-prd-to-linear/SKILL.md) | Steps 2.2–2.4 — drafts the PRD, self-reviews it, creates the Project (`Planned`) and publishes the Document, returns the section-anchor map | Gate 1 |
| [`scaffold-linear-milestones`](./skill-prompts/scaffold-linear-milestones/SKILL.md) | Stages 3a–3b — decomposes the approved Document into ordered epics and creates one Milestone each | Gate 2 |
| [`push-linear-stories`](./skill-prompts/push-linear-stories/SKILL.md) | Stage 3c — drafts stories with AC, runs the duplicate pre-flight, creates Triage issues with verified PRD deep-links | Gate 2 → Gate 3 |

Fill `{{LINEAR_INITIATIVE}}` and `{{LINEAR_TEAM}}` before committing. Verify by confirming each appears in the available-skills list under its slug — **`/agents` does not list skills.** Then smoke-test the stop: invoke `publish-prd-to-linear` with a two-line brief and confirm it refuses to write anything until you reply `go`. Full instantiation and routing checklist in [`skill-prompts/README.md`](./skill-prompts/README.md).

> **These three are the outer-loop Linear writers.** They create Documents, Projects, Milestones and Triage issues. The Phase 3 dev-loop writer (`linear-task-agent`) deliberately does none of this and refuses if asked — the split is by design, not by omission.

#### Verification checklist

- [ ] `claude mcp list` shows `linear: connected` in the repo (registration committed to `.mcp.json`)
- [ ] Smoke-test issue created in Triage with the `ai-generated` label
- [ ] Claude Code can list teams, projects, and initiatives the user has access to
- [ ] Audit logging is on in Linear (Workspace settings → Security → Audit log)
- [ ] Claude Code tool-permission settings keep Linear to read + create until the team matures
- [ ] The three Phase 1 skills are committed under `.claude/skills/`, every placeholder filled, each listed in the available-skills list, and one confirm-before-write stop smoke-tested

> **Permission inheritance:** Claude Code inherits the connecting user's Linear permissions. A user can only read/write what they themselves can read/write — there is no privilege escalation.

---

### Step 1: Stakeholder Interview Capture

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-stakeholder-interviews)

| Attribute | Detail |
|-----------|--------|
| **Input** | Scheduled stakeholder meetings, interview guide |
| **Tool** | **Fireflies.ai** — records, transcribes, generates summaries with action items |
| **Output** | Transcripts, AI summaries, extracted requirements input |
| **Human** | Prepare questions, conduct interview, validate summary within 24h |

**Workflow:**
1. Prepare an interview guide with key questions. Share the agenda with stakeholders.
2. Fireflies.ai auto-joins and records. Focus on the conversation, not note-taking.
3. After the meeting, review the AI summary. Correct errors and extract requirements-relevant statements.
4. **Pull related Linear context** (optional but recommended for ongoing products): in Claude Code with the **Linear MCP server connected**, run the [`linear-context-pull`](./PROMPTS.md#linear-context-pull) prompt to fetch relevant initiatives, active projects, and prior issues. This grounds the PRD in real backlog state instead of starting from a blank page.
5. Feed the structured interview findings **plus** the Linear context bundle into Claude Code for PRD generation (Step 2).

**For formal user research (if needed):** Use Claude Code to structure and synthesise interview transcripts — paste transcripts into Claude Code with the interview structuring prompt from [PROMPTS.md](./PROMPTS.md#interview-summary-structuring). This replaces dedicated research tools for most projects.

> **No writes to Linear in this step.** Stakeholder capture is read-only against Linear. Issue creation begins after PRD approval (Step 3).

---

### Step 2: PRD Generation & Publish to Linear

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-prd-generation--publish-to-linear)

| Attribute | Detail |
|-----------|--------|
| **Input** | Interview summaries, project brief, competitive analysis, technical constraints, **Linear context bundle from Step 1** |
| **Tool** | **Claude Code** with the **Linear MCP server connected** (read in 2.1–2.3, write in 2.4) |
| **Output** | A Linear **Project** (Planned state) with the PRD published as an attached **Linear Document** — the canonical PRD location |
| **Human** | Provide complete context, review and refine AI output, run stakeholder review in Linear, mark Document approved |

**Workflow:**

**2.1 — Gather context.** Compile all inputs — interview summaries, business objectives, competitive data, technical constraints, and the Linear context bundle from Step 1.

> **Steps 2.2 to 2.4 are one procedure, and the [`publish-prd-to-linear`](./skill-prompts/publish-prd-to-linear/SKILL.md) skill runs them as one** — draft, self-review, resolve, publish, return the anchor map. The three sub-steps below remain the definition of what it does and the paste-prompt path for teams that have not installed it.

**2.2 — Draft in Claude Code.** Use Claude Code with the [PRD generation prompt](./PROMPTS.md#prd-generation). Feed ALL context in a single prompt (leverage the 200K+ token context window). Where the Linear bundle includes existing initiatives or related issues, ask Claude Code to **cite the Linear IDs** in the relevant PRD sections so traceability is established from day one. The draft stays local (Markdown) — no Linear write yet.

**2.3 — Self-review with Claude Code.** Run the draft back through Claude Code with the [gap analysis prompt](./PROMPTS.md#gap-analysis) to get a "second opinion" on completeness. Refine. Then run a PM review against the PRD completeness checklist (see [QUALITY-GATES.md](./QUALITY-GATES.md#gate-1-prd-completeness)) and remove any hallucinated requirements.

**2.4 — Publish to Linear.** Run the [`prd-to-linear-document`](./PROMPTS.md#prd-to-linear-document) prompt. Claude calls the Linear MCP to:
   - Create the **Linear Project** under the appropriate Initiative, in **Planned** state (no Milestones, no Issues yet).
   - Create a **Linear Document** attached to that Project containing the full PRD in Markdown, with stable heading anchors so issues can deep-link to specific sections (`§3.2`, `§5.1`, etc.).
   - Apply baseline labels to the Project: `phase:requirements`, `ai-generated`.
   - Return the Project URL and Document URL.

**2.5 — Stakeholder review in Linear.** Share the Linear Document URL with stakeholders. They comment **directly on the Document** (Linear supports inline comments and threaded discussion on Documents). Non-engineering reviewers without Linear seats can be added as **Guests** scoped to the Project — they get comment-only access without consuming a paid seat. Iterate the Document via Claude (Document edit prompts) or by hand until consensus.

**2.6 — Approve.** When sign-off is obtained, the PM:
   - Updates the Document status / heading to **Approved v1.0** (or sets a `status:approved` label on the Document — pick one workspace convention).
   - Moves the Linear Project from **Planned** to **Backlog** (or **Started** when Step 3 begins).
   - Records sign-offs (PM, Tech Lead, Sponsor) as a checked list at the bottom of the Document.

> **Gate 1 (PRD approved in Linear) must pass before Milestones or Issues are created in Step 3.** The Linear Document is the contract; everything in Step 3 cites a section of it.

---

### Step 3: User Story Creation — Milestones & Issues in Linear

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-user-stories--milestones--issues)

| Attribute | Detail |
|-----------|--------|
| **Input** | Approved PRD (Linear Document URL from Step 2.4), persona definitions, the Linear Project (already exists in Planned/Backlog state) |
| **Tool** | **Claude Code** with the **Linear MCP server connected** (read + create) |
| **Output** | **Milestones** (epics) and **Issues** (stories in Triage state) added to the existing Linear Project, fully labelled and deep-linked to PRD sections in the Document |
| **Human** | Validate completeness, approve milestone scaffold, accept stories one-by-one out of the AI Inbox view |

This step is a three-stage write to Linear, with a human approval between each stage. The Project already exists — Step 3 only adds Milestones and Issues to it. Nothing reaches an Active state automatically.

> **Stages 3a and 3b are one procedure** — the [`scaffold-linear-milestones`](./skill-prompts/scaffold-linear-milestones/SKILL.md) skill decomposes the approved Document and creates the Milestones, stopping at Gate 2. **Stage 3c is a second procedure**, run by [`push-linear-stories`](./skill-prompts/push-linear-stories/SKILL.md) only after that gate passes. The cut between them is the human approval, not a line count.

**Stage 3a — Decompose into epics**
1. Use Claude Code with the [epic decomposition prompt](./PROMPTS.md#epic-decomposition). Reference the PRD by Linear Document URL — Claude Code can read the Document directly via MCP. Output stays local — no Linear write yet.

**Stage 3b — Add Milestones to the existing Project (with human approval)**
1. Run the [`prd-to-linear-scaffold`](./PROMPTS.md#prd-to-linear-scaffold) prompt against the existing Linear Project. Claude calls the Linear MCP to:
   - Create one **Milestone** per epic identified in Stage 3a, in PRD order.
   - Set milestone target dates if the PRD timeline supplies them.
2. **Gate 2 — Human approval of the milestone scaffold.** PM reviews the Milestones in Linear and confirms names, ordering, and date alignment with the PRD timeline before any issues are pushed. If wrong, delete the Milestones and re-run.

**Stage 3c — Push stories with acceptance criteria**
1. Use Claude with the story generation prompt to draft stories per epic. Each story follows: "As a [persona], I want [goal], so that [benefit]."
2. For each story, use Claude with the acceptance criteria prompt. Ensure Given/When/Then format with happy path + error + edge cases.
3. Run the [`stories-to-linear-push`](./PROMPTS.md#stories-to-linear-push) prompt. Claude calls the Linear MCP to create issues with:
   - **State:** `Triage` (or `Backlog`) — never `In Progress` or beyond
   - **Parent / Milestone:** the milestone for the corresponding epic
   - **Labels:** `phase:requirements`, `ai-generated`, `needs-human-review`, plus `nfr` where applicable
   - **Description:** the user story + Given/When/Then AC + a `**PRD section:** [§X.Y](<linear-document-url>#section-anchor)` traceability **deep-link** that opens the relevant heading in the Linear Document
4. **Gate 3 — Per-story human acceptance.** PM works through the **AI Inbox** Linear view (filter: `ai-generated AND needs-human-review`). For each story:
   - Click the PRD deep-link to verify the cited section exists and matches.
   - Verify persona, goal, AC are correct.
   - Edit if needed (AC tightening, scope correction).
   - Remove the `needs-human-review` label and move to `Backlog` to accept. Stories without a valid PRD deep-link are deleted.

**Important:** Do NOT push AI-hallucinated scope. If Claude generates features not in the PRD Document, remove them at Stage 3c step 3 before the MCP call, or delete in Linear during Gate 3. Every story in the final backlog must deep-link to an approved PRD section.

---

### Step 4: Requirements Gap Analysis (Linear-diff)

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-gap-analysis-linear-diff)

| Attribute | Detail |
|-----------|--------|
| **Input** | Complete PRD, the Linear project + issue set produced in Step 3 |
| **Tool** | **Claude Code** with the **Linear MCP server connected** (read + comment) |
| **Output** | Gap analysis report posted as a comment on the Linear Project, plus prioritised findings list |
| **Human** | Validate findings, decide which gaps to address, update PRD, trigger another Step 3 cycle for accepted gaps |

**Workflow:**
1. **Run gap analysis:** Reference the Linear PRD Document URL in the gap analysis prompt from [PROMPTS.md](./PROMPTS.md#gap-analysis). Claude reads the Document via MCP and checks for missing NFRs, unstated assumptions, contradictions, dependency gaps, and security omissions.
2. **Run Linear-diff sweep:** Run the [`linear-gap-sweep`](./PROMPTS.md#linear-gap-sweep) prompt. Claude reads both the PRD Document and the current Linear Project state (Milestones + Issues) via MCP, compares them, and posts **a single consolidated comment** on the Linear Project listing gaps with severity. **Claude does not auto-create issues for gaps** — comments only.
3. **Run market check (optional):** Ask Claude to identify what competitors typically offer in this space and what industry standards apply. Claude's training data covers most competitive landscapes; for very recent data, supplement with manual web research.
4. **Prioritise findings:** Consolidate the gap-analysis output and Linear-diff comment into a single list: Critical / High / Medium / Low.
5. **Resolve gaps:** Edit the Linear PRD Document to address Critical/High gaps (bump the version note at the top), then re-run Step 3 (Stage 3c) for the accepted gap stories with label `gap-analysis` added. Document Medium/Low as future considerations directly inside the PRD Document under an "Out-of-scope / future" section.

**Escalation:** Any Critical gap that changes project scope must be approved by the project sponsor.

---

### Step 5: Project Scalability & Cost Assessment

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-scalability--cost)

| Attribute | Detail |
|-----------|--------|
| **Input** | PRD, expected user volumes, target markets |
| **Tool** | **Claude** + free cloud provider pricing calculators (AWS/Azure/GCP) |
| **Output** | Scalability requirements, rough cost estimate |
| **Human** | Validate assumptions, confirm budget alignment |

**Workflow:**
1. Use Claude to extract scalability factors from the PRD and estimate infrastructure cost ranges.
2. Validate against free cloud pricing calculators (no paid tool needed).
3. Add scalability and cost requirements to the PRD as non-functional requirements.

---

## Phase Handoff

All Phase 1 artifacts live in **Linear** — one tool, one URL per artifact, one audit log.

| Artifact | Format | Location |
|----------|--------|----------|
| PRD v1 (approved) | Linear Document (Markdown, with stable section anchors) | **Linear Project → Documents** |
| User Story Backlog | Linear Project + Milestones + Issues | **Linear Project** |
| Gap Analysis Report | Comment on the Linear Project | **Linear Project → Activity** |
| Stakeholder sign-offs | Checked list at the bottom of the PRD Document | **Linear PRD Document** |
| Interview Summaries | Fireflies exports | Shared drive (the only artifact outside Linear) |

**Handoff Checklist:**
- [ ] PRD Document marked **Approved v1.0** with PM, Tech Lead, Sponsor sign-offs recorded inside
- [ ] Linear Project moved to **Backlog** (or **Started** when Step 3 has begun)
- [ ] PRD passes all Gate 1 quality criteria
- [ ] All user stories have acceptance criteria (≥ 3 per story)
- [ ] Every Linear issue carries the `phase:requirements` label and a clickable `**PRD section:** [§X.Y](document-url#anchor)` deep-link in the description
- [ ] **AI Inbox is empty** — every `ai-generated AND needs-human-review` story has been accepted or deleted
- [ ] PRD epic count = Linear Milestone count (parity check)
- [ ] Gap analysis completed; Critical/High gaps resolved (or new stories pushed and accepted)
- [ ] Stakeholder sign-off obtained inside the PRD Document
- [ ] Artifacts traceable from a single Linear Project URL

---

## Linear Workspace Setup

The PM should configure these once per workspace before running Phase 1 on a new project. Keep the taxonomy small — resist adding labels until a real workflow forces them.

**Hierarchy (use Linear's built-in primitives, do not invent new ones):**

| Linear primitive | What it represents in AI-DLC | Created by |
|------------------|------------------------------|------------|
| **Initiative** | A product line or major program (e.g., "TimeSync v2") | Human, once |
| **Project** | One PRD's worth of scope (e.g., "TimeSync v2 — Scheduling Engine") | Claude via MCP after Gate 1 |
| **Milestone** | A PRD epic (release slice within a Project) | Claude via MCP after Gate 2 |
| **Issue** | One user story with AC in the description | Claude via MCP after Gate 2, accepted at Gate 3 |

**Recommended labels:**

| Label | Meaning |
|-------|---------|
| `phase:requirements`, `phase:design`, `phase:dev`, `phase:qa`, `phase:security`, `phase:cicd`, `phase:delivery` | Which AI-DLC phase the issue originated in |
| `ai-generated` | Authored by Claude via MCP (always paired with `needs-human-review` at creation) |
| `needs-human-review` | Gate label — must be removed by a human before the issue is considered accepted |
| `gap-analysis` | Story created from a Step 4 gap finding |
| `source:fireflies` | Originated from a stakeholder interview transcript |
| `nfr` | Non-functional requirement |

**Saved views the PM should configure:**

1. **AI Inbox** — `label = ai-generated AND label = needs-human-review` (the daily review queue)
2. **Phase 1 Active** — `label = phase:requirements AND state != Done`
3. **Gap Backlog** — `label = gap-analysis AND state != Done`

---

## PRD-to-Linear Loop (end-to-end)

```
[Step 1] Stakeholder interviews (Fireflies) + Linear context pull (read-only)
   │
[Step 2.1-2.3] Claude Code drafts PRD locally → self-review → PM review (Markdown only)
   │
[Step 2.4] prd-to-linear-document → Claude creates Linear Project (Planned)
           and publishes PRD as a Linear Document with section anchors
   │
[Step 2.5] Stakeholder review HAPPENS IN LINEAR (Document comments).
           Guests added for non-engineering reviewers. Iterate the Document.
   │
   ▼  GATE 1: PM marks Document Approved v1.0; Project moves out of Planned
   │
[Step 3a] Claude decomposes PRD into epics (chat only)
   │
[Step 3b] prd-to-linear-scaffold → Claude adds Milestones to the existing Project
   │
   ▼  GATE 2: Milestone scaffold approved by PM in Linear
   │
[Step 3c] stories-to-linear-push → Claude creates Triage issues with AC and
           clickable PRD-section deep-links into the Linear Document
   │
   ▼  GATE 3: PM clears AI Inbox (per-story acceptance)
   │
[Step 4] linear-gap-sweep → Claude reads the PRD Document + Project state,
           posts a single gap comment on the Linear Project (no auto-issues)
   │
   └─► If gaps accepted: edit the PRD Document → re-enter Step 3c with `gap-analysis`
   │
[Step 5] Scalability/cost NFRs added to the PRD Document → optional `nfr` issues
   │
   ▼
[Phase 2: System Design]
```

Three explicit human gates ensure that **no Claude-authored Linear item reaches an Active state without sign-off**, and every issue traces back via a clickable deep-link to an approved section of the PRD Document.

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **Duplicate issues** — Claude is re-run and pushes the same stories twice | The `stories-to-linear-push` prompt **must** call `linear-context-pull` first and report any title or AC matches before creating. The [`push-linear-stories`](./skill-prompts/push-linear-stories/SKILL.md) skill has that read-only pre-flight inlined as a step it cannot skip, which is what makes a re-run after a partial push safe. PM rejects duplicates in the AI Inbox. |
| **Silent state changes / scope creep** — Claude moves issues to In Progress or edits AC on existing tickets | **Withhold update / state-change actions** in Claude Code's tool-permission settings for the Linear MCP server. Claude Code can create issues and post comments, not mutate existing ones. Promote scopes only when team maturity justifies it. |
| **Hallucinated stories** — fabricated personas or features that never appeared in the PRD | `needs-human-review` gate label + the `**PRD section: §X.Y**` citation rule. Stories without a valid citation are deleted at Gate 3. The AI Inbox view enforces a daily sweep. |
| **Permission escalation surprises** | Claude inherits the connecting user's Linear permissions only — no escalation. Off-board users by revoking the OAuth grant in Linear (Settings → Security & access → OAuth applications) and `claude mcp remove linear` on their machine. |

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Skill Templates →](./skill-prompts/) (three Claude Code skills — one per Linear write step)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowcharts →](./FLOWCHART.md) (six per-step diagrams)

## External References

- [Linear MCP server docs](https://linear.app/docs/mcp) — server URL, OAuth, available tools
- [Claude × Linear integration page](https://linear.app/integrations/claude)
- [Claude Code MCP setup](https://code.claude.com/docs/en/mcp) — `claude mcp add`, project vs user scope, `.mcp.json`, `/mcp` authentication
