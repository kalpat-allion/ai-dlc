# Phase 1: Requirement Gathering — Summary Flowchart

A three-row, horizontal end-to-end view of the six per-step flowcharts in [FLOWCHART.md](./FLOWCHART.md). Each row is its own `flowchart LR` diagram so the contents render left-to-right on every Mermaid renderer (no nested-direction support needed). Read the rows in order — top to bottom — and use the "→ Row N" exit badges to follow the flow between them. The per-step diagrams remain the source of truth: open [FLOWCHART.md](./FLOWCHART.md) to drill into sub-stages, prompt names, and gate criteria.

## Legend

| Symbol | Meaning |
|--------|---------|
| 🤖 | AI-assisted step (Claude Code or Fireflies.ai) — no external write |
| 🔌 | Claude Code calling the **Linear MCP server** (read or write) |
| 👤 | Human-led step |
| Diamond | Decision point / Quality gate |
| Dark navy node | Phase entry or exit |
| Purple node | One-time setup callout (Step 0) |
| Blue node | Step containing Linear MCP write actions |
| Amber node | Cross-step loop / fallback callout |

## Row 1 — Setup and PRD

```mermaid
flowchart LR
    START([Phase 1 Entry<br/>Linear admin + Claude Pro/Max/Team/Enterprise]) --> S0[Step 0: One-Time Setup<br/>Wire Claude Code ↔ Linear MCP - project scope<br/>claude mcp add --scope project → committed .mcp.json<br/>Auth once per project · Smoke test + verify<br/>👤 Admins · 🤖 Claude Code · 🔌 MCP]
    S0 --> S1[Step 1: Stakeholder Interviews<br/>Fireflies.ai records + transcribes · PM validates<br/>Optional Linear context pull READ-ONLY<br/>Claude Code structures findings - local only<br/>👤 PM · 🤖 Fireflies + Claude Code · 🔌 MCP]
    S1 --> S2[Step 2: PRD Generation<br/>Compile context → PRD v1 → self + PM review<br/>prd-to-linear-document publishes Project + Document<br/>Stakeholder review IN LINEAR<br/>👤 PM · 🤖 Claude · 🔌 MCP]
    S2 --> G1{GATE 1<br/>PRD Approved in Linear?}
    G1 -- No --> S2
    G1 -- Yes --> NEXT1([→ Row 2<br/>Step 3: User Stories])

    style START fill:#1B3A5C,color:#fff
    style S0 fill:#5C2E8A,color:#fff
    style S2 fill:#3D6B9F,color:#fff
    style NEXT1 fill:#1B3A5C,color:#fff
```

## Row 2 — User Stories and Gap Analysis

```mermaid
flowchart LR
    FROM1([← From Row 1<br/>Gate 1: Yes]) --> S3[Step 3: User Stories<br/>3a Decompose epics chat-only<br/>3b prd-to-linear-scaffold → Milestones<br/>3c Stories + AC → Triage Issues w/ PRD deep-links<br/>👤 PM · 🤖 Claude · 🔌 MCP]
    S3 --> G2{GATE 2<br/>Milestones Approved?}
    G2 -- No --> S3
    G2 -- Yes --> G3{GATE 3<br/>AI Inbox Empty?<br/>per-story accept/edit/delete}
    G3 -- No --> S3
    G3 -- Yes --> S4[Step 4: Gap Analysis<br/>gap-analysis prompt vs PRD Document<br/>linear-gap-sweep posts ONE consolidated comment<br/>Optional market check · Prioritise findings<br/>👤 PM · 🤖 Claude · 🔌 MCP]
    S4 --> SEV{Severity?}
    SEV -- "Critical / High" --> LOOPBACK[Edit PRD + bump version<br/>Loop back to Step 3 above<br/>label: gap-analysis · re-clear AI Inbox]
    LOOPBACK --> S3
    SEV -- "Medium / Low" --> NEXT2([→ Row 3<br/>Step 5: Scalability and Cost])

    style FROM1 fill:#1B3A5C,color:#fff
    style S3 fill:#3D6B9F,color:#fff
    style S4 fill:#3D6B9F,color:#fff
    style LOOPBACK fill:#7A5C1B,color:#fff
    style NEXT2 fill:#1B3A5C,color:#fff
```

## Row 3 — Scalability and Handoff

```mermaid
flowchart LR
    FROM2([← From Row 2<br/>Severity: Medium / Low]) --> S5[Step 5: Scalability & Cost<br/>Extract scalability factors from PRD<br/>Validate cost vs AWS / Azure / GCP calculators<br/>Add NFRs to PRD Document<br/>👤 PM · 🤖 Claude · 🔌 optional MCP for nfr issues]
    S5 --> FG{FINAL GATE<br/>Handoff Ready?<br/>Gates 1-3 + NFRs in Document<br/>+ Critical/High gaps resolved}
    FG -- No --> RETRY[Refresh context bundle<br/>Loop back to Row 1 · Step 2 Stage 2.1<br/>re-run Steps 2-5 as needed]
    FG -- Yes --> END([→ Phase 2: System Design<br/>Handoff: single Linear Project URL<br/>= PRD Document + Milestones + accepted Backlog])

    style FROM2 fill:#1B3A5C,color:#fff
    style END fill:#1B3A5C,color:#fff
    style RETRY fill:#7A5C1B,color:#fff
```

## How to view these diagrams

The Mermaid blocks above render natively in any Mermaid-compatible viewer:

1. **GitHub / GitLab** — the blocks render inline when this Markdown file is viewed in the repo.
2. **VS Code** — with the built-in Markdown preview plus a Mermaid extension, or the Markdown Preview Mermaid Support extension.
3. **[mermaid.live](https://mermaid.live)** — paste a single `flowchart` block (without the triple backticks) to pan, zoom, and export as SVG/PNG.

## What this summary collapses

| Source diagram | Collapsed to |
|----------------|--------------|
| Step 0: One-Time Setup (project-scoped `claude mcp add` + auth + smoke test + verify) | Single node `S0` |
| Step 1: I1 → I5 (prep, record, validate, context pull, structure) | Single node `S1` |
| Step 2: 2.1 → 2.6 (compile → draft → review → publish → stakeholder review → approve) | Single node `S2` + Gate 1 |
| Step 3: 3a, 3b, 3c (epics → milestones → stories) | Single node `S3` + Gate 2 + Gate 3 |
| Step 4: G1 → G5 (gap analysis → sweep → market check → prioritise → route) | Single node `S4` + severity branch |
| Step 5: SC1 → SC2 (scalability factors → NFRs to Document) | Single node `S5` + Final Gate |

## The three human gates (recap)

1. **Gate 1 — PRD Approved (in Linear)** — PM marks Document `Approved v1.0`; Project leaves Planned.
2. **Gate 2 — Milestones Approved** — PM accepts the Milestone scaffold before stories push.
3. **Gate 3 — AI Inbox Empty** — every `ai-generated AND needs-human-review` issue accepted (or deleted). Final Gate (Step 5) re-checks all three plus scalability/cost.

See [QUALITY-GATES.md](./QUALITY-GATES.md) for full pass/fail criteria.

## Related Documents

- [Per-step Flowcharts →](./FLOWCHART.md)
- [Process Definition →](./PROCESS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Prompt Templates →](./PROMPTS.md)
