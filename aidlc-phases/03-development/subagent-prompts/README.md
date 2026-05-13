# Phase 3 Specialist Subagent Prompts

This directory holds the **system-prompt templates** for the Phase 3 specialist Claude Code subagents referenced in [`../PROCESS.md`](../PROCESS.md) (Step 0b → "Bundle the recurring workflow into a local Claude Code subagent" and "Specialist subagents for Phase 3 roles"). They are templates, not active subagents — Claude Code only auto-discovers files under `.claude/agents/`.

> **Skills come first.** Subagent system prompts reference the project skills committed in [PROCESS.md → Step 0a](../PROCESS.md#step-0a-one-time-setup--bootstrap-claude-code-skills) — for example, a `backend-engineer` spawn relies on the project's `scaffold-endpoint` skill to know the repo's endpoint convention, and a `code-reviewer` run relies on the `code-review-checklist` skill for the project-specific checklist. Land Step 0a's `.claude/skills/` bundle **before** committing these subagent templates so the system prompts have something to reference. If a subagent template references a skill that has not been authored yet, file the skill first; do not let dangling references ship.

## Files

| File | Role |
|------|------|
| [`linear-task-agent.md`](./linear-task-agent.md) | Sole Linear MCP writer for the dev loop — find / start / progress / PR-open / done / file-followup / update |
| [`software-architect.md`](./software-architect.md) | Pre-implementation design pass in Step 3.0 — read-only, requires developer approval before scaffolding |
| [`frontend-engineer.md`](./frontend-engineer.md) | UI implementation in Steps 3.1–3.3 |
| [`backend-engineer.md`](./backend-engineer.md) | Server-side implementation in Steps 3.1–3.3 |
| [`code-reviewer.md`](./code-reviewer.md) | Pre-PR self-review in Step 3.5 (read-only) |
| [`refactor-specialist.md`](./refactor-specialist.md) | Step 5 refactoring against `tech-debt` issues |
| [`conflict-resolver.md`](./conflict-resolver.md) | Resolves merge / rebase / cherry-pick conflicts on the working tree, completes the operation, and produces a resolution report; never `--abort`s without explicit instruction |

## How to instantiate per repo

1. Copy the chosen template into `.claude/agents/<role>.md` at the consuming repo root.
2. Replace the placeholders with the repo's values:
   - `{{TEAM_STACK}}` — e.g., `Next.js 14 + TypeScript`, `FastAPI + Python 3.12`
   - `{{COMPONENT_LIBRARY_PATH}}` — e.g., `apps/web/src/components/ui`
   - `{{TEST_RUNNER}}` — e.g., `vitest`, `pytest`
   - `{{FRONTEND_ROOT}}` / `{{BACKEND_ROOT}}` / `{{MIGRATIONS_PATH}}` / `{{SHARED_SCHEMAS_PATH}}` (software-architect)
   - `{{API_CLIENT_PATH}}` (frontend), `{{ORM}}`, `{{DATASTORE}}` (backend, software-architect)
   - `{{BACKEND_FRAMEWORK}}` — e.g., `NestJS 10`, `FastAPI`, `Django 5` (software-architect)
   - `{{FRONTEND_FRAMEWORK}}` — e.g., `React 18 + Vite + TypeScript`, `Next.js 14` (software-architect)
   - `{{REALTIME_STACK}}` — e.g., `Pusher`, `Socket.io`, `Server-Sent Events`; leave blank if not applicable (software-architect)
   - `{{ASYNC_JOB_STACK}}` — e.g., `BullMQ + Redis`, `Celery + RabbitMQ`, `Sidekiq`; leave blank if not applicable (software-architect)
   - `{{AUTH_STACK}}` — e.g., `JWT via Passport`, `NextAuth`, `Auth0` (software-architect)
   - `{{OBSERVABILITY_STACK}}` — e.g., `Datadog tracing + metrics + structured logs`, `OpenTelemetry → Grafana` (software-architect)
   - `{{SECRETS_MECHANISM}}` — e.g., `ConfigService for all secrets`, `AWS Secrets Manager`, `Doppler` (software-architect)
   - `{{TEAM_PREFIX}}` — Linear team prefix, e.g., `ENG`, `ALN` (linear-task-agent)
   - `{{LINEAR_PROJECT}}` — Linear project name or identifier the dev loop writes into (linear-task-agent)
   - `{{PR_TITLE_FORMAT}}` — PR-title convention, e.g., `[ENG-XXX] <imperative summary>` (linear-task-agent)
   - `{{LABEL_CONVENTIONS}}` — comma-separated allowed labels, e.g., `feature, bug, tech-debt, chore` (linear-task-agent)
   - `{{TECH_DEBT_LABEL}}` — exact label name for tech-debt follow-ups, typically `tech-debt` (linear-task-agent)
   - `{{BUG_LABEL}}` — exact label name for bug follow-ups, typically `bug` (linear-task-agent)
3. Adjust the `model:` frontmatter if the team's default differs (Sonnet for high-throughput coding and Linear MCP work, Opus for review/refactor).
4. Commit `.claude/agents/<role>.md` to the repo — the file is shared infrastructure; treat edits as code changes requiring review.
5. Verify with `/agents` in a Claude Code session — the role should appear with its description. For `linear-task-agent`, run the no-write smoke test from [`../PROCESS.md`](../PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp): "Use the linear-task-agent to fetch my next Todo issue from the active cycle and print its branchName — do not transition state yet." For `software-architect`, run the approval-gate smoke test: "Use the software-architect to design the architecture for ENG-XXX — present the plan and stop without recommending implementation; do not invoke any implementation specialist." A passing run produces the structured plan and ends at the approval question without handing off. For `conflict-resolver`, run the dry-explanation smoke test: "Use the conflict-resolver to describe how it would handle a hypothetical 3-file merge conflict between `feature-x` and `main` — explain its workflow and resolution rules without running any git commands." A passing run articulates the 7-step workflow and the rule priority order, names the never-`--abort`-without-instruction safety property, and does not touch the working tree.
