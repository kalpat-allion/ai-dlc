# Phase 1: Requirement Gathering — Process Flowcharts

The phase is split into six per-step flowcharts so each can be navigated, embedded in step-specific docs, or printed independently. The underlying process, sub-stages, and gate criteria live in [PROCESS.md](./PROCESS.md) and [QUALITY-GATES.md](./QUALITY-GATES.md); the diagrams here mirror that source-of-truth and chain end-to-end (each step's exit node feeds the next step's entry node).

## Table of Contents

- [Step 0: One-Time Setup](#step-0-one-time-setup)
- [Step 1: Stakeholder Interviews](#step-1-stakeholder-interviews)
- [Step 2: PRD Generation & Publish to Linear](#step-2-prd-generation--publish-to-linear)
- [Step 3: User Stories — Milestones & Issues](#step-3-user-stories--milestones--issues)
- [Step 4: Gap Analysis (Linear-diff)](#step-4-gap-analysis-linear-diff)
- [Step 5: Scalability & Cost](#step-5-scalability--cost)

## Legend

| Symbol | Meaning |
|--------|---------|
| 🤖 | AI-assisted step (Claude or Fireflies.ai) — no external write |
| 🔌 | Claude calling the **Linear MCP connector** (read or write) |
| 👤 | Human-led step |
| Diamond | Decision point / Quality gate |
| Dark navy node | Phase / step entry or exit |
| Purple node | One-time setup callout (Step 0) |
| Blue node | Linear MCP write action |
| Amber node | Cross-flowchart loop / fallback callout |

## Abbreviations

| Abbreviation | Meaning |
|--------------|---------|
| AC | Acceptance Criteria |
| AI | Artificial Intelligence |
| CLI | Command-Line Interface |
| MCP | Model Context Protocol (Anthropic open standard for tool/data connectors) |
| NFR | Non-Functional Requirement |
| OAuth | Open Authorization (delegated-access protocol) |
| PM | Product Manager |
| PRD | Product Requirements Document |

---

## Step 0: One-Time Setup

One-off connector wiring per workspace and per user. Path A covers Claude.ai web and Claude Desktop; Path B covers Claude Code (CLI / repo). Output is a verified Claude ↔ Linear MCP integration ready for Step 1's read-only context pull and Step 2's first write.

```mermaid
flowchart TD
    S0_START([Start: Phase 1 prerequisites<br/>Linear admin + Claude Pro/Max/Team/Enterprise]) --> S0_ADMIN

    S0_ADMIN[Workspace admin: confirm Linear plan exposes MCP<br/>+ allow-list mcp.linear.app + audit log on<br/>👤 Linear admin]
    S0_ADMIN --> S0_ANTH

    S0_ANTH[Anthropic admin Team/Enterprise: enable Linear connector<br/>scopes read + create issues + create comments<br/>update / state-change DISABLED until team mature<br/>👤 Anthropic admin]
    S0_ANTH --> S0_PATH{Choose surface}

    S0_PATH -- claude.ai web / Desktop --> S0_A1
    S0_PATH -- Claude Code CLI --> S0_B1

    S0_A1[Path A 1: Settings - Connectors - Linear - Connect<br/>OAuth in new tab - sign in - approve scopes<br/>👤 each user]
    S0_A1 --> S0_A2[Path A 2: Per-conversation toggle<br/>+ - Connectors - Linear ON<br/>off by default - explicit opt-in per chat<br/>👤 user]
    S0_A2 --> S0_TEST

    S0_B1[Path B 1: claude mcp add --transport http --scope user<br/>linear https://mcp.linear.app/mcp<br/>🤖 Claude Code]
    S0_B1 --> S0_B2[Path B 2: claude - /mcp - select linear - approve OAuth<br/>👤 user]
    S0_B2 --> S0_B3[Path B 3: claude mcp list - linear connected<br/>🤖 Claude Code]
    S0_B3 --> S0_TEST

    S0_TEST[Smoke test: prompt Claude<br/>List my Linear teams<br/>then create a draft issue labelled ai-generated<br/>🔌 Claude + Linear MCP]
    S0_TEST --> S0_VERIFY{Verification checklist:<br/>connector Connected,<br/>smoke-test issue in Triage,<br/>teams + projects readable,<br/>audit log on,<br/>update scope disabled?}

    S0_VERIFY -- No --> S0_ADMIN
    S0_VERIFY -- Yes --> S0_END([Setup complete<br/>Ready for Step 1: Stakeholder Interviews])

    style S0_START fill:#1B3A5C,color:#fff
    style S0_END fill:#1B3A5C,color:#fff
    style S0_ADMIN fill:#5C2E8A,color:#fff
    style S0_ANTH fill:#5C2E8A,color:#fff
    style S0_TEST fill:#3D6B9F,color:#fff
```

---

## Step 1: Stakeholder Interviews

Entry point is a verified Claude ↔ Linear MCP setup from Step 0. Sub-stages I1 → I5 cover interview prep, Fireflies.ai recording, AI-summary review, an optional read-only Linear context pull, and structuring findings with Claude. There is no gate at the end of Step 1 — the structured findings flow straight into Step 2's PRD context bundle. No writes to Linear in this step.

```mermaid
flowchart TD
    I_IN([From Step 0: Claude + Linear MCP verified]) --> I1

    I1[I1 Prepare interview guide<br/>key questions, agenda shared with stakeholders<br/>👤 PM] --> I2
    I2[I2 Conduct interviews<br/>Fireflies.ai auto-joins, records, transcribes,<br/>generates summary with action items<br/>🤖 Fireflies.ai] --> I3
    I3[I3 Review AI summaries within 24h<br/>correct errors, extract requirement-relevant statements<br/>👤 PM validates] --> I4
    I4[I4 Linear context pull - optional but recommended<br/>linear-context-pull prompt fetches initiatives,<br/>active projects, prior issues - READ-ONLY<br/>🔌 Claude + Linear MCP] --> I5
    I5[I5 Structure findings with Claude<br/>interview structuring prompt on transcripts<br/>+ Linear context bundle - chat output only<br/>🤖 Claude]
    I5 --> I_OUT([To Step 2: PRD Generation<br/>Inputs: structured findings + Linear context bundle])

    style I_IN fill:#1B3A5C,color:#fff
    style I_OUT fill:#1B3A5C,color:#fff
    style I4 fill:#3D6B9F,color:#fff
```

---

## Step 2: PRD Generation & Publish to Linear

Entry point is the structured findings + Linear context bundle from Step 1. Sub-stages 2.1 → 2.6 compile context, draft the PRD in Claude, self-review and PM-review the Markdown draft, publish to a Linear Project + Document via `prd-to-linear-document`, run stakeholder review inside Linear, and approve at Gate 1. On Gate 1 No, the loop returns to 2.2 to regenerate. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-prd-completeness--published-in-linear).

```mermaid
flowchart TD
    P_IN([From Step 1: Structured findings<br/>+ Linear context bundle]) --> P1

    P1[2.1 Compile context + Linear bundle<br/>interviews, brief, competitive data,<br/>technical constraints<br/>👤 PM gathers inputs] --> P2
    P2[2.2 Generate PRD v1<br/>PRD generation prompt - feed all context once<br/>cite Linear IDs in relevant sections<br/>Markdown only - no Linear write yet<br/>🤖 Claude] --> P3
    P3[2.3 Self-review + PM review<br/>gap-analysis prompt on draft<br/>+ PM checks PRD completeness checklist<br/>remove hallucinated requirements<br/>🤖 Claude + 👤 PM] --> P4
    P4[2.4 prd-to-linear-document<br/>Claude creates Linear Project Planned<br/>+ attached PRD Document with section anchors<br/>+ labels phase:requirements, ai-generated<br/>🔌 Claude + Linear MCP] --> P5
    P5[2.5 Stakeholder review IN LINEAR<br/>inline comments on the Document<br/>Guests added for non-engineering reviewers<br/>iterate Document via Claude or by hand<br/>👤 PM + stakeholders] --> G1

    G1{GATE 1: PRD Approved in Linear?<br/>see QUALITY-GATES.md Gate 1}
    G1 -- No --> P2
    G1 -- Yes --> P6[2.6 Mark Document Approved v1.0<br/>record sign-offs PM + Tech Lead + Sponsor,<br/>move Project out of Planned<br/>👤 PM]
    P6 --> P_OUT([To Step 3: User Stories<br/>Inputs: PRD Document URL + existing Project])

    style P_IN fill:#1B3A5C,color:#fff
    style P_OUT fill:#1B3A5C,color:#fff
    style P4 fill:#3D6B9F,color:#fff
```

---

## Step 3: User Stories — Milestones & Issues

Entry point is the approved PRD Document URL + existing Linear Project from Step 2. Stage 3a decomposes epics in chat only. Stage 3b runs `prd-to-linear-scaffold` to add Milestones to the existing Project, gated by Gate 2. Stage 3c runs `stories-to-linear-push` to create Triage issues with PRD deep-links, gated per-story by Gate 3 (AI Inbox empty). On Gate 2 No, the loop returns to S3B to regenerate the scaffold; on Gate 3 No, the loop returns to per-story acceptance. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-user-story-completeness-incl-linear-scaffolding).

```mermaid
flowchart TD
    S_IN([From Step 2: PRD Document Approved v1.0<br/>+ Project out of Planned]) --> S3A

    S3A[3a Decompose into epics<br/>epic-decomposition prompt against PRD Document URL<br/>chat only - no Linear write yet<br/>🤖 Claude] --> S3B
    S3B[3b prd-to-linear-scaffold<br/>Claude adds one Milestone per epic to existing Project<br/>milestone target dates from PRD timeline<br/>🔌 Claude + Linear MCP] --> G2

    G2{GATE 2: Milestones Approved?<br/>see QUALITY-GATES.md Gate 2}
    G2 -- No --> S3B
    G2 -- Yes --> S3C[3c Generate stories + AC<br/>persona/goal/benefit + Given/When/Then<br/>happy + error + edge cases<br/>🤖 Claude]

    S3C --> PUSH[stories-to-linear-push<br/>Triage issues with labels<br/>phase:requirements, ai-generated, needs-human-review<br/>+ PRD section deep-links into the Document<br/>🔌 Claude + Linear MCP]
    PUSH --> INBOX[AI Inbox review<br/>filter ai-generated AND needs-human-review<br/>per story: click deep-link, verify, edit, accept<br/>delete stories without valid PRD citation<br/>👤 PM]
    INBOX --> G3{GATE 3: AI Inbox Empty?<br/>see QUALITY-GATES.md Gate 2 Linear scaffolding}

    G3 -- No --> INBOX
    G3 -- Yes --> S_OUT([To Step 4: Gap Analysis<br/>Inputs: PRD Document + Milestones + accepted Backlog])

    style S_IN fill:#1B3A5C,color:#fff
    style S_OUT fill:#1B3A5C,color:#fff
    style S3B fill:#3D6B9F,color:#fff
    style PUSH fill:#3D6B9F,color:#fff
```

---

## Step 4: Gap Analysis (Linear-diff)

Entry point is the PRD Document + accepted backlog from Step 3. Stages G1 → G5 run the gap-analysis prompt against the Document, run `linear-gap-sweep` to post a consolidated comment on the Project, prioritise findings, and route them: Critical/High edit the PRD and loop back into Step 3 Stage 3c with a `gap-analysis` label; Medium/Low are documented inside the PRD Document under "Out-of-scope / future". The cross-flowchart loop is rendered as an amber callout because the actual Stage 3c push lives in Step 3's flowchart. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-final-review-phase-handoff).

```mermaid
flowchart TD
    G_IN([From Step 3: PRD Document<br/>+ Milestones + accepted Backlog]) --> G1N

    G1N[G1 Run gap analysis<br/>gap-analysis prompt referencing PRD Document URL<br/>missing NFRs, unstated assumptions,<br/>contradictions, dependency gaps, security omissions<br/>🤖 Claude reads Document via MCP] --> G2N
    G2N[G2 linear-gap-sweep<br/>Claude reads PRD Document + Project state<br/>Milestones + Issues - posts ONE consolidated<br/>comment on the Linear Project<br/>NO auto-issues - comment only<br/>🔌 Claude + Linear MCP] --> G3N
    G3N[G3 Optional market check<br/>competitor coverage + industry standards<br/>Claude training data + manual web research<br/>🤖 Claude] --> G4N
    G4N[G4 Prioritise findings<br/>consolidate into Critical / High / Medium / Low<br/>👤 PM] --> G5N

    G5N{Severity?}
    G5N -- Critical / High --> GHI[Edit PRD Document + bump version<br/>👤 PM resolves]
    GHI --> GLOOP[Loop back to Step 3 Stage 3c<br/>re-run stories-to-linear-push<br/>add label gap-analysis to new stories<br/>then re-clear AI Inbox at Gate 3]
    GLOOP --> G_OUT
    G5N -- Medium / Low --> GLOW[Document remaining gaps in PRD Document<br/>under Out-of-scope / future section<br/>👤 PM]
    GLOW --> G_OUT([To Step 5: Scalability & Cost<br/>Inputs: refreshed PRD Document + accepted gap stories])

    style G_IN fill:#1B3A5C,color:#fff
    style G_OUT fill:#1B3A5C,color:#fff
    style G2N fill:#3D6B9F,color:#fff
    style GLOOP fill:#7A5C1B,color:#fff
```

---

## Step 5: Scalability & Cost

Entry point is the refreshed PRD Document from Step 4. Sub-stages SC1 → SC2 extract scalability factors with Claude, validate cost ranges against free cloud calculators (AWS / Azure / GCP), and add NFRs to the PRD Document (optionally pushing `nfr` labelled issues). The Final Gate is the Phase 1 handoff readiness check covering all of Gates 1–3 plus scalability/cost. On No, the loop returns to Step 2 Stage 2.1 to refresh inputs — rendered as an amber callout because that target sits in Step 2's flowchart. On Yes, the phase exits to Phase 2: System Design.

```mermaid
flowchart TD
    SC_IN([From Step 4: Refreshed PRD Document<br/>+ accepted gap stories]) --> SC1

    SC1[SC1 Analyse requirements<br/>extract scalability factors from PRD<br/>estimate infra cost ranges<br/>validate vs AWS / Azure / GCP free calculators<br/>🤖 Claude + Cloud calculators] --> SC2
    SC2[SC2 Add NFRs to PRD Document<br/>performance, scalability, cost targets<br/>optional: push nfr-labelled issues to Linear<br/>👤 PM - 🔌 if pushing nfr issues] --> FG

    FG{Final Gate: Handoff Ready?<br/>PRD Approved v1.0 + AI Inbox empty<br/>+ Critical/High gaps resolved + NFRs in Document<br/>see QUALITY-GATES.md Gate 3}
    FG -- No --> FGRETRY[Loop back to Step 2 Stage 2.1<br/>refresh context bundle and regenerate PRD<br/>re-run Steps 2 - 5 as needed]
    FGRETRY --> SC1
    FG -- Yes --> SC_OUT([→ Phase 2: System Design<br/>Handoff: single Linear Project URL<br/>= PRD Document + Milestones + accepted Backlog])

    style SC_IN fill:#1B3A5C,color:#fff
    style SC_OUT fill:#1B3A5C,color:#fff
    style FGRETRY fill:#7A5C1B,color:#fff
```

---

## Three Human Gates

The flow has three explicit human gates so that no Claude-authored Linear item reaches an Active state without sign-off:

1. **Gate 1 — PRD Approved (in Linear).** PM marks the Linear Document as Approved v1.0 after stakeholder review on the Document; Project leaves Planned state. Required before any Milestones or Issues are created.
2. **Gate 2 — Milestones Approved.** PM accepts the Milestone scaffold added to the existing Project before stories are pushed.
3. **Gate 3 — AI Inbox Empty.** Every `ai-generated AND needs-human-review` issue is accepted (or deleted) — clicking each PRD deep-link is part of the per-story acceptance — before phase handoff. The Final Gate (Step 5) re-checks all three plus scalability/cost completeness.

---

## Related Documents

- [Process Definition →](./PROCESS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Prompt Templates →](./PROMPTS.md)
- [PRD Template →](../templates/prd-template.md)
- [User Story Template →](../templates/user-story-template.md)
