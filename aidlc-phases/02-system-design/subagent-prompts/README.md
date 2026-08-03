# Phase 2 Design Subagent Prompts

This directory holds the **system-prompt templates** for the Phase 2 specialist Claude Code subagents referenced in [`../PROCESS.md`](../PROCESS.md) (Step 1 architecture design and Step 4 accessibility review). They are templates, not active subagents — Claude Code only auto-discovers files under `.claude/agents/`.

All three are **name-invoked**, so they face no auto-trigger competition. The auto-triggering counterparts for this phase live in [`../skill-prompts/`](../skill-prompts/). Boundaries against artifacts shipped by other phases are recorded in [`docs/ROUTING.md`](../../../docs/ROUTING.md); placeholders below that recur under different spellings elsewhere are reconciled in [`docs/PLACEHOLDERS.md`](../../../docs/PLACEHOLDERS.md).

**`solution-architect` and `architecture-reviewer` were deliberately not merged.** They cover the same design and read almost like two halves of one job, and merging them would be a real convenience — but the architecture gate requires a review by someone *who is not the author*, and a same-session follow-up to your own proposal cannot honestly satisfy that. Merged, the checkbox still gets ticked and the guarantee is gone. Each template therefore refuses the other's work explicitly, in its description and again in its escalation list. **Do not relax either refusal when adapting these files.**

## Files

| File | Role | Wraps |
|------|------|-------|
| [`solution-architect.md`](./solution-architect.md) | The system-level architecture pass: 2-3 candidate architectures with components, communication, data architecture, infrastructure, bounded contexts, trade-offs at launch and at 10x and top risks; every seriously-considered option stress-tested at 10x; a justified recommendation that stops at a Tech Lead approval gate. Read-only apart from the proposal document | [`architecture-proposal`](../PROMPTS.md#architecture-proposal), [`trade-off-interrogation`](../PROMPTS.md#trade-off-interrogation) |
| [`architecture-reviewer.md`](./architecture-reviewer.md) | The independent pre-build production-readiness review of the chosen architecture: severity-ranked findings across scalability, security, reliability, operability and cost, an explicit Unevaluated list, and a PASS / PASS WITH FIXES / FAIL verdict derived mechanically from the Critical and High counts. Read-only | [`design-review`](../PROMPTS.md#design-review) |
| [`accessibility-auditor.md`](./accessibility-auditor.md) | One WCAG 2.1 AA audit of one component, screen, or page — findings as Issue / Severity / Recommendation / success criterion, a mandatory statement of what a source read cannot measure, and a verdict that can only reach PASS with the automated scan plus a keyboard pass on the rendered page. Read-only | [`accessibility-review`](../PROMPTS.md#accessibility-review-claude) |

The **Wraps** column is this directory's purpose. Shipped templates must stand alone in a consuming repo that has no `aidlc-phases/` tree, so each agent **inlines** its procedure with `{{PLACEHOLDERS}}` instead of linking back to `PROMPTS.md` — this table is the only record of where each procedure came from. Keep it current, or the round-trip is lost.

## How to instantiate per repo

1. Copy the chosen template into `.claude/agents/<role>.md` at the consuming repo root.
2. Replace the placeholders with the repo's values:
   - `{{ARCHITECTURE_DIR}}` — where architecture proposals and design-review reports live, e.g. `docs/architecture/` (solution-architect, architecture-reviewer)
   - `{{ADR_DIR}}` — where the recorded architecture decisions live, e.g. `docs/adrs/`, `docs/decisions/` (solution-architect, architecture-reviewer)
   - `{{TEAM_STACK}}` — the team's current stack, e.g. `Next.js 14 + TypeScript`, `FastAPI + Python 3.12`. **Same value as the Phase 3 specialists use** (solution-architect)
   - `{{CLOUD_PROVIDER}}` — e.g. `AWS`, `GCP`, `Azure`, or `undecided` for genuinely greenfield work. **Same value the infrastructure helpers use** (solution-architect)
   - `{{FRONTEND_FRAMEWORK}}` — e.g. `React 18`, `Next.js 14`, `SvelteKit` (accessibility-auditor)
   - `{{FRONTEND_ROOT}}` — where UI code lives, e.g. `apps/web` (accessibility-auditor)
   - `{{A11Y_SCAN_COMMAND}}` — the exact command that runs the automated accessibility scan against a rendered page, e.g. `pnpm test:a11y`, `npx @axe-core/cli http://localhost:3000`. **This is not the unit-test command** — it must render the page, or the auditor's PASS condition can never be met (accessibility-auditor)
   - `{{A11Y_REPORT_PATH}}` — where the WCAG report is committed, e.g. `docs/accessibility/wcag-aa-report.md` (accessibility-auditor)
3. Adjust the `model:` frontmatter if the team's default differs. All three ship as `opus` — each is a judgement pass whose output is a severity call or a design commitment, not high-throughput authoring.
4. **Do not weaken three specific rules when adapting these templates.** Each is the only thing standing where nothing downstream would contradict a wrong answer:
   - `solution-architect`'s refusal to review its own proposal, and `architecture-reviewer`'s refusal to review a design produced in its own session.
   - `architecture-reviewer`'s consequence-based severity table and the rule that the verdict follows the counts. Severity written by tone is how a blocker becomes a Medium.
   - `accessibility-auditor`'s rule that a source read can never return PASS. Contrast, reflow, and computed tap-target size are not in the source, and a PASS is the verdict that stops anyone else looking.
5. Commit `.claude/agents/<role>.md` to the repo — the file is shared infrastructure; treat edits as code changes requiring review.
6. Verify with `/agents` in a Claude Code session — the role should appear with its description. Per-role smoke tests:
   - `solution-architect` — *"Use the solution-architect to propose architecture options for this product from the requirements doc, and stop at the approval question."* A passing run produces 2-3 options, stress-tests each seriously-considered one against all six questions, gives a recommendation with a specific rejection reason per loser, and ends at the verbatim approval line without starting schema or contract work.
   - `architecture-reviewer` — *"Use the architecture-reviewer to review the chosen architecture for production readiness."* A passing run returns severity-ranked findings with a consequence and a triggering load on every Critical and High, a non-empty Unevaluated section where inputs were thin, and a verdict that matches its own counts.
   - `accessibility-auditor` — *"Use the accessibility-auditor to audit this screen against WCAG 2.1 AA."* A passing run reports findings with success-criterion references, **names what a source read could not measure**, and does not return PASS unless it actually ran the scan and described a keyboard traversal. This is the single most important check on this agent.

## Routing

These three are name-invoked, so the description is a routing claim rather than an auto-trigger. It is still the whole boundary surface — run both checks below after installing more than one.

**Negative-routing check.** Ask for ordinary work these agents must not claim, in plain language, and confirm none of them loads:

- *"implement the checkout endpoint"* and *"review my diff before I PR"* — development work; both belong to the repo's implementation and code-review specialists.
- *"design the architecture for ENG-247 before I scaffold"* — a story-scoped design pass inside an existing codebase. `solution-architect` must **not** take this; it is the per-story design specialist's, and the seam is a story identifier plus an existing codebase versus a product and a blank slate.
- *"write the migration for the new tables"* and *"generate the OpenAPI spec"* — `data-model-design` and `api-contract-freeze`.
- *"draw the container diagram"* — `render-design-diagrams`.
- *"write up the ADR for the decision we just made"* — `/adr`.

**Refusal check.** Drive each agent into its sharpest refusal and confirm it redirects rather than complies:

- `solution-architect` — *"we don't have the NFR numbers yet, just assume something sensible and give me the options"*, then *"great, option B it is — now review it and tell me it's production-ready"*. It must refuse both: the first because invented targets decide the architecture, the second because it cannot be its own independent reviewer.
- `architecture-reviewer` — *"the gate is this afternoon, can you call those two Highs Mediums so we can pass"*, and *"you found the problem, just rewrite the design so it works"*. It must restate the counts and refuse the redesign.
- `accessibility-auditor` — *"the contrast looks fine to me, just mark it PASS"* and *"we can't run the dev server today, give me a PASS from the code"*. It must return `PASS WITHHELD` and name what is missing, rather than producing a verdict the evidence does not support.

The seam worth re-reading before editing any description here: `solution-architect` and `architecture-reviewer` share a subject and split on **direction of travel** — one produces the design, the other audits it — and that split is a gate requirement, not a tidiness preference. `accessibility-auditor` shares the word "review" with the repo's code-review tooling and splits on **what is being reviewed**: a rendered interface against a published standard, never a diff.
