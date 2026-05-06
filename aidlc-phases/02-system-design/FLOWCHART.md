# Phase 2: System Design — Process Flowcharts

The phase is split into six per-step flowcharts so each can be navigated, embedded in step-specific docs, or printed independently. The underlying process, sub-stages, and gate criteria live in [PROCESS.md](./PROCESS.md) and [QUALITY-GATES.md](./QUALITY-GATES.md); the diagrams here mirror that source-of-truth and chain end-to-end (each step's exit node feeds the next step's entry node).

## Table of Contents

- [Step 0: One-Time Setup](#step-0-one-time-setup)
- [Step 1: Architecture Design](#step-1-architecture-design)
- [Step 2: Data Modelling](#step-2-data-modelling)
- [Step 3: API Contract Design](#step-3-api-contract-design)
- [Step 4: UI/UX Wireframing](#step-4-uiux-wireframing)
- [Step 5: Tech Stack Definition](#step-5-tech-stack-definition)

## Legend

| Symbol | Meaning |
|--------|---------|
| 🤖 | AI/tool-driven action (Claude, Claude Code, Claude Design, Eraser MCP, Linear MCP) |
| 👤 | Human-led action |
| Diamond | Decision point or quality gate |
| Dark blue node | Phase / step entry or exit |
| Mid blue node | One-time setup callout |
| Amber node | Fallback / escalation branch |

## Abbreviations

| Abbreviation | Meaning |
|--------------|---------|
| 3NF | Third Normal Form (relational schema design) |
| ADR | Architecture Decision Record |
| AC | Acceptance Criteria |
| API | Application Programming Interface |
| C4 | Context / Container / Component / Code architecture model |
| CI | Continuous Integration |
| CLI | Command-Line Interface |
| CSS | Cascading Style Sheets |
| DSL | Domain-Specific Language |
| ER / ERD | Entity-Relationship / Entity-Relationship Diagram |
| FK | Foreign Key |
| HTML | HyperText Markup Language |
| IaC | Infrastructure as Code |
| MCP | Model Context Protocol |
| NFR | Non-Functional Requirement |
| OAuth | Open Authorization |
| OpenAPI | Open API Specification (formerly Swagger) |
| ORM | Object-Relational Mapping |
| PM | Product Manager |
| PRD | Product Requirements Document |
| RPC | Remote Procedure Call |
| SPOF | Single Point of Failure |
| SR | Screen Reader |
| SVG / PNG | Scalable Vector Graphics / Portable Network Graphics |
| UI / UX | User Interface / User Experience |
| WCAG | Web Content Accessibility Guidelines |

---

## Step 0: One-Time Setup

One-off connector wiring per workspace and per user. Path A covers Claude.ai web and Claude Desktop; Path B covers Claude Code (CLI / repo). Claude Design enablement is a separate first-party check (no MCP). Output is a verified ability to drive Eraser from inside Claude and to use Claude Design surfaces in Step 4.

```mermaid
flowchart TD
    S0_START([Start: Phase 2 prerequisites<br/>Eraser admin + Claude Pro/Max/Team/Enterprise]) --> S0_ADMIN

    S0_ADMIN[Workspace admin: Eraser plan exposes MCP<br/>+ allow-list app.eraser.io + audit log on<br/>👤 Eraser admin]
    S0_ADMIN --> S0_ANTH

    S0_ANTH[Anthropic admin Team/Enterprise: enable Eraser connector<br/>scopes generate + generateEdit + export + search<br/>delete DISABLED until team mature<br/>👤 Anthropic admin]
    S0_ANTH --> S0_PATH{Choose surface}

    S0_PATH -- claude.ai web / Desktop --> S0_A1
    S0_PATH -- Claude Code CLI --> S0_B1

    S0_A1[Path A 1: Settings - Connectors - Eraser - Connect<br/>OAuth in new tab - sign in - approve<br/>👤 each user]
    S0_A1 --> S0_A2[Path A 2: Per-conversation toggle<br/>+ - Connectors - Eraser ON<br/>👤 user]
    S0_A2 --> S0_TEST

    S0_B1[Path B 1: claude mcp add --transport http --scope user<br/>eraser https://app.eraser.io/api/mcp<br/>🤖 Claude Code]
    S0_B1 --> S0_B2[Path B 2: claude - /mcp - select eraser - approve OAuth<br/>API-key fallback for CI: ERASER_API_KEY env<br/>👤 user]
    S0_B2 --> S0_B3[Path B 3: claude mcp list - eraser connected<br/>🤖 Claude Code]
    S0_B3 --> S0_TEST

    S0_TEST[Smoke test: ask Claude via Eraser MCP<br/>to list workspaces + create test sequence diagram<br/>🤖 Claude + Eraser MCP]
    S0_TEST --> S0_VERIFY{Verification checklist:<br/>connector Connected,<br/>diagram appears,<br/>search + export work,<br/>audit log on,<br/>delete scope disabled?}

    S0_VERIFY -- No --> S0_ADMIN
    S0_VERIFY -- Yes --> S0_CD

    S0_CD[Claude Design enablement check<br/>Pro/Max/Team: on by default<br/>Enterprise: Admin - Features - Claude Design ON<br/>👤 admin if Enterprise]
    S0_CD --> S0_END([Setup complete<br/>Ready for Step 1: Architecture Design])

    style S0_START fill:#1B3A5C,color:#fff
    style S0_END fill:#1B3A5C,color:#fff
    style S0_ADMIN fill:#2A5A8A,color:#fff
    style S0_ANTH fill:#2A5A8A,color:#fff
    style S0_CD fill:#2A5A8A,color:#fff
```

---

## Step 1: Architecture Design

Entry point is the Phase 1 Linear Project URL (PRD Document + backlog). Sub-stages 1.1 → 1.8 produce architecture options, Eraser + Mermaid diagrams, a design review, and ADRs. Gate 1 is Tech Lead approval; on No, the loop returns to A2 to regenerate options. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-architecture).

```mermaid
flowchart TD
    A_IN([From Phase 1: Linear Project URL<br/>PRD Document v1.0 + backlog]) --> A1

    A1[1.1 Pull PRD + Linear context<br/>read-only, record PRD URL + version<br/>🤖 Claude + Linear MCP] --> A2
    A2[1.2 Generate 2-3 architecture options<br/>architecture-proposal prompt<br/>🤖 Claude] --> A3
    A3[1.3 Trade-off interrogation<br/>10x stress-test, SPOFs, cost cliffs<br/>🤖 Claude + 👤 Tech Lead] --> A4
    A4[1.4 Render diagrams in Eraser<br/>cloud + system + sequence<br/>DSL committed to /docs/diagrams/eraser/<br/>🤖 Claude + Eraser MCP] --> A5
    A5[1.5 In-repo C4 + Mermaid<br/>Context + Container - /docs/diagrams/mermaid/<br/>🤖 Claude] --> A6
    A6[1.6 Design review<br/>severity-ranked issues - resolve all<br/>Critical and High<br/>🤖 Claude design-review prompt] --> A7
    A7[1.7 Create ADRs per significant decision<br/>cite PRD section + reject alternatives<br/>🤖 Claude - /docs/adrs/] --> A8
    A8[1.8 Team review<br/>capture objections in ADR Consequences<br/>👤 Dev team] --> G1

    G1{GATE 1: Architecture<br/>approved by Tech Lead?<br/>see QUALITY-GATES.md Gate 1}
    G1 -- No --> A2
    G1 -- Yes --> A_OUT([To Step 2: Data Modelling<br/>Inputs: ADRs + diagrams + chosen architecture])

    style A_IN fill:#1B3A5C,color:#fff
    style A_OUT fill:#1B3A5C,color:#fff
```

---

## Step 2: Data Modelling

Entry point is the approved architecture from Step 1. Sub-stages 2.1 → 2.6 extract entities, generate ORM-format schema, render ER diagrams in Eraser and Mermaid, run a backend review, and produce migrations + seeds. There is no dedicated gate at the end of Step 2 — schema and API contracts are gated together at the end of Step 3 (Gate 2). Step 2 exits straight into Step 3.

```mermaid
flowchart TD
    D_IN([From Step 1: ADRs + diagrams approved<br/>chosen architecture + bounded contexts]) --> D1

    D1[2.1 Extract entities from PRD<br/>attributes, relationships, cardinalities<br/>cross-check vs bounded contexts<br/>🤖 Claude + Linear MCP] --> D2
    D2[2.2 Generate schema in ORM format<br/>Prisma / TypeORM / Drizzle / SQL<br/>indexes covering top query patterns<br/>🤖 Claude Code - schema-generation prompt] --> D3
    D3[2.3 ER diagram in Eraser<br/>DSL committed + PNG/SVG export<br/>🤖 Claude + Eraser MCP] --> D4
    D4[2.4 Mermaid erDiagram in repo<br/>/docs/diagrams/mermaid/erd.mmd<br/>🤖 Claude] --> D5
    D5[2.5 Backend review<br/>3NF, FK cardinalities, covering indexes,<br/>any denormalisation needs an ADR + measured query<br/>👤 Backend dev] --> D6
    D6[2.6 Migrations + seed data<br/>5-10 rows per entity - run cleanly locally<br/>🤖 Claude Code]
    D6 --> D_OUT([To Step 3: API Contract Design<br/>Inputs: frozen schema + migrations])

    style D_IN fill:#1B3A5C,color:#fff
    style D_OUT fill:#1B3A5C,color:#fff
```

---

## Step 3: API Contract Design

Entry point is the reviewed schema from Step 2. Sub-stages 3.1 → 3.5 map flows to endpoints, generate the OpenAPI 3.1 spec, walk it in Swagger UI, stand up Prism mock servers, and review/freeze. Gate 2 covers both data model and API contracts — on No, the loop returns to D2 (schema regeneration) since gate failures most often originate in the schema. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-data-model--api-contracts).

```mermaid
flowchart TD
    P_IN([From Step 2: Schema + migrations clean]) --> P1

    P1[3.1 Map PRD flows to endpoints<br/>resource model, verbs, naming<br/>🤖 Claude + Linear MCP] --> P2
    P2[3.2 Generate OpenAPI 3.1<br/>endpoints, schemas, error envelope,<br/>pagination, idempotency, x-prd-section<br/>🤖 Claude Code - api-contract prompt<br/>- /docs/api/openapi.yaml] --> P3
    P3[3.3 Walk every endpoint in Swagger UI<br/>local Docker or Swagger Editor<br/>👤 Tech Lead] --> P4
    P4[3.4 Prism mock server live<br/>prism mock docs/api/openapi.yaml<br/>URL added to README<br/>👤 Tech Lead / dev] --> P5
    P5[3.5 Review + freeze contracts<br/>any change after freeze requires<br/>PM approval + impact analysis<br/>👤 Tech Lead + backend dev] --> G2

    G2{GATE 2: Data model + API<br/>contracts approved?<br/>see QUALITY-GATES.md Gate 2}
    G2 -- No --> D2_RETRY[Loop back to 2.2<br/>schema regeneration in Step 2]
    G2 -- Yes --> P_OUT([To Step 4: UI/UX Wireframing<br/>Inputs: frozen OpenAPI + mock URL])

    style P_IN fill:#1B3A5C,color:#fff
    style P_OUT fill:#1B3A5C,color:#fff
    style D2_RETRY fill:#7A5C1B,color:#fff
```

---

## Step 4: UI/UX Wireframing

Entry point is the frozen API contract from Step 3. Sub-stages 4.1 → 4.7 onboard the design system into Claude Design, enable the frontend-design Skill, generate wireframes per flow, iterate via inline comments, optionally hand off to Claude Code, run an accessibility review, and capture stakeholder sign-off. Gate 3 is PM approval. The fallback branch (UFB → Figma / v0 / Cursor + shadcn / Bolt) covers cases Claude Design genuinely cannot meet; fallback flows still re-enter the accessibility review at U6. On Gate 3 No, the loop returns to U3 to regenerate wireframes. See [QUALITY-GATES.md → Gate 3](./QUALITY-GATES.md#gate-3-wireframes--tech-stack).

```mermaid
flowchart TD
    U_IN([From Step 3: Frozen OpenAPI + mock URL<br/>+ PRD user flows + brand assets]) --> U1

    U1[4.1 Onboard design system<br/>link repo + brand - inherit tokens<br/>🤖 Claude Design reads repo] --> U2
    U2[4.2 Enable frontend-design Skill<br/>Settings - Skills - frontend-design ON<br/>👤 Architect] --> U3
    U3[4.3 Generate wireframes per flow<br/>live HTML/CSS/React on canvas<br/>loading, empty, error, success<br/>desktop 1440 + mobile 375<br/>🤖 Claude Design - claude-design-wireframe] --> U4
    U4[4.4 Iterate via inline comments + chat<br/>targeted region edits + global tweaks<br/>👤 PM + designers] --> U5
    U5[4.5 Production handoff if needed<br/>Handoff to Claude Code - branch + PR<br/>preserves design-system tokens<br/>🤖 Claude Design - Claude Code] --> U6
    U6[4.6 Accessibility review<br/>WCAG 2.1 AA - keyboard, focus, contrast,<br/>SR labels, semantic HTML, motion-reduce<br/>🤖 Claude accessibility-review prompt] --> U7
    U7[4.7 Stakeholder review on canvas<br/>inline comments - iterate - approve<br/>👤 PM + stakeholders] --> G3

    G3{GATE 3: Wireframes<br/>approved by PM?<br/>see QUALITY-GATES.md Gate 3}
    G3 -- No --> U3
    G3 -. Claude Design cannot cover .-> UFB
    UFB[Fallback: Figma pixel work /<br/>Bolt / Lovable interactive demo /<br/>v0 per-component drop-in /<br/>Cursor + shadcn IDE-driven<br/>record reason in flow ADR]
    UFB --> U6
    G3 -- Yes --> U_OUT([To Step 5: Tech Stack Definition<br/>Inputs: approved wireframes + WCAG report])

    style U_IN fill:#1B3A5C,color:#fff
    style U_OUT fill:#1B3A5C,color:#fff
    style UFB fill:#7A5C1B,color:#fff
```

---

## Step 5: Tech Stack Definition

Entry point is the approved wireframes plus the architecture from Step 1 and team expertise profile. Sub-stages 5.1 → 5.4 frame each decision, generate scored comparisons, write ADRs, and capture team buy-in plus training plans. Gate 4 is the phase handoff readiness check — on No, fix missing artifacts and re-evaluate; on Yes, the phase exits to Phase 3: Development. See [QUALITY-GATES.md → Gate 4](./QUALITY-GATES.md#gate-4-phase-handoff).

```mermaid
flowchart TD
    T_IN([From Step 4: Approved wireframes<br/>+ architecture + team expertise]) --> T1

    T1[5.1 Frame each decision<br/>language, framework, datastores, cache,<br/>broker, IaC, observability, auth<br/>+ constraints per decision<br/>👤 Tech Lead] --> T2
    T2[5.2 Scored comparisons<br/>1-5 matrix across criteria + recommendation<br/>Perplexity for fast-moving areas<br/>🤖 Claude tech-stack-comparison] --> T3
    T3[5.3 ADRs per decision<br/>rejected alternatives with reasons<br/>🤖 Claude adr-generation - /docs/adrs/] --> T4
    T4[5.4 Team buy-in + training plans<br/>every unfamiliar choice gets a plan<br/>book / course / mentor / time budget<br/>👤 Dev team] --> G4

    G4{GATE 4: Phase handoff<br/>complete?<br/>see QUALITY-GATES.md Gate 4}
    G4 -- No --> FIX[Fix missing artifacts<br/>re-link Linear issues<br/>👤 Tech Lead]
    FIX --> G4
    G4 -- Yes --> T_OUT([To Phase 3: Development<br/>Handoff: ADRs + diagrams + schema +<br/>OpenAPI + wireframes + WCAG + stack ADRs])

    style T_IN fill:#1B3A5C,color:#fff
    style T_OUT fill:#1B3A5C,color:#fff
    style FIX fill:#7A5C1B,color:#fff
```

---

## Related Documents

- [Process Definition →](./PROCESS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Prompt Templates →](./PROMPTS.md)
- [ADR Template →](../templates/adr-template.md)
