# Phase 1: Requirement Gathering — Prompt Templates

> **All prompts in this file are designed for Claude.** Copy the prompt, replace `[PLACEHOLDERS]` with your project content, and run. Each prompt produces a structured output that requires human review.
>
> Prompts marked **Linear-aware** require the **Linear connector to be enabled** in the conversation (claude.ai web/desktop) or the Linear MCP server to be connected (Claude Code). See [PROCESS.md → Step 0](./PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp) for setup.

---

## PRD Generation

```
You are a senior product manager creating a Product Requirements Document.

## Project Context
- **Product Name:** [Product name]
- **Problem Statement:** [What problem does this solve?]
- **Target Users:** [Who are the primary users?]
- **Business Objectives:** [What are the business goals?]
- **Technical Constraints:** [Existing systems, tech stack, compliance]
- **Competitive Landscape:** [Key competitors and their offerings]
- **Timeline:** [Target delivery timeline]

## Stakeholder Interview Findings
[Paste interview summaries here]

## Additional Context
[Paste competitive analysis, user research, or technical documentation]

Generate a PRD with these sections:
1. **Executive Summary** — One paragraph
2. **Problem Statement** — Problem and affected users
3. **Goals & Success Metrics** — SMART goals with measurable KPIs (minimum 3)
4. **Target Users & Personas** — Full personas with demographics, goals, pain points, usage context (minimum 2)
5. **Functional Requirements** — Grouped by feature area, each with description, user stories, and priority (Must Have / Should Have / Nice to Have)
6. **Non-Functional Requirements** — Performance, security, scalability, accessibility, compliance
7. **User Flows** — Key user journeys step by step
8. **Out of Scope** — What this version does NOT include
9. **Assumptions & Dependencies**
10. **Risks & Mitigations** (minimum 3)
11. **Timeline & Milestones**

Be specific. Use the actual context provided. Make every requirement testable.

### Example (trimmed)
**Input:** Product Name: TimeSync · Problem: distributed teams double-book meetings across timezones · Users: remote engineering managers · Objective: cut scheduling rework by 40%.

**Output excerpt:**
> **3. Goals & Success Metrics**
> - G1: Reduce double-booked meetings by 40% within 90 days of launch (baseline: 12% of meetings).
> - G2: 80% of pilot managers schedule a multi-timezone meeting in <2 minutes (baseline: 6 min).
> - G3: NPS ≥ 40 from pilot cohort by end of Q2.
>
> **5. Functional Requirements — Scheduling**
> - FR-1 (Must): System detects timezone conflicts at slot-selection time and surfaces 3 alternative slots ranked by participant working hours.
> - FR-2 (Should): Calendar sync with Google + Outlook; conflicts visible within 30s of external change.
```

---

## Epic Decomposition

```
You are a senior delivery lead breaking a PRD into development epics.

## PRD Content
[Paste the full PRD or functional requirements section]

For each epic provide:
1. **Epic Name** — Short title
2. **Description** — 2-3 sentences
3. **Personas Affected**
4. **Key User Stories** — Full format (As a..., I want..., so that...)
5. **Dependencies** — Other epics this depends on
6. **Complexity** — T-shirt size (S/M/L/XL) with justification
7. **Priority** — Must Have / Should Have / Nice to Have

Organise in recommended implementation order. Identify cross-cutting concerns (auth, permissions, notifications) as separate epics.
```

---

## User Story Generation

```
Generate user stories for the following feature.

## Feature Context
- **Feature Name:** [name]
- **Epic:** [parent epic]
- **Description:** [what this feature does]
- **Target Persona(s):** [who uses it]

## Requirements
[Paste relevant PRD section]

For each story, use this format:

### Story: [Short title]
**As a** [persona], **I want** [goal], **So that** [benefit].

**Acceptance Criteria:**
- **Given** [precondition], **When** [action], **Then** [result]
- [Minimum: happy path + error case + edge case]

**Priority:** Must Have / Should Have / Nice to Have
**Size:** S / M / L / XL

Rules:
- Each story independently deliverable (1-3 days of work)
- Include error handling and edge case stories
- Focus on WHAT not HOW — no implementation details
- Full coverage of the feature with no gaps
```

---

## Acceptance Criteria

```
Generate acceptance criteria for this user story.

**As a** [persona], **I want** [goal], **So that** [benefit].

## Context
- **Feature:** [name]
- **Technical Notes:** [any constraints]

Cover ALL of the following in Given/When/Then format:
1. **Happy Path** — Standard successful flow (2-3 scenarios)
2. **Validation & Errors** — Invalid inputs, missing fields
3. **Boundary Conditions** — Min/max values, empty states, character limits
4. **Edge Cases** — Concurrent users, timeouts, partial data
5. **Permissions** — Unauthorised access, role restrictions
6. **Performance** — Response time expectations
7. **Accessibility** — Keyboard navigation, screen reader (if UI-facing)

Every criterion must be testable — a QA engineer should write a test directly from it.
```

---

## Gap Analysis

> If the PRD has already been published to a Linear Document (Step 2.4 onwards), provide the Document URL and ask Claude to read it via the Linear connector instead of pasting. The pasted-PRD form remains valid for the pre-publish self-review in Step 2.3.

```
You are a Chief Product Officer reviewing this PRD for completeness.

## PRD
[Paste complete PRD — OR — give the Linear Document URL and instruct Claude to read it via MCP]

Analyse systematically:

### 1. Functional Completeness
- Features implied by user flows but not documented?
- All CRUD operations covered per entity?
- All personas served? Error/empty states defined?

### 2. Non-Functional Completeness
- Performance targets? Scalability targets? Security requirements?
- Accessibility (WCAG)? Internationalisation? Data retention?

### 3. Consistency
- Contradictions between requirements?
- Terms used consistently? Metrics align with goals?

### 4. Assumptions & Risks
- Unstated assumptions? Undocumented external dependencies?
- Missing regulatory/compliance requirements?

### 5. Scope Clarity
- Clear in-scope vs out-of-scope boundary?
- Requirements too vague to implement?

For each gap:
- **Gap:** [Description]
- **Severity:** Critical / High / Medium / Low
- **Recommendation:** [How to fix]
- **Affected Section:** [Which PRD section]
```

---

## Interview Summary Structuring

```
Convert these stakeholder interview notes into structured requirements input.

## Raw Notes / Transcript
[Paste transcript from Fireflies.ai]

Extract and organise:
1. **Key Statements** — Requirements-relevant quotes with speaker identified
2. **Pain Points** — Ranked by frequency/emphasis
3. **Requested Features** — Explicitly requested capabilities
4. **Implied Requirements** — Not directly stated but implied
5. **Constraints** — Budget, timeline, technical, regulatory
6. **Conflicting Views** — Where stakeholders disagreed
7. **Open Questions** — Need follow-up
8. **Priority Indicators** — Language suggesting relative priority

Format as structured sections ready to feed into the PRD generation prompt.
```

---

## Linear Context Pull

**Linear-aware.** Run before drafting a PRD on an existing product to ground Claude in the current backlog state.

```
Using the Linear connector, pull a context bundle for an upcoming PRD on [FEATURE / PRODUCT NAME].

Search scope:
- Initiative(s): [name(s) or "auto-detect from feature description"]
- Team(s): [team names]
- Time window for issues: last [N] months

For each of the following, return a compact summary (no raw dumps):
1. **Active initiatives** — name, goal, current state, owner
2. **Active projects under those initiatives** — name, target date, state, % complete
3. **Recent or open issues that touch this feature area** — ID, title, state, labels (filter to last [N] months or "open")
4. **Closed issues in the last [N] months that overlap** — ID, title, what was actually shipped (one line each)
5. **Open gaps you can already see** — areas the feature description implies but no Linear issue covers

Output as a structured Markdown bundle ready to paste into the PRD generation prompt. Include the Linear IDs verbatim so the PRD can cite them.

If the Linear connector is not available, or if any of the inputs above are still unfilled placeholders, stop and ask — do not invent initiative names, project names, or issue IDs.

Do NOT create, update, or comment on anything in Linear in this step. Read-only.
```

---

## PRD-to-Linear Document

**Linear-aware. Writes to Linear.** Run during Step 2.4 — after PM self-review but **before** stakeholder review. Creates a Linear Project (Planned state) and publishes the PRD as a Linear Document attached to that Project. Stakeholders then review and comment on the Document directly in Linear.

```
Using the Linear connector, publish the PRD below to Linear.

## PRD (Markdown)
[Paste the full PRD draft. Use ATX headings (## , ### ) so anchors are stable.]

## Target placement
- Initiative: [existing initiative name or "create new: <name>"]
- Team: [team name]
- Document title: [e.g., "TimeSync v2 — Scheduling Engine PRD"]

## Action
1. **Confirm the plan before writing.** Output:
   - The **Linear Project** you intend to create: name, parent Initiative, initial state (must be `Planned`), labels (`phase:requirements`, `ai-generated`)
   - The **Linear Document** you intend to create: title, the section headings extracted from the PRD (so anchor URLs are predictable)
   Wait for me to reply "go" before calling any Linear write tool.

2. **On "go":**
   a. Create the Linear **Project** under the named Initiative.
      - Initial state: `Planned`
      - Description (short): one paragraph summary + the line "PRD lives in the attached Linear Document. Created by Claude via MCP — AI-DLC Phase 1."
      - Labels: `phase:requirements`, `ai-generated`
   b. Create a Linear **Document** attached to the Project, containing:
      - A header block with `**Status:** Draft v0.1 — pending stakeholder review`, `**Author:** Claude (AI-assisted) + [PM name]`, `**Date:** [today]`
      - The full PRD body, preserving all heading levels (Linear Documents use Markdown).
      - A trailing `## Sign-offs` section with empty checkboxes for `PM`, `Tech Lead`, `Sponsor`.
   c. Return as a Markdown table:
      | Item | URL |
      | Linear Project | <url> |
      | Linear Document | <url> |
      | Section anchors | <list of `#anchor-id` for the H2 sections, so issues can deep-link them> |

3. **Do NOT** create Milestones or Issues in this step. Those happen in Step 3 after Gate 1 approves the Document.

4. After creation, share the Document URL with me. Stakeholder review happens in Linear via Document comments (Step 2.5 in PROCESS.md).
```

---

## PRD-to-Linear Scaffold (Milestones)

**Linear-aware. Writes to Linear.** Run after Gate 1 (PRD Document approved) and before any stories are pushed. The Linear Project already exists from Step 2.4 — this prompt only adds Milestones.

```
Using the Linear connector, add Milestones to an existing Linear Project for the approved PRD.

## Inputs
- Linear Project URL (must already exist, must be out of `Planned` state): [URL]
- Linear PRD Document URL (status: Approved): [URL]
- Epics (from Stage 3a):
  [Paste the list of epics with one-line descriptions and target dates if known]

## Action
1. **Confirm before writing.** First, list:
   - The Milestones you intend to create (one per epic), in order
   - Target dates if the PRD timeline gives them
   - A note if any epic looks like it should map to multiple Milestones (rare — flag for me to decide)
   Wait for me to reply "go" before calling any Linear write tool.

2. **On "go":**
   - Create one Milestone per epic on the existing Linear Project, in PRD order, with the epic name and a one-line description.
   - Set target dates where the PRD provides them.
   - Return the created Milestone IDs and names as a Markdown table for review.

3. **Do NOT** create or modify the Project itself (it already exists). Do NOT create Issues — those are created by the `stories-to-linear-push` prompt after Gate 2.
```

---

## Stories-to-Linear Push

**Linear-aware. Writes to Linear.** Run after Gate 2 (milestone scaffold approved). Creates Triage-state issues with full AC and clickable PRD-section deep-links into the Linear Document.

```
Using the Linear connector, push the approved user stories below into the Linear Project.

## Stories with Acceptance Criteria
[Paste the stories produced by the user story + AC prompts. Each story must include: title, As-a/I-want/So-that, Given/When/Then AC, priority, size, and the PRD section it traces to (e.g., §3.2).]

## Project context
- Linear Project URL: [from Step 2.4]
- Linear PRD Document URL: [from Step 2.4 — used to construct deep-links]
- Section anchor map: [e.g., §3.2 → #functional-requirements—scheduling, §5.1 → #user-flows—booking. Provide from the prd-to-linear-document output.]
- Milestone mapping: [list which epic → which Milestone ID, from the scaffold step]

## Action
1. **Pre-flight duplicate check.** First, read the existing issues on the Project (and the parent Initiative). For each story you intend to push, check for an existing issue with a similar title or overlapping AC. If a likely duplicate is found, list it and DO NOT create — flag for me to resolve.

2. **Confirm the plan before writing.** Output a table:
   | # | Title | Milestone | Labels | PRD §X.Y → anchor URL | Duplicate? |
   Wait for me to reply "go" before calling any Linear write tool.

3. **On "go", for each story create one Linear issue with:**
   - **State:** `Triage` (never `In Progress` or beyond)
   - **Parent / Milestone:** as mapped above
   - **Labels:** `phase:requirements`, `ai-generated`, `needs-human-review`, plus `nfr` if it is a non-functional requirement
   - **Description (Markdown):**
     ```
     **As a** [persona], **I want** [goal], **So that** [benefit].

     ## Acceptance Criteria
     - **Given** ... **When** ... **Then** ...
     - (etc.)

     ---
     **PRD section:** [§X.Y](<LINEAR-DOCUMENT-URL>#<section-anchor>)
     **Priority:** [Must / Should / Nice]
     **Size:** [S/M/L/XL]
     **Source:** Generated by Claude via MCP — AI-DLC Phase 1
     ```
   - The PRD-section line MUST be a clickable Markdown link that opens the right heading in the Linear Document.

4. After creation, return a markdown table of `Issue ID | Title | URL | PRD §X.Y deep-link` so the PM can audit the AI Inbox by clicking each link to verify the citation.

Rules:
- Never set state beyond `Triage` / `Backlog`.
- Never edit or change the state of an existing issue.
- Every issue must include a clickable PRD-section deep-link. If a story does not have a valid `§X.Y` anchor in the section anchor map, refuse to create it and report which story.
```

---

## Linear Gap Sweep

**Linear-aware. Writes a single comment to Linear.** Run during Step 4 to diff the Linear Project against the PRD Document.

```
Using the Linear connector, perform a gap sweep against the approved PRD.

## Inputs
- Linear Project URL: [URL]
- Linear PRD Document URL (Approved v1.0): [URL]

## Action
1. Read the PRD directly from the Linear Document (do not ask me to paste it). Extract the section structure (§ numbers + headings).

2. Read the current state of the Linear Project: all Milestones, Issues (including state, labels, descriptions), and any existing Project comments.

3. For each PRD section, check whether at least one Linear Issue covers it (look at the issue descriptions for the `**PRD section:** [§X.Y](...)` deep-link). Flag:
   - **Missing coverage** — PRD section with no Issue citing it
   - **Weak coverage** — Issue exists but AC do not match the PRD requirement
   - **Orphan issues** — Issue without a clickable PRD-section deep-link
   - **Out-of-scope drift** — Issue that goes beyond the PRD or cites a section that no longer exists in the Document

4. Produce a single consolidated comment in Markdown with sections:
   - **Summary** — counts per category
   - **Critical gaps** — must fix before phase exit
   - **High** — should fix
   - **Medium / Low** — defer
   For each finding include: severity, the affected PRD §X.Y (as a deep-link to the Document), the Linear Issue ID (if any), and a one-line recommendation.

5. **Confirm before posting.** Show me the comment text. Wait for me to reply "post" before calling the Linear comment tool.

6. **On "post":** add the comment to the Linear Project's activity feed (not as an issue, not as new issues for each gap — one comment, on the Project).

Do NOT create new issues for the gaps. Issue creation, if needed, is handled by editing the PRD Document to incorporate the accepted gaps, then re-running `stories-to-linear-push` with the `gap-analysis` label after the PM accepts specific findings.
```

---

## Scalability & Cost Assessment

```
Review these requirements for scalability implications.

## PRD Summary
[Paste PRD or key sections]

## Expected Scale
- Launch users: [number]
- 12-month users: [number]
- Geographic regions: [regions]
- Data sensitivity: [PII / financial / health / etc.]

Provide:
1. **Scalability-Relevant Requirements** — Which requirements have scaling implications
2. **Infrastructure Pattern** — Recommended architecture pattern with justification
3. **Key Cloud Services** — Compute, database, cache, CDN needed
4. **Rough Cost Range** — Monthly at launch and 12 months (Low/Medium/High)
5. **Scaling Triggers** — Thresholds where architecture must change
6. **Cost Risks** — Where costs could spike unexpectedly
```
