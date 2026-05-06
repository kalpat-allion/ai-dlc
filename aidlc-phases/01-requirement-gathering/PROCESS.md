# Phase 1: Requirement Gathering — Process Definition

## Overview

This document defines the AI-assisted workflow for the Requirement Gathering phase, using the consolidated AI-DLC tool stack.

**Phase Duration:** 1–2 weeks (varies by project size)
**Phase Owner:** Product Manager / Business Analyst
**Tools Used:** Claude, Fireflies.ai, **Linear** (project management spine, integrated with Claude via MCP)

> **Tool Philosophy:** Claude handles ALL AI reasoning in this phase — PRD generation, user stories, acceptance criteria, gap analysis, and cost estimation. **Linear is the single home for both the PRD and the work breakdown.** The PRD lives as a **Linear Document** attached to the Linear Project; epics become Milestones; stories become Issues. Claude reads from and writes to Linear directly through the official Linear connector (Model Context Protocol), so the PRD, the backlog, the gap analysis, and the stakeholder sign-off all live on one continuous, traceable thread inside Linear. No separate documentation tool (Notion, Confluence, repo `/docs/`) is needed for the PRD itself.

---

## Process Steps

### Step 0: One-Time Setup — Connect Claude to Linear via MCP

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Linear workspace admin access, Claude account (Pro / Max / Team / Enterprise) |
| **Tool** | **Linear MCP server** (`https://mcp.linear.app/mcp`) — official, hosted by Linear, OAuth 2.1 |
| **Output** | Claude (web/desktop) and Claude Code can read & write Linear issues, projects, comments, initiatives |
| **Human** | Workspace admin enables the connector; each user authorises once via OAuth |

The Linear MCP server is **the** integration mechanism — do not build a custom Linear API wrapper. It is centrally hosted, OAuth-authenticated, and supported across every Claude surface. Set this up once per workspace, then once per user.

#### Path A — claude.ai (web) and Claude Desktop

1. **Workspace admin (Linear):** confirm the workspace is on a plan that exposes the MCP endpoint and that no IP allow-list blocks `mcp.linear.app`. No additional install is required on Linear's side — the server is hosted by Linear.
2. **Workspace admin (Anthropic, Team/Enterprise only):** in the Anthropic admin console, **enable Linear** under organisation Connectors. Set tool permissions — for Phase 1 we recommend **read + create issues + create comments**, with **update / state-change disabled** until the team is confident (see [Risks & Guardrails](#risks--guardrails)).
3. **Each user (Claude.ai):** open **Settings → Connectors** (or `/` in chat → "Manage connectors"). Find **Linear** in the directory and click **Connect**.
4. **OAuth flow:** Linear opens in a new tab. Sign in. **Review the requested scopes** — at minimum `read`, `write:issues`, `write:comments`. Approve. Linear redirects back to Claude; the connector shows as Connected.
5. **Per-conversation toggle:** open a chat, click **+** (or `/`) → **Connectors**, toggle **Linear** on for that conversation. (Off by default — explicit opt-in per conversation prevents accidental writes.)
6. **Smoke test:** prompt Claude with `List my Linear teams.` — Claude should call the Linear MCP `list_teams` tool and return your teams. Then `Create a draft issue in team <X> titled "MCP smoke test", label "ai-generated".` Confirm it appears in Linear's Triage.

#### Path B — Claude Code (developer / CLI use)

Use Claude Code when generating stories or pushing them to Linear from inside the repo (e.g., during PRD authoring in markdown).

1. **Add the server** (run from repo root for project scope, or anywhere for user scope):
   ```bash
   claude mcp add --transport http --scope user linear https://mcp.linear.app/mcp
   ```
   Use `--scope project` instead of `--scope user` if you want the server committed to the repo's `.mcp.json` so the team shares it.
2. **Authenticate:** open Claude Code (`claude`). Run `/mcp`, select **linear**, approve OAuth in the browser that opens.
3. **Verify:** `claude mcp list` should show `linear: connected`. Inside a session prompt: `Via Linear MCP, fetch issue ALN-123 and summarise it.`
4. **Revoke when done** (e.g., off-boarding): `claude mcp remove linear`, and revoke the OAuth grant in Linear under **Settings → Security & access → OAuth applications**.

#### Verification checklist

- [ ] Linear connector shows **Connected** in Claude settings
- [ ] Smoke-test issue created in Triage with the `ai-generated` label
- [ ] Claude can list teams, projects, and initiatives the user has access to
- [ ] Audit logging is on in Linear (Workspace settings → Security → Audit log)
- [ ] Anthropic admin tool-permission policy reviewed (Team/Enterprise)

> **Permission inheritance:** Claude inherits the connecting user's Linear permissions. A user can only read/write what they themselves can read/write — there is no privilege escalation.

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
4. **Pull related Linear context** (optional but recommended for ongoing products): in a Claude conversation with the **Linear connector enabled**, run the [`linear-context-pull`](./PROMPTS.md#linear-context-pull) prompt to fetch relevant initiatives, active projects, and prior issues. This grounds the PRD in real backlog state instead of starting from a blank page.
5. Feed the structured interview findings **plus** the Linear context bundle into Claude for PRD generation (Step 2).

**For formal user research (if needed):** Use Claude to structure and synthesise interview transcripts — paste transcripts into Claude with the interview structuring prompt from [PROMPTS.md](./PROMPTS.md#interview-summary-structuring). This replaces dedicated research tools for most projects.

> **No writes to Linear in this step.** Stakeholder capture is read-only against Linear. Issue creation begins after PRD approval (Step 3).

---

### Step 2: PRD Generation & Publish to Linear

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-prd-generation--publish-to-linear)

| Attribute | Detail |
|-----------|--------|
| **Input** | Interview summaries, project brief, competitive analysis, technical constraints, **Linear context bundle from Step 1** |
| **Tool** | **Claude** with the **Linear connector enabled** (read in 2.1–2.3, write in 2.4) |
| **Output** | A Linear **Project** (Planned state) with the PRD published as an attached **Linear Document** — the canonical PRD location |
| **Human** | Provide complete context, review and refine AI output, run stakeholder review in Linear, mark Document approved |

**Workflow:**

**2.1 — Gather context.** Compile all inputs — interview summaries, business objectives, competitive data, technical constraints, and the Linear context bundle from Step 1.

**2.2 — Draft in Claude.** Use Claude with the [PRD generation prompt](./PROMPTS.md#prd-generation). Feed ALL context in a single prompt (leverage the 200K+ token context window). Where the Linear bundle includes existing initiatives or related issues, ask Claude to **cite the Linear IDs** in the relevant PRD sections so traceability is established from day one. The draft stays in chat (Markdown) — no Linear write yet.

**2.3 — Self-review with Claude.** Paste the draft back into Claude with the [gap analysis prompt](./PROMPTS.md#gap-analysis) to get a "second opinion" on completeness. Refine. Then run a PM review against the PRD completeness checklist (see [QUALITY-GATES.md](./QUALITY-GATES.md#gate-1-prd-completeness)) and remove any hallucinated requirements.

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
| **Tool** | **Claude** with the **Linear connector enabled** (read + create) |
| **Output** | **Milestones** (epics) and **Issues** (stories in Triage state) added to the existing Linear Project, fully labelled and deep-linked to PRD sections in the Document |
| **Human** | Validate completeness, approve milestone scaffold, accept stories one-by-one out of the AI Inbox view |

This step is a three-stage write to Linear, with a human approval between each stage. The Project already exists — Step 3 only adds Milestones and Issues to it. Nothing reaches an Active state automatically.

**Stage 3a — Decompose into epics**
1. Use Claude with the [epic decomposition prompt](./PROMPTS.md#epic-decomposition). Reference the PRD by Linear Document URL — Claude can read the Document directly via MCP. Output stays in chat — no Linear write yet.

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
| **Tool** | **Claude** with the **Linear connector enabled** (read + comment) |
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
[Step 2.1-2.3] Claude drafts PRD in chat → self-review → PM review (Markdown only)
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
| **Duplicate issues** — Claude is re-run and pushes the same stories twice | The `stories-to-linear-push` prompt **must** call `linear-context-pull` first and report any title or AC matches before creating. PM rejects duplicates in the AI Inbox. |
| **Silent state changes / scope creep** — Claude moves issues to In Progress or edits AC on existing tickets | At admin level, **disable update / state-change scopes** in the Anthropic Connectors policy. Claude can create issues and post comments, not mutate existing ones. Promote scopes only when team maturity justifies it. |
| **Hallucinated stories** — fabricated personas or features that never appeared in the PRD | `needs-human-review` gate label + the `**PRD section: §X.Y**` citation rule. Stories without a valid citation are deleted at Gate 3. The AI Inbox view enforces a daily sweep. |
| **Permission escalation surprises** | Claude inherits the connecting user's Linear permissions only — no escalation. Off-board users by revoking the OAuth grant in Linear (Settings → Security & access → OAuth applications) and `claude mcp remove linear` on their machine. |

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowcharts →](./FLOWCHART.md) (six per-step diagrams)
- [PRD Template →](../templates/prd-template.md)
- [User Story Template →](../templates/user-story-template.md)

## External References

- [Linear MCP server docs](https://linear.app/docs/mcp) — server URL, OAuth, available tools
- [Claude × Linear integration page](https://linear.app/integrations/claude)
- [Anthropic — Use connectors to extend Claude's capabilities](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)
- [Claude Code MCP setup](https://code.claude.com/docs/en/mcp)
