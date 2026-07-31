# Artifact Routing Map

> **What this is.** One map of every shipped subagent and skill in this repository, organised by **the work it owns** rather than by the phase directory it happens to live in. Its job is to answer one question: *when a developer says X, which artifact should take it, and which must not?*
>
> **Why it is not per-phase.** Artifacts are **not scoped to the phase that ships them.** `linear-task-agent` is used whenever someone works a story, in month one or month nine. `publish-prd-to-linear` is used whenever requirements are written, including a mid-project change request. A boundary between two artifacts used across phases cannot be expressed inside either phase's README — which is exactly why this file exists.
>
> **Scope: 18 shipped artifacts** — 7 development subagents, 2 DevOps subagents, 6 DevOps skills, 3 requirement skills. Proposed-but-unbuilt artifacts are listed in [PROMPT-CONVERSION-ANALYSIS.md](./PROMPT-CONVERSION-ANALYSIS.md) and enter this map when they ship, not before.

---

## The one rule

**Skills auto-trigger from their description; subagents are name-invoked but can still be auto-selected.** So the `description:` field is a claim on user utterances, and two artifacts claiming the same utterance is a defect — the router picks one, silently, and the loser's guarantees never run.

Boundaries are therefore drawn on **three axes, in this order**:

1. **Write type** — what the artifact creates or mutates (a Linear Document vs. an issue state; a `backend` block vs. resource HCL). This is the most durable axis, because it does not move when the project does.
2. **File ownership** — which paths it writes. Two artifacts touching `.github/workflows/` need a file-level split, not a topic-level one.
3. **Verb** — the literal words a user says. Least durable, most immediate: a stolen verb hijacks ordinary work on day one.

**Never draw a boundary on phase.** "Use this during Phase 1" is not a boundary; a team in month six doing requirements work is still doing requirements work.

---

## Master map

The **Provenance** column records which phase ships the template. It is a lookup path, **not a usage restriction**.

### Tracker writes (Linear MCP)

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `publish-prd-to-linear` | Skill | The requirements Document, the Project that carries it, the section-anchor map | Milestones, Issues, marking anything approved | P1 |
| `scaffold-linear-milestones` | Skill | Milestones — creation, order, target dates | The Project, the Document, Issues | P1 |
| `push-linear-stories` | Skill | First-time story creation from a requirements document: Triage issues, their AC, labels, deep-links | Milestones, any state move past Triage, clearing `needs-human-review` | P1 |
| `linear-task-agent` | Subagent | Writes **around a story already in flight**: find, start, progress comment, PR-open, done, follow-up filing, field updates | Backlog construction, sprint planning, code | P3 |

**The tie-break, stated as a write type:** the three skills **construct the backlog**; `linear-task-agent` **operates on a story that already exists in it**. Both write issues, so "does it create an issue?" is not the test. The test is *where the issue comes from* — a requirements document (skills) or a developer working a story who found something (agent).

> **⚠ This boundary is documented here but not yet enforced in `linear-task-agent.md`.** Its scope-discipline rule fences writes to *other* projects; the three skills write into the *same* project, so nothing in the template currently refuses backlog construction. The edit is specified in PROMPT-CONVERSION-ANALYSIS.md → *Extend, don't create*, and until it lands this row describes intent rather than behaviour. **Do not read this table as verification.**

### Application code

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `software-architect` | Subagent | Per-story design pass inside an existing codebase, read-only, stops at developer approval | System-wide architecture, new service boundaries, new datastores, implementation | P3 |
| `frontend-engineer` | Subagent | UI implementation **and its component tests**, in the same commit | Server-side code, E2E suites, review | P3 |
| `backend-engineer` | Subagent | Server-side implementation **and its integration tests**, migrations inside a story | UI code, E2E suites, review | P3 |
| `refactor-specialist` | Subagent | Refactoring executed against an existing `tech-debt` issue | Identifying candidates for filing, behaviour changes | P3 |
| `code-reviewer` | Subagent | Pre-PR self-review of a diff, read-only | Applying fixes, post-open PR review, architecture review | P3 |
| `conflict-resolver` | Subagent | Merge / rebase / cherry-pick conflict resolution on the working tree | Anything not conflict-shaped; never `--abort`s unasked | P3 |

**Test-writing is deliberately not a separate owner.** The implementation specialists claim unit and integration tests by **positive** trigger, because the test ships in the same commit as the code. An artifact that claimed "write the tests" generally would take that work away from them mid-story.

### Infrastructure and pipelines

| Artifact | Type | Owns | Never owns | Provenance |
|---|---|---|---|---|
| `terraform-iac-engineer` | Subagent | Resource HCL, modules, plan-only tests; loops until plan is clean. **Holds no apply credential** | The `backend` block, the state store, CI credentials, anything in `.github/workflows/`, `apply` | P6 |
| `iac-state-backend-bringup` | Skill | The `backend` block, state store, lock, state IAM roles, `.gitignore` state entries | Resource HCL, workflows, CI credentials | P6 |
| `ci-identity-and-secrets-bringup` | Skill | OIDC provider and trust policy, secret-manager wiring, the drift workflow | The state store, build workflows, the Infracost workflow, resource HCL | P6 |
| `cost-guardrails-bringup` | Skill | The Infracost workflow, budget alerts, the unit-economics metric, the cost reviewer checklist | The cost estimate itself, any other workflow, approving a costly PR | P6 |
| `cicd-pipeline-bringup` | Skill | Build / test / deploy-to-staging workflows, `@claude` wiring, agentic workflows, branch protection | The drift workflow, the Infracost workflow, the production gate | P6 |
| `deploy-and-rollback-bringup` | Skill | The production approval gate, deploy strategy, rollback trigger, the drill | The build pipeline, SLO definitions | P6 |
| `container-image-engineer` | Subagent | One service's Dockerfile and `.dockerignore`; self-certifies 7 of 9 standards, names the other 2 | CVE counts, scan verdicts, image-size comparisons — **under any circumstances** | P6 |
| `observability-bringup` | Skill | Dashboards, SLOs, alerts, alert runbooks under `/docs/runbooks/alerts/` | Instrumentation code, service runbooks under `/docs/runbooks/services/`, incident work | P6 |

**`.github/workflows/` has four claimants and the split is by file, not topic** — build/test (`cicd-pipeline-bringup`), drift (`ci-identity-and-secrets-bringup`), Infracost (`cost-guardrails-bringup`), production gate (`deploy-and-rollback-bringup`). A topic-level boundary here would produce four artifacts that each believe they own "the CI setup".

---

## Contested surfaces

Where two artifacts could plausibly take the same sentence. These are the rows worth re-reading before editing any description.

| Utterance | Goes to | Not to | Because |
|---|---|---|---|
| "create the issues for this epic" | `push-linear-stories` | `linear-task-agent` | Backlog construction from a requirements document, not work on a story in flight |
| "file a bug for that auth race" | `linear-task-agent` | `push-linear-stories` | Discovered while working a story; no requirements document behind it |
| "write the tests for this" | `frontend-engineer` / `backend-engineer` | anything else | Ships in the same commit as the code |
| "review this PR for cost" | *nothing yet* | `code-reviewer` | `cost-guardrails-bringup` deliberately contains **no** form of "review" in its triggers, precisely to avoid this collision. It produces a checklist a human uses |
| "set up terraform for this repo" | `iac-state-backend-bringup` | `terraform-iac-engineer` | The backend must exist before resources; the subagent assumes it does |
| "add the S3 bucket module" | `terraform-iac-engineer` | `iac-state-backend-bringup` | Resource HCL, not backend bring-up |
| "write the runbook" | `observability-bringup` **if alert-triggered** | — | Alert runbooks live under `alerts/`; service runbooks under `services/` are Phase 3-owned and are a different artifact class |
| "how many CVEs in this image" | *nothing* | `container-image-engineer` | **No scanner exists in this stack.** The agent must refuse rather than answer; nothing downstream would contradict a fabricated figure |

---

## Reserved verbs

A verb in this list is spoken for. **Never add it to a new artifact's trigger phrases** without resolving the collision first.

| Verb / phrase | Owner |
|---|---|
| implement, build the feature, scaffold the component | `frontend-engineer` / `backend-engineer` |
| review my diff, anything to fix before I PR | `code-reviewer` |
| refactor | `refactor-specialist` |
| resolve the conflicts, fix the rebase | `conflict-resolver` |
| my next task, move ENG-XXX to, comment progress, open the PR | `linear-task-agent` |
| file a bug, log a follow-up, track this for later | `linear-task-agent` (and `file-followup-bug` where built) |
| apply, destroy, terraform state | **nobody** — no agent holds an apply credential by design |

Two naming patterns keep the surface small and are worth preserving:

- **The `-bringup` suffix is a routing asset.** No ordinary development utterance contains "bring up", and it makes the subject a *repository capability* rather than a *unit of code*.
- **Tracker-writing skills always name the requirements artifact** ("from the PRD", "the backlog structure") rather than a bare tracker noun. That is what keeps them clear of the story-loop agent sharing the same MCP server.

---

## Known collisions in consuming repos

Predicted, not observed — both trees below are gitignored, so these are what happens in *any* repo that already followed the framework, not a live defect here.

| New artifact | Collides with | Status |
|---|---|---|
| the three requirement skills | a project-scope `requirement-gathering` skill | Unverified — run the negative-routing check after install |
| `cicd-pipeline-bringup` | a project-scope `cicd-devops` skill | Mitigated at build: its triggers deliberately do **not** lead with "CI/CD". Any edit reintroducing "set up CI/CD for this repo" re-opens it |
| `render-design-diagrams` *(unbuilt)* | a project-scope `architecture-diagram` skill | Verbatim trigger collision — deconflict before authoring |
| `e2e-and-coverage-engineer` *(unbuilt)* | a project-scope `qa-test-engineer` agent | Renamed pre-emptively; the existing agent carries the opposite scope |

---

## Adding a new artifact

1. Write the description first. If you cannot state what it owns **and** what it must never take, it is not one artifact.
2. Grep this file's reserved-verb table for every trigger phrase you drafted.
3. Check the write-type axis: does anything already write the same object? If yes, the boundary goes in **both** descriptions, in the same PR.
4. Run the negative-routing check — ask for ordinary work in plain language and confirm the new artifact does not load.
5. Add a row here. **An artifact that ships without a row in this file has no recorded boundary**, which means the next author cannot avoid colliding with it.

> **Maintenance rule.** This file is only worth what its last update is worth. Any PR that ships, renames, or retires an artifact edits this file in the same PR — the same rule the per-phase READMEs' Wraps columns already carry.
