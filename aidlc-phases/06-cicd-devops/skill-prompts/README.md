# Phase 6 Bring-Up Skill Prompts

This directory holds the **`SKILL.md` templates** for the Phase 6 bring-up skills referenced in [`../PROCESS.md`](../PROCESS.md). They are templates, not active skills — Claude Code only auto-discovers folders under `.claude/skills/`.

Each skill encodes **one ordered bring-up procedure** ending at a quality gate. Unlike the [Phase 3 specialist subagents](../../03-development/subagent-prompts/), which a developer invokes by name, **a skill auto-triggers from its description alone.** That makes the `description:` field the entire routing surface — read [Routing](#routing) before editing one.

## Files

| Folder | Procedure | Gate | Wraps |
|--------|-----------|------|-------|
| [`iac-state-backend-bringup/`](./iac-state-backend-bringup/SKILL.md) | Versioned encrypted state store with public access denied, native locking, split read/write IAM roles, root module per environment, state out of git, audit log covers the store | Gate 1 (state + repo layout) | — (prose process steps only) |
| [`ci-identity-and-secrets-bringup/`](./ci-identity-and-secrets-bringup/SKILL.md) | OIDC trust scoped to repo + branch + environment, plan and apply roles split, every secret resolved from the secret manager at run time, static keys named for revocation, scheduled drift detection | Gate 1 (CI auth + secrets + drift) | — |
| [`cost-guardrails-bringup/`](./cost-guardrails-bringup/SKILL.md) | Infracost on every IaC PR, budget alerts at 50/80/100%, one unit-economics metric, reviewer checklist for what Infracost cannot judge | Gate 1 (cost items) | — |
| [`cicd-pipeline-bringup/`](./cicd-pipeline-bringup/SKILL.md) | Justified pipeline stages, `@claude` fix-on-mention pinned and gated, the four agentic workflows, secrets via OIDC, branch protection | Gate 2 | [`cicd-pipeline-generation`](../PROMPTS.md#cicd-pipeline-generation), [`claude-code-action--pr-review-workflow`](../PROMPTS.md#claude-code-action--pr-review-workflow) **(fix-mode half only — see note)**, [`agentic-workflow-templates`](../PROMPTS.md#agentic-workflow-templates) |
| [`observability-bringup/`](./observability-bringup/SKILL.md) | Three dashboards as code, one SLO per NFR with burn-rate alerts, alert routing, a draft runbook per alert | Gate 4 (dashboards + alerts) | [`observability--dashboard-generation`](../PROMPTS.md#observability--dashboard-generation), [`slo-and-alert-generation`](../PROMPTS.md#slo-and-alert-generation), [`runbook-generation-from-alert`](../PROMPTS.md#runbook-generation-from-alert) |
| [`deploy-and-rollback-bringup/`](./deploy-and-rollback-bringup/SKILL.md) | Staging auto-deploy, production approval gate, strategy implementation, health-gated rollout, SLO-triggered auto-rollback, deploy annotations, the drill | Gate 5 | — |

The **Wraps** column is this directory's purpose. Because shipped templates must stand alone in a consuming repo, each skill **inlines** its procedure with `{{PLACEHOLDERS}}` rather than linking back to `PROMPTS.md` — so the table above is the only record of where a procedure came from. Keep it current, or the round-trip is lost.

> **Note — `cicd-pipeline-bringup` wraps only half of one prompt.** The `claude-code-action--pr-review-workflow` prompt generates a `claude.yml` containing *both* auto-review-on-PR-open and `@claude` fix-mode. **The skill commits fix-mode only.** Whether an AI reviews every PR, and which reviewer, is a per-project decision recorded elsewhere — and the prompt's review-mode block is a bootstrap with no size gate, no re-review idempotency and no fail-open handling. Shipped by a pipeline bring-up, teams reliably mistake it for the production reviewer. The prompt stays whole for the project that deliberately wants both; the skill is where the guardrail lives.

## How to instantiate per repo

1. Copy the chosen **folder** into `.claude/skills/<skill-name>/` at the consuming repo root, preserving the folder name. **The folder name must equal the `name:` field** — Claude Code uses the folder name as the invocation slug, and a mismatch means the skill silently never loads. Use `cp -r`; globbing the Markdown files flattens the structure and produces zero working skills.
2. Replace the placeholders with the repo's values:
   - `{{TF_CLI}}` — `terraform` or `tofu`, per the project's IaC licensing decision (iac-state-backend-bringup, ci-identity-and-secrets-bringup, deploy-and-rollback-bringup)
   - `{{CLOUD_PROVIDER}}` — e.g. `AWS`, `GCP`, `Azure` (iac-state-backend-bringup, ci-identity-and-secrets-bringup, cost-guardrails-bringup)
   - `{{IAC_DIR}}` — e.g. `infra/`, `terraform/` (iac-state-backend-bringup, ci-identity-and-secrets-bringup, cost-guardrails-bringup)
   - `{{SECRETS_MANAGER}}` — e.g. `AWS Secrets Manager`, `GCP Secret Manager`, `Azure Key Vault`, `HashiCorp Vault`, `1Password`, `Doppler` (ci-identity-and-secrets-bringup)
   - `{{ALERT_CHANNEL}}` — where budget and pipeline-failure alerts land, e.g. `#platform-oncall` (cost-guardrails-bringup)
   - `{{DEPLOY_TARGET}}` — e.g. `ECS Fargate`, `Cloud Run`, `App Runner`, `Kubernetes`, `Vercel`, `Fly.io` (cicd-pipeline-bringup, deploy-and-rollback-bringup)
   - `{{TEST_COMMAND}}` — the coverage-gated test command, e.g. `pnpm test -- --coverage` (cicd-pipeline-bringup)
   - `{{CLAUDE_ACTION_VERSION}}` — an **exact** released version, e.g. `v1.0.158`, never a floating `@v1` (cicd-pipeline-bringup)
   - `{{OBS_BACKEND}}` — `Datadog` or `Grafana`, one only (observability-bringup, deploy-and-rollback-bringup)
   - `{{ALERT_ROUTER}}` — e.g. `PagerDuty`, `Opsgenie`, `incident.io` (observability-bringup)
3. **`cicd-pipeline-bringup` needs one extra paste.** Its step 4 commits four starter agentic workflows held in [`../PROMPTS.md#agentic-workflow-templates`](../PROMPTS.md#agentic-workflow-templates) rather than inlined — inlining them would put the skill body past the two-screen limit. Copy those four Markdown files into `.github/agentic-workflows/` during instantiation, adjusting only cron, limits and project phrasing.
4. Commit `.claude/skills/<skill-name>/SKILL.md` to the repo — skills at project scope are shared team infrastructure; treat edits as code changes requiring review.
5. **Verify. `/agents` does not list skills** — skills are a different surface and are verified differently. Run all four checks:
   - **Discovery.** Open a Claude Code session; the skill appears in the available-skills list under its slug. If not, the folder name and the `name:` field are out of alignment.
   - **Explicit invocation.** Type `/<skill-name>` with a representative input. The procedure should run top-to-bottom and stop at its gate.
   - **Auto-trigger, in messy language.** This is the check that matters, because the description is the whole routing surface:
     - `iac-state-backend-bringup` — *"we've got terraform in here but no backend configured, can you sort that out"*
     - `ci-identity-and-secrets-bringup` — *"our github actions are using an AWS key from 2022, can we stop doing that"*
     - `cost-guardrails-bringup` — *"we got a surprise bill last month, can we get warned earlier"*
     - `cicd-pipeline-bringup` — *"this repo has no CI at all, can you get something running"*
     - `observability-bringup` — *"this service is about to go live and we can't see anything"*
     - `deploy-and-rollback-bringup` — *"we've never actually tried rolling back, should we test it"*
   - **Refusal.** Drive each skill into its sharpest refusal and confirm it redirects rather than complies:
     - `iac-state-backend-bringup` — *"just use one IAM role for everything, we'll tighten it later"*
     - `ci-identity-and-secrets-bringup` — *"just put the AWS keys in GitHub secrets for now"*
     - `cost-guardrails-bringup` — *"just estimate what this month will cost"*
     - `cicd-pipeline-bringup` — *"make the Claude review a required check so people can't ignore it"*
     - `observability-bringup` — *"just guess the metric names, we'll fix them later"*
     - `deploy-and-rollback-bringup` — *"go ahead and deploy to prod to test the gate"*
6. **Negative-routing check, once, after installing more than one.** Ask for ordinary work these skills must not claim — *"implement the checkout endpoint"*, *"review my diff before I PR"*, *"resolve these merge conflicts"* — and confirm none of the six loads. If one does, its description has stolen a verb from a specialist subagent; narrow it and re-test.

## Routing

Skills auto-trigger; subagents do not. A skill whose trigger phrases reuse a verb one of the specialist subagents already claims **wins that route by default and hijacks ordinary work.** Two consequences when editing anything in this directory:

- **Never add a trigger phrase containing a bare development verb** — implement, build, write the tests, review the diff, refactor, resolve conflicts, open the PR, ship. Those belong to the specialists.
- **The six skills must exclude each other by name.** They share a surface (`.github/workflows/`, cloud identity, cost, deploys) and the boundary between them is *which files each owns*, not which topic each covers:

| Skill | Owns | Explicitly does not own |
|---|---|---|
| `iac-state-backend-bringup` | the `backend` block, the state store, the lock, the state IAM roles, `.gitignore` state entries | resource HCL, anything in `.github/workflows/`, CI credentials |
| `ci-identity-and-secrets-bringup` | the OIDC provider and trust policy, secret-manager wiring, the drift workflow | the state store itself, build workflows, the Infracost workflow, resource HCL |
| `cost-guardrails-bringup` | the Infracost workflow, budget alerts, the unit-economics metric | the cost estimate itself, any other workflow |
| `cicd-pipeline-bringup` | build/test/deploy-to-staging workflows, `@claude`, agentic workflows, branch protection | the drift workflow, the Infracost workflow, the production gate |
| `observability-bringup` | dashboards, SLOs, alerts, alert runbooks | instrumentation code, service runbooks, incident work |
| `deploy-and-rollback-bringup` | the production gate, strategy, rollback trigger, the drill | the build pipeline, the SLO definition |

Two notes on why the collision surface is small, worth preserving as the set grows. The **`-bringup` suffix is a routing asset**, not just a naming convention — no ordinary development utterance contains "bring up", and it makes each description's subject a *repository capability* rather than a *unit of code*. And `cicd-pipeline-bringup` deliberately **does not lead with the bare phrase "CI/CD"**, which is what keeps it from colliding with project-scope skills of that name in consuming repos; any future edit that reintroduces "set up CI/CD for this repo" as a trigger re-opens that collision.
