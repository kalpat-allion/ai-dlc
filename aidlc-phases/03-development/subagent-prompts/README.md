# Phase 3 Specialist Subagent Prompts

This directory holds the **system-prompt templates** for the Phase 3 specialist Claude Code subagents referenced in [`../PROCESS.md`](../PROCESS.md) (Step 0 → "Bundle the recurring workflow into a local Claude Code subagent" and "Specialist subagents for Phase 3 roles"). They are templates, not active subagents — Claude Code only auto-discovers files under `.claude/agents/`.

## Files

| File | Role |
|------|------|
| [`linear-task-agent.md`](./linear-task-agent.md) | Sole Linear MCP writer for the dev loop — find / start / progress / PR-open / done / file-followup / update |
| [`frontend-engineer.md`](./frontend-engineer.md) | UI implementation in Steps 3.1–3.3 |
| [`backend-engineer.md`](./backend-engineer.md) | Server-side implementation in Steps 3.1–3.3 |
| [`code-reviewer.md`](./code-reviewer.md) | Pre-PR self-review in Step 3.5 (read-only) |
| [`refactor-specialist.md`](./refactor-specialist.md) | Step 5 refactoring against `tech-debt` issues |

## How to instantiate per repo

1. Copy the chosen template into `.claude/agents/<role>.md` at the consuming repo root.
2. Replace the placeholders with the repo's values:
   - `{{TEAM_STACK}}` — e.g., `Next.js 14 + TypeScript`, `FastAPI + Python 3.12`
   - `{{COMPONENT_LIBRARY_PATH}}` — e.g., `apps/web/src/components/ui`
   - `{{TEST_RUNNER}}` — e.g., `vitest`, `pytest`
   - `{{FRONTEND_ROOT}}` / `{{BACKEND_ROOT}}` / `{{MIGRATIONS_PATH}}`
   - `{{API_CLIENT_PATH}}` (frontend), `{{ORM}}`, `{{DATASTORE}}` (backend)
   - `{{TEAM_PREFIX}}` — Linear team prefix, e.g., `ENG`, `ALN` (linear-task-agent)
   - `{{LINEAR_PROJECT}}` — Linear project name or identifier the dev loop writes into (linear-task-agent)
   - `{{PR_TITLE_FORMAT}}` — PR-title convention, e.g., `[ENG-XXX] <imperative summary>` (linear-task-agent)
   - `{{LABEL_CONVENTIONS}}` — comma-separated allowed labels, e.g., `feature, bug, tech-debt, chore` (linear-task-agent)
   - `{{TECH_DEBT_LABEL}}` — exact label name for tech-debt follow-ups, typically `tech-debt` (linear-task-agent)
   - `{{BUG_LABEL}}` — exact label name for bug follow-ups, typically `bug` (linear-task-agent)
3. Adjust the `model:` frontmatter if the team's default differs (Sonnet for high-throughput coding and Linear MCP work, Opus for review/refactor).
4. Commit `.claude/agents/<role>.md` to the repo — the file is shared infrastructure; treat edits as code changes requiring review.
5. Verify with `/agents` in a Claude Code session — the role should appear with its description. For `linear-task-agent`, run the no-write smoke test from [`../PROCESS.md`](../PROCESS.md#step-0-one-time-setup--connect-claude-to-linear-via-mcp): "Use the linear-task-agent to fetch my next Todo issue from the active cycle and print its branchName — do not transition state yet."
