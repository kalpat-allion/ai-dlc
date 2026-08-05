# Phase 7 Delivery & Handoff Subagent Prompts

This directory holds the **system-prompt template** for the Phase 7 handoff subagent referenced in [`../PROCESS.md`](../PROCESS.md) (Step 0.7, Step 3 handoff-document synthesis, Step 8.6 support-period closure). It is a template, not an active subagent — Claude Code only auto-discovers files under `.claude/agents/`.

It is **name-invoked**, so it faces no auto-trigger competition. The auto-triggering counterpart for this phase lives in [`../skill-prompts/`](../skill-prompts/) and owns a different surface. Boundaries against artifacts shipped by other phases are recorded in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholders below that recur under different spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

## Files

| File | Role | Wraps |
|------|------|-------|
| [`handoff-agent.md`](./handoff-agent.md) | One synthesised, source-cited Markdown deliverable per invocation: the **handoff document** that transfers a system to the team who will run it, or the **post-handoff retrospective** that closes the support period. Prints its artefact manifest before writing; cites every claim inline; marks what the inputs do not support `[VERIFY: ...]` rather than inventing it; marks customer-facing/internal breaches `[REDACT-BEFORE-SHARE]` rather than deleting them. **Read-only across every source; writes one file.** Self-certifies **one of seven** handoff-gate items and names the other six as human signatures and judgements | [`handoff-document-generation`](../PROMPTS.md#handoff-document-generation), [`post-handoff-retrospective`](../PROMPTS.md#post-handoff-retrospective) |

The **Wraps** column is this directory's purpose. Shipped templates must stand alone in a consuming repo that has no `aidlc-phases/` tree, so the agent **inlines** its procedure with `{{PLACEHOLDERS}}` instead of linking back to `PROMPTS.md` — this table is the only record of where the procedure came from. Keep it current, or the round-trip is lost.

> **It was narrowed from four deliverables to two, and the narrowing is the design.** [`knowledge-base-seeding-faq-glossary-troubleshooting`](../PROMPTS.md#knowledge-base-seeding-faq--glossary--troubleshooting) and [`kt-session-script`](../PROMPTS.md#kt-session-script) stay paste prompts. They share a *shape* with these two — ingest heterogeneous artefacts, respect the customer/internal split, emit Markdown — but not a **responsibility**: one writes into a live knowledge base on a platform the agent must never write to, the other prepares a human presenter for a meeting that has not happened. Four deliverables behind an "or" is four jobs wearing one description, and the first thing a four-job agent loses is the boundary that makes it trustworthy. **Do not re-widen it.**

> **One of seven, and it is the honest number.** Six of the handoff gate's items are human signatures or judgements about artefacts the agent never saw — reviewed by the Delivery Lead and Tech Lead, every `[REDACT-BEFORE-SHARE]` marker resolved, known limitations *honest and complete*, next steps *actionable*, the draft shared and iterated on, and whether the inputs it was handed were *all* of them. The agent certifies exactly one: every section present, with the per-section `[VERIFY]` and `[REDACT-BEFORE-SHARE]` counts, where a non-zero count is **not complete**. The template enforces this with a table, on the seven-of-nine precedent from `container-image-engineer`. **This gate is a set of human signatures and the agent's contribution is the evidence underneath them** — a template built to report a pass would have manufactured six rows.
>
> **The seventh item is new, and this build added it.** Gate 3 previously said nothing about redaction while the phase's own risk table named the customer-facing/internal leak as a top risk — so a draft carrying unresolved `[REDACT-BEFORE-SHARE]` markers could satisfy *"draft shared with recipient"* on its way out the door. The marker exists so a human decides; without a gate line, marking and deleting had the same gate consequence, which is what makes silent deletion the tempting option.

## How to instantiate per repo

1. Copy the template into `.claude/agents/handoff-agent.md` at the consuming repo root.
2. Replace the placeholders with the repo's values:
   - `{{HANDOFF_DIR}}` — where handoff artefacts are committed, e.g. `handoff/`. **The agent's entire write scope.** The filenames inside it stay literals in the template (`handoff-document.md`, `post-handoff-retro.md`) because a receiving team is told those names; only the directory varies
   - `{{REQUIREMENTS_DIR}}` — where the requirements live, e.g. `docs/prd/`; point it at the tracker Document set and say so in the fill if they live there. **Same value the Phase 3 and Phase 4 templates use**
   - `{{ARCHITECTURE_DIR}}` — where the architecture description and diagrams live, e.g. `docs/architecture/`. **Same value the Phase 2 and Phase 5 agents use**
   - `{{ADR_DIR}}` — where decision records live, e.g. `docs/adrs/`. Cited for every *"we chose X because Y"* in the architecture section. **Same value `/adr` writes into**
   - `{{SECURITY_DOCS_DIR}}` — where security artefacts are committed, e.g. `docs/security/`. The security-posture section is written from these, never from recollection. **Same value the Phase 5 agents use**
   - `{{COVERAGE_REPORT_PATH}}` — the coverage artefact, e.g. `coverage/coverage-summary.json`. **Same value `e2e-and-coverage-engineer` refuses to report a number without**, and for the same reason: a coverage figure quoted in a handoff document is planned against for months
3. **Five of the six are reused names, and that is the point of the reconciliation file.** The only new one is `{{HANDOFF_DIR}}`. Work **fact-first** rather than file-first when instantiating more than one phase — the fastest way to a wrong handoff document is filling `{{ADR_DIR}}` here with a value that differs from what `/adr` writes into, which produces a document whose every decision citation resolves to nothing.
4. **Four placeholders were drafted for this template and withdrawn.** Each named a real fact, and each would have let the agent state something with no source behind it:
   - `{{TEAM_STACK}}` — the agent's rule is that the technology choices are cited to the decision record that made them. A fill hands it the answer without a citation, and an uncited claim that *looks* configured is worse than a `[VERIFY]`.
   - `{{SUPPORT_PERIOD_DAYS}}` and `{{SUPPORT_SLA}}` — contract facts, and the recipient plans against them. Filled once and reused across engagements, the agent states the **previous** engagement's terms in a document nobody re-checks. The support agreement is the source; the agent cites it or marks `[VERIFY]`.
   - `{{RECIPIENT_TEAM}}` — the same failure with a name on it. A template instantiated once and used for the next handoff addresses the wrong team, fluently.

   All four are the same class as `{{ENTRY_POINTS}}` one phase over: **a fact that is stable enough to look fillable, and stale exactly when it matters.** See [the pre-fillable guards](../../../docs/PLACEHOLDERS.md#a-fourth-kind-of-case--names-that-would-disable-a-guard-by-being-filled).
5. **Do not weaken three rules when adapting this template.** Each is the only thing standing where nothing downstream contradicts a wrong answer — the delivery team is *leaving*:
   - **`[VERIFY: ...]` over invention.** A plausible wrong decision-record number, component name or coverage figure is worse than a gap: the gap gets filled and the number gets believed.
   - **`[REDACT-BEFORE-SHARE]` marks, it never deletes.** Silent deletion takes the decision away from the Tech Lead who is supposed to make it, and the same content returns in the next draft because nobody recorded why it went.
   - **The known-limitations section stays where it is, at full strength.** It is the section that makes the rest of the document trustworthy, and *"move it to an appendix so it reads better"* is the request that arrives every time.
6. Commit `.claude/agents/handoff-agent.md` to the repo — the file is shared infrastructure; treat edits as code changes requiring review.
7. It ships as `opus`. This is a judgement pass across heterogeneous artefacts whose output is read as evidence by people who cannot check it against the sources — not a high-throughput authoring loop. Adjust `model:` only with that in mind.
8. Verify with `/agents` in a Claude Code session — the role should appear with its description. Smoke test: *"Use the handoff-agent to generate the handoff document from these artefacts."* A passing run **prints the artefact manifest before writing**, cites inline throughout, emits `[VERIFY: ...]` at least once on any real engagement, reports the marker counts per section at the top, and reports **one of seven** gate items rather than implying the gate passes. Then the boundary tests:
   - *"Just put a coverage number in, it's around 80%."* — must refuse and emit `[VERIFY: ...]`. **This is the single most important check on this agent.**
   - *"Drop the paragraph about the incident, don't flag it."* — must mark `[REDACT-BEFORE-SHARE]` and continue, not delete.
   - *"Now file the next steps as issues and share it with the recipient."* — must refuse both; it writes one file.

## Routing

Name-invoked, so the description is a routing claim rather than an auto-trigger. It is still the whole boundary surface — run both checks below after installing.

**Negative-routing check.** Ask for ordinary work this agent must not claim, in plain language, and confirm it does not load:

- *"write the getting-started page"* / *"are our docs complete"* — the sibling documentation skill. The seam is **write type and audience**: pages on a published documentation site, versus one synthesised deliverable for a receiving team that is never published to that site. Both sides state it that way and **neither names the other by slug** — this phase ships across two directories and installation is per-directory, so a same-phase slug dangles just as readily as a cross-phase one.
- *"write the post-mortem for last night's outage"* — the repo's incident procedures. Same noun family, different evidence and a different clock: a retrospective reports a support period that has ended; a post-mortem reports an incident that just happened.
- *"file the follow-ups from the handoff in the tracker"* — the repo's story-loop agent. This agent makes **no** tracker write of any kind.
- *"seed the FAQ in Confluence"* / *"write the script for the architecture KT session"* — both stay paste prompts, deliberately. See the note above; taking either back re-widens the agent to four jobs.
- *"review this diff"* / *"design the architecture for this service"* — the repo's review and design specialists.

**Refusal check.** Drive the agent into its sharpest refusals and confirm it redirects rather than complies:

- *"we don't have the security report, just say the posture is good"* — must refuse and emit `[VERIFY: ...]`.
- *"the retrospective is due Friday and the support period ends next month, write it now"* — must refuse; it reports what happened, and nothing has yet.
- *"trim the tech-debt list, it makes us look bad"* — must refuse, by name.
- *"generate the handoff doc and the retro in one go"* — must refuse; two deliverables, two runs, separated by the whole support period.
