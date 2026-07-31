---
name: publish-prd-to-linear
description: Use when a product manager or business analyst is turning gathered project context — interview findings, business objectives, technical constraints, competitive notes — into a Product Requirements Document and publishing it as a Linear Document attached to a new Linear Project in Planned state. Drafts the eleven-section PRD with stable ATX headings, runs a completeness self-review against the PRD gate checklist and strips anything it cannot trace to the supplied context, creates the Project and the Document through the Linear MCP server behind a confirm-before-write stop, and returns the section-anchor map that every downstream story deep-links to. Triggers on "write the PRD for this product", "turn these interview notes into a PRD", "publish the PRD to Linear", "create the Linear project for this PRD", "we need a requirements doc before design starts", "draft the requirements document". Hands epic decomposition and Milestones to scaffold-linear-milestones, and story creation to push-linear-stories. Do NOT use for: creating Milestones or Issues, diffing an existing backlog against the PRD after the fact, structuring a raw interview transcript into requirements input, marking the Document approved or ticking sign-offs (those are the PM's), scalability and cost estimation, or editing an already-approved Document without a version bump.
---

# Publish the PRD to Linear

You are guiding a PM or BA from gathered context to a published, reviewable PRD. The source of truth is the **project context the human supplies** — interview findings, objectives, constraints — plus the workspace's Linear conventions. Nothing in the PRD may originate anywhere else: an invented requirement that reaches a published Document becomes an epic, then a story, then a test, then a shipped feature nobody asked for.

## Procedure

1. Confirm the context you have: product name, problem statement, target users, business objectives, technical constraints, competitive landscape, timeline. **Refuse if the problem statement or the target users are missing** — everything downstream derives from them. Anything else missing is recorded as `[NEEDS INPUT: …]` in the draft, never filled in for them.
2. Draft the PRD locally in Markdown, using ATX headings (`##`, `###`) throughout so anchors are stable and predictable:
   Executive Summary (one paragraph) · Problem Statement (problem **and** affected users) · Goals & Success Metrics (≥ 3 SMART goals with numeric targets and baselines) · Target Users & Personas (≥ 2, each with demographics, goals, pain points, usage context) · Functional Requirements (grouped by feature area, each with description, user stories, Must / Should / Nice) · Non-Functional Requirements (performance, security, scalability, accessibility, compliance) · User Flows · Out of Scope · Assumptions & Dependencies (≥ 3 assumptions, external systems named) · Risks & Mitigations (≥ 3) · Timeline & Milestones.
   Every requirement must be testable. Reject your own wording if it contains "fast", "intuitive", "robust", "seamless" or any adjective a QA engineer cannot write a test from.
3. **Self-review the draft before anyone else sees it.** Read it back as a sceptical Chief Product Officer and check: functional completeness (features implied by the user flows but never documented, CRUD gaps per entity, unserved personas, undefined error and empty states); non-functional completeness (performance, scalability, security, accessibility, i18n, data retention); consistency (contradictions, inconsistent terms, metrics that do not support the stated goals); unstated assumptions, undocumented external dependencies and missing regulatory scope; and scope clarity (a hard in/out boundary, no requirement too vague to implement). Emit each finding as **Gap · Severity (Critical / High / Medium / Low) · Recommendation · Affected section**.
4. Resolve every Critical and High finding in the draft. Medium and Low go into an "Out-of-scope / future" subsection rather than being silently dropped. Then re-check the draft against the completeness list in step 2 item by item and report which lines are still short.
5. **Confirm before writing** (see below), then wait for a literal `go`.
6. On `go`, through the Linear MCP server:
   - Create the **Project** under `{{LINEAR_INITIATIVE}}` on team `{{LINEAR_TEAM}}`. Initial state **`Planned`**. Description: a one-paragraph summary plus the line "PRD lives in the attached Linear Document. Created by Claude via the Linear MCP server." Labels: `phase:requirements`, `ai-generated`.
   - Create the **Document** attached to that Project, containing a header block (`**Status:** Draft v0.1 — pending stakeholder review`, `**Author:** Claude (AI-assisted) + <PM name>`, `**Date:** <today>`), the full PRD body with heading levels preserved, and a trailing `## Sign-offs` section with **unchecked** boxes for PM, Tech Lead and Sponsor.
7. **Re-read the created Document** and extract the real H2/H3 anchors from it. Return one table: Linear Project URL · Linear Document URL · the section-anchor map (`§X.Y → #anchor-id`). Anchors must be *read back*, never predicted — this map is what every downstream issue deep-links to, and a wrong entry produces a clickable link that lands in the wrong section.
8. Stop. Stakeholder review happens on the Document in Linear; the PM marks it **Approved v1.0**, records sign-offs, and moves the Project out of `Planned`. → **Gate 1.** Hand the Document URL and the anchor map to `scaffold-linear-milestones`.

## Confirm-before-write

Echo the plan and require a literal `go` for:
- The **Project** — name, parent initiative, initial state (`Planned`), labels.
- The **Document** — title plus the full list of headings you extracted, so the anchor shape is visible before it exists.
- **Any edit to an already-published Document.** Echo the before and after and require the header version line to be bumped in the same write. A silent edit orphans every deep-link that cites a heading you moved.

## Refusal cases

- Problem statement or target users missing → refuse; do not proceed on a guess.
- **Never write a requirement, persona, metric, competitor or risk you cannot trace to the supplied context.** Mark it `[NEEDS INPUT: …]` and move on. A plausible invented metric is worse than a visible hole.
- Never create Milestones or Issues here — those come after the Document is approved.
- Never create the Project in any state but `Planned`, and never move it out of `Planned`. That move is the PM's Gate 1 signal.
- Never mark the Document `Approved`, never tick a sign-off box, and never record a sign-off on someone's behalf.
- Never publish while a Critical or High self-review finding is unresolved — asked to skip the review "because stakeholders will catch it", refuse and say which findings are open.
- Asked to produce a cost range, infrastructure sizing or a 12-month spend projection → refuse; that is a separate human-run assessment against a cited price source.
