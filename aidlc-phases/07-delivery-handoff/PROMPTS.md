# Phase 7: Delivery & Handoff — Prompt Templates

> **All prompts are for Claude or Claude Code.** semantic-release, the docs platform (Mintlify / GitBook / Docusaurus), the KB platform (Atlassian / Notion), Git, and the Anthropic Claude Code Action run natively — Claude generates and polishes content over MCP servers wired in [PROCESS.md Step 0](./PROCESS.md#step-0-one-time-setup--wire-ai-tools-into-the-handoff-loop).

---

## Release Notes Polish

```
You are a release-notes editor. Polish raw semantic-release output into user-friendly copy a Release Manager can publish with one round of review.

## Raw Notes from semantic-release
[Paste raw output from semantic-release / conventional commits]

## Release Context
- **Version:** [e.g., 2.5.0]
- **Audience:** [end users / developers / internal operations]
- **Star feature of this release:** [highlight the most impactful change]
- **Tone:** [professional / friendly / technical / marketing]
- **Previous CHANGELOG entry (voice reference):** [paste the last release's published notes so the polished output matches voice]

Transform the raw notes:
1. **Lead with impact** — What will users notice first? Put it at the top.
2. **Group clearly** — Features / Improvements / Bug Fixes / Breaking Changes
3. **Translate technical → user language** — "Refactored auth middleware" becomes "Faster sign-in (30% improvement)"
4. **Strip internal-only changes** — Chores, refactors, test updates that users don't care about
5. **Highlight breaking changes prominently** — At the top with migration guidance
6. **Include upgrade instructions** if any action is required
7. **Match the tone** — Matches our existing changelog style

Rules:
- Be specific. "Improved performance" → "Reduced API latency from 250ms to 180ms (p95)"
- Active voice. "We added X" not "X was added"
- Lead each bullet with a verb
- Include context for why a change matters when non-obvious
- **Never invent metrics, percentages, or behavior the raw notes do not support.** If the raw notes say "improved X" without a number, write "improved X" — do not fabricate "30% faster" to make the bullet punchier. Specificity is for facts the commits prove, not for marketing.
- **Flag uncertainty rather than guess.** If a commit's user-facing impact is unclear from its message, output `[NEEDS REVIEW: <commit hash> — <one-line question>]` instead of inventing a description.

Output as markdown ready to publish to GitHub Releases + CHANGELOG.md.
```

---

## Documentation Generation (per page)

```
Generate documentation for this [page type].

## Page Type
[Getting Started / User Guide / API Reference / Runbook / FAQ / etc.]

## Context
- **Product:** [brief description]
- **Target audience:** [developers / end users / ops team]
- **Prerequisites:** [what reader should know]

## Source Material
[Paste relevant code / architecture / operational info]

Generate the page with:
1. **Clear hierarchy** — H1 title, H2 sections, H3 subsections
2. **Action-oriented headings** — "Install X" not "Installation"
3. **Code examples** — Working, realistic examples (not `foo`/`bar`)
4. **Screenshots or diagrams referenced** — With descriptive alt text
5. **Common pitfalls** section — What users get wrong + how to fix
6. **Next steps** — Link to relevant adjacent pages

Voice:
- Second person ("you") for user guides
- Imperative ("run this command") for how-to
- Third person for reference docs
- Avoid "simply", "just", "easy" — these dismiss reader struggles

Output markdown ready for Docusaurus.
```

---

## Doc Completeness Check

```
Check documentation completeness for this project.

## Project Scope
[Paste PRD executive summary + main features]

## Documentation Currently Available
[List existing docs: README, API reference, runbooks, etc.]

## Code Overview
[Paste directory structure or point Claude Code to the repo]

Cross-reference the codebase against a complete documentation expectation:

### User-Facing Docs
- [ ] Getting Started / Quick Start
- [ ] Core features tutorials (one per major feature)
- [ ] How-to guides for common tasks
- [ ] FAQ

### Developer Docs
- [ ] Architecture overview
- [ ] API reference (from OpenAPI)
- [ ] ADRs (key decisions)
- [ ] Local development setup
- [ ] Contributing guide
- [ ] Testing guide

### Operations Docs
- [ ] Deployment guide
- [ ] Runbooks (one per alert)
- [ ] Monitoring overview
- [ ] Incident response playbook
- [ ] Disaster recovery plan

### Handoff Docs
- [ ] Handoff document
- [ ] KT recordings index

For each gap:
- **Missing item:** [What's missing — name the page, not a vague theme]
- **Priority:** [Critical / High / Medium / Low]
- **Rationale:** [Why this matters — tie to a concrete user, role, or operational risk, not "best practice"]
- **Suggested source:** [What to base it on — ADR number, file path, Linear label, runbook name]

Rules:
- Do NOT flag items that don't apply to this project (e.g., no end-user docs needed for a library; no DR plan for a hobby tool).
- Do NOT flag a gap if the item exists under a different name — search the docs index first; prefer EDIT over ADD when 70%+ of the content is already there.
- If you cannot tell whether a page exists from the inputs provided, output `[INSUFFICIENT INPUT: <what would let you decide>]` for that row instead of guessing.
- Group output by Priority (Critical → Low). Within a priority, order by which gap blocks the most other work.
```

---

## Handoff Document Generation

```
You are the delivery team's handoff lead. Generate a comprehensive handoff document by synthesising the artefacts below — every claim cites its source artefact inline, and you flag (rather than invent) anything the inputs do not cover.

## Project Artefacts to Synthesise
- **PRD:** [paste executive summary + goals]
- **Architecture:** [paste overview + key ADRs]
- **Tech Stack:** [from ADRs]
- **Codebase Structure:** [directory tree + brief description]
- **Test Coverage:** [current coverage % + test strategy]
- **Security Posture:** [current SAST/SCA status, compliance state]
- **Deployment:** [how deploys work, rollback strategy]
- **Observability:** [monitoring tools, SLOs, runbook inventory]
- **Known Issues:** [open tech debt, limitations]

## Handoff Context
- **Handing off from:** [delivery team]
- **Handing off to:** [recipient team, their background]
- **Support period:** [duration]
- **Handoff date:** [date]

Generate a handoff document with:

### Executive Summary (1 page)
- What was built in 2-3 sentences
- Current state (in production / beta / etc.)
- Top 3 things the recipient needs to know

### Product Overview
- Problem solved and users served
- Core features
- Key success metrics

### Architecture
- Component diagram with explanations
- Data flow overview
- Technology choices and why (reference ADRs)
- Scalability path

### Operations
- How to run locally
- How to deploy (staging + prod)
- How to monitor (dashboards + alerts)
- How to respond to incidents

### Security
- Current security posture summary
- Compliance status
- Known security debt

### Testing
- Test strategy summary
- Coverage metrics
- How to run tests
- Regression test baseline

### Known Limitations & Tech Debt
- Explicit list of things that aren't perfect
- Recommended prioritisation

### Recommended Next Steps
- For the recipient to consider first 30 days
- For the recipient to consider first 90 days
- Strategic considerations for next 12 months

### Contact & Support
- Post-handoff support terms
- Escalation path during support period
- Key contacts in the delivery team

### Appendices
- Link to all documentation
- Link to KT recordings
- Link to handoff checklist

Rules:
- **Cite sources inline.** Every architectural claim, ADR reference, runbook link, and "we decided X because Y" must point to the artefact it comes from. Uncited claims get removed during review — pre-empt the cut by citing as you write.
- **Never invent ADR numbers, component names, file paths, or coverage percentages.** If a number is not in the inputs, write `[VERIFY: <what to check>]` and continue.
- **Honesty over polish in Known Limitations.** A handoff document that hides tech debt is worse than one that lists it — the recipient finds it on day 8 anyway, and trust collapses.
- **Respect the customer-facing/internal split** from `AGENTS.md` Delivery & Handoff section. Internal post-mortem language, individual performance feedback, and raw incident transcripts do not appear in this document. If an input contains forbidden content, mark the affected paragraph `[REDACT-BEFORE-SHARE]` and continue — do not silently drop it.

Output as markdown ready for Delivery Lead + Tech Lead review.
```

---

## Knowledge Base Seeding (FAQ / Glossary / Troubleshooting)

```
Generate [FAQ / Glossary / Troubleshooting] content for the knowledge base.

## Context
- **Product:** [brief description]
- **Target users:** [who will read this]
- **Source material:** [PRD / code / Linear tickets / support conversations]

## Specific Request
[Which document type to generate: FAQ, Glossary, or Troubleshooting]

### For FAQ
Generate 15-20 frequently asked questions based on:
- Questions from KT sessions (paste transcripts)
- Anticipated questions from the PRD
- Common questions for this product type

For each question:
- **Q:** [Question in user's voice]
- **A:** [Clear, direct answer; link to more details where relevant]

Group by category: Getting Started / Billing / Features / Technical / Troubleshooting

### For Glossary
Extract domain terms from:
- PRD (product terminology)
- Architecture docs (technical terms)
- Codebase (internal naming)

For each term:
- **Term:** [name]
- **Definition:** [clear, one-paragraph explanation]
- **See also:** [related terms]
- **Used in:** [where in the system this appears]

### For Troubleshooting
Generate from:
- Linear bug history (paste summary — group by root-cause label so recurring causes show as counts)
- Known issues from QA

For each issue:
- **Symptom:** [What the user observes]
- **Likely cause:** [Most common root cause]
- **Diagnosis:** [How to verify the cause]
- **Resolution:** [Steps to fix]
- **Prevention:** [How to avoid recurrence]

Rules:
- **Only generate entries the source material supports.** Each FAQ/Glossary/Troubleshooting entry must trace to a specific transcript, PRD section, or Linear issue ID. Do not pad to hit a target count — 12 grounded FAQ entries beats 20 invented ones.
- **No template content.** "How do I get started?" with a generic answer is noise; either ground it in the actual onboarding flow or omit it.
- **Cross-link every entry** to the canonical code path, runbook, or ADR it references. KB pages with no outbound links rot fastest.

Output markdown ready for the knowledge base platform.
```

---

## Quarterly KB Completeness Check

> Used at PROCESS.md Step 7.5. Variant of Doc Completeness Check tuned for KB scope (FAQ + Glossary + Troubleshooting + Onboarding). Runs against the live KB via the Atlassian Rovo MCP or Notion MCP, and pulls live signal from Linear MCP.

```
Audit our knowledge base against live signal from the last 90 days and surface additions, edits, and deletions.

## KB Platform & Scope
- **Platform:** [Atlassian Confluence (via Rovo MCP) / Notion (via Notion MCP) / Docusaurus]
- **Spaces / pages in scope:** [paste root URLs or KB space keys — limit to handoff-seeded content]
- **Content types in scope:** [FAQ / Glossary / Troubleshooting / Onboarding — pick from PROCESS.md Step 7]

## Live Signal to Diff Against
- **Linear bugs (last 90d):** [paste from Linear MCP — title, root-cause label, severity, fix commit, opened/closed dates. Group by root-cause label and include the issue count per group — that count is the recurrence signal the rules below key off.]
- **KT session summaries appended since last check:** [paste links from handoff doc]
- **Production incidents (last 90d):** [paste from incident.io / PagerDuty]

## Existing KB Snapshot
[Paste KB page index or have Claude Code pull it via the docs/KB MCP — title, last-modified, owner, page URL]

For each finding, output one row in the table below:

| Action | KB Page | Reason | Source Signal | Suggested Owner | Effort |
|--------|---------|--------|----------------|-----------------|--------|
| [ADD] | [proposed page title + parent space] | [what's missing] | [Linear ID(s) / incident #] | [from CODEOWNERS or KB owner registry] | [S/M/L] |
| [EDIT] | [existing page URL] | [what's stale + suggested change] | [signal] | [page owner of record] | [S/M/L] |
| [DELETE] | [existing page URL] | [why it's no longer relevant] | [no traffic in last 90d / contradicts current code] | [page owner] | [S] |

Rules:
1. **Only flag changes the live signal supports** — do not invent gaps. If a root cause produced **3 or more separate Linear bug issues** in the window (or 1+ production incident) and isn't in Troubleshooting, that's an ADD. A root cause behind a single Linear issue and no incident is not — it is a one-off until it recurs.
2. **Prefer EDIT over ADD** when an existing page is 70%+ correct.
3. **Flag pages with no owner in the KB owner registry** as a DELETE candidate after 90 days of no edits — ownerless pages decay.
4. **Cross-link suggestions** — if an EDIT or ADD references code/runbook/ADR, include the link target.
5. Group output by Action (all ADDs first, then EDITs, then DELETEs). Sort within each group by descending live-signal weight — Linear issue count per root cause first, then severity, then incident count.

Output as a markdown table the recipient KB owner can paste straight into a Linear `kb-quarterly` issue with one sub-issue per row.
```

---

## KT Session Script

```
You are preparing a presenter for a knowledge-transfer session. Generate a script that lets them deliver the session without rehearsing twice — concrete timings, demo steps with fallbacks, and the three things the audience must leave understanding.

## Session Topic
[Architecture deep-dive / Code walkthrough / Deployment demo / Incident response drill]

## Audience
[Recipient team's background and experience]

## Duration
[30 / 45 / 60 minutes]

## Key Things to Cover
[List the 3-5 most important concepts]

Generate:

### Session Outline (with timings)
- 0:00 — Introduction and session goals
- 0:02 — [Topic 1] with demonstration
- 0:15 — [Topic 2] with code/diagram walkthrough
- ...
- X:00 — Q&A
- X:05 — Action items and next steps

### Demonstration Steps
For each demo segment:
- What to show on screen
- What to narrate
- Common questions to anticipate
- Fallback if something breaks live

### Key Messages
- The 3 things the audience MUST leave understanding
- Common misconceptions to proactively address

### Preparation Checklist
- Environment setup needed before session
- Materials to have open (docs, code, dashboards)
- Test the demo end-to-end beforehand

Output as structured markdown to guide the presenter.
```

---

## KT Session Summary

> Used at PROCESS.md Step 4.4. Run against a Fathom or Loom transcript right after the session ends, then append the output to the handoff document and seed runbook/KB updates downstream.

```
Summarise this knowledge transfer session into a structured, searchable reference and queue downstream updates.

## Session Metadata
- **Session topic:** [Architecture deep-dive / Code walkthrough / Deployment demo / Incident response drill / Q&A]
- **Date:** [YYYY-MM-DD]
- **Duration:** [actual minutes]
- **Presenter(s):** [delivery team names + roles]
- **Attendees:** [recipient team names + roles]
- **Recording link:** [Fathom / Loom URL]

## Source Material
- **Transcript:** [paste raw timestamped transcript from Fathom or Loom AI]
- **Demo artefacts shown on screen:** [list code paths, dashboards, runbooks, ADRs the presenter opened]
- **Customer-vs-internal context:** [from AGENTS.md Phase 7 section — pull the forbidden-content list inline]

Generate four sections:

### 1. Section Headings with Timestamps
A bulleted index of the session, one entry per topic shift, in the form:
- `[mm:ss]` **Topic** — one-sentence description of what's covered.

This becomes the chapter list pinned at the top of the recording.

### 2. Key Decisions per Timestamp
For every decision that was made in the session (architectural call, runbook change, escalation rule, "we will use X not Y"):
- **Decision:** [the decision]
- **Timestamp:** `[mm:ss]`
- **Rationale stated:** [presenter's reasoning, in their words]
- **Should this be an ADR?** [Yes / No — Yes if it changes a Phase 2 decision or sets a recipient-side convention]

### 3. Action Items per Owner
Every "I will…" or "we should…" or "let's follow up on…" extracted, grouped by owner:
- **Owner:** [name]
- **Action:** [what they committed to]
- **Due date if stated:** [date / "next session" / unspecified]
- **Linear issue to file:** [Yes — with title suggestion / No]

### 4. Open Questions + Cross-Link Targets
Two sub-lists:

**Open questions** — anything the presenter could not answer live:
- **Question:** [in attendee's voice]
- **Asked at:** `[mm:ss]`
- **Owner to research:** [name]

**Cross-link targets** — every artefact mentioned that should be linked from the handoff doc and KB:
- **Type:** [Runbook / ADR / Code path / Dashboard / Linear issue]
- **Reference:** [URL or repo path with line range]
- **Mentioned at:** `[mm:ss]`
- **Suggested KB section to backlink from:** [FAQ / Troubleshooting / Glossary / Onboarding]

Rules:
- **Strip filler.** "Um", "right", "does that make sense" — drop them.
- **Preserve presenter wording on rationale.** Do not paraphrase the "why" — the recipient needs the exact reasoning when they revisit the decision in 6 months.
- **Flag any content that breaches the AGENTS.md customer-vs-internal split** as `[REDACT-BEFORE-SHARE]` inline. Do not silently drop it; the Tech Lead reviews the redactions.
- **No invented content.** If the transcript is ambiguous, mark the entry `[AMBIGUOUS — review transcript at mm:ss]` rather than guessing.

Output as a single markdown block ready to append to the handoff document and to feed into the KB Seeding prompt downstream.
```

---

## Post-Handoff Retrospective

```
Generate a post-handoff retrospective document.

## Handoff Context
- **Project:** [name]
- **Handoff date:** [date]
- **Support period ending:** [date]

## Data Points
- **Issues raised during support period:** [count + categories]
- **Documentation updates required:** [list]
- **Gaps discovered post-handoff:** [list]
- **Recipient team satisfaction:** [feedback summary]

## Delivery Team Reflections
[Paste notes from the delivery team]

## Recipient Team Feedback
[Paste notes from the recipient]

Generate a retrospective document with:

### What Went Well
- Handoff artefacts that were most valuable
- Processes that worked smoothly
- AI-assisted elements that accelerated handoff

### What Could Be Improved
- Gaps in documentation or KT
- Process friction points
- Tool limitations

### Lessons Learned for Future AI-DLC Engagements
- Artefacts to standardise
- Process improvements
- Tool recommendations

### Recommendations for This Engagement
- Follow-up actions for recipient team
- Areas where continued delivery team involvement might add value

Rules:
- **Anchor every claim to a data point** — issue counts, satisfaction score, doc updates, gap list. "Documentation was strong" without a metric is sentiment; "12 of 14 support-period issues resolved without a doc gap" is a finding.
- **Be specific about what to standardise.** "Write better runbooks" is not actionable; "adopt the runbook-per-alert pattern from this engagement as the AI-DLC default" is.
- **Separate engagement-specific recommendations from framework-level lessons.** The first goes to the recipient; the second goes to the AI-DLC framework owner. Keep them in different sections.

Output as structured markdown for internal reference and surface the framework-level lessons to the AI-DLC framework owner.
```
