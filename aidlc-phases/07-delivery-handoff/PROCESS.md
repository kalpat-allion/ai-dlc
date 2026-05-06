# Phase 7: Delivery & Handoff — Process Definition

## Overview

This document defines the AI-assisted workflow for the Delivery & Handoff phase of the AI-DLC. It is the phase where the engagement crosses from "we built it" to "the recipient runs it" — every artefact, secret, dashboard, runbook, and agentic workflow that was set up across Phases 1–6 has to be transferred so the recipient inherits the full AI-driven loop, not a skeleton repo. The process below is built around **semantic-release + the Anthropic Claude Code Action** for release notes, **Mintlify (or GitBook / Docusaurus + AI)** as the documentation platform driven by its hosted MCP server, **Claude Code with the Atlassian Remote MCP / Notion MCP** for handoff-document synthesis and knowledge-base seeding, **Fathom (or Loom AI)** for KT recording with structured chapter and action-item extraction, and **1Password + Pulumi ESC** for credential and infrastructure handoff that preserves the Phase 6 secrets posture. The handoff is **not** complete when the docs are written — it is complete when the recipient's Claude Code can run `@claude` reviews, query Sentry MCP, query Pulumi MCP, and operate the same agentic loop the delivery team used.

**Phase Duration:** 1–2 weeks (dedicated handoff period) + 30-day support window + 90-day debrief
**Phase Owner:** Delivery Lead / Tech Lead
**Tools Used:**

- **Releases:** semantic-release + Conventional Commits + Anthropic Claude Code Action (`anthropics/claude-code-action@v1`) + GitHub MCP server
- **Docs platform (primary):** Mintlify (auto-hosted MCP server at `/mcp`, AI-native search, llms.txt + skill.md auto-generation) — alternates: GitBook (auto-hosted MCP at `/~gitbook/mcp`) or Docusaurus (OSS, paired with Claude Code over the repo)
- **Doc generation:** Claude Code (multi-source ingestion via `--add-dir`, MCP-driven completeness checks)
- **API reference:** auto-generated from the OpenAPI spec produced in Phase 2 (Mintlify, docusaurus-plugin-openapi-docs, Redoc, or Scalar)
- **Handoff document synthesis:** Claude Code with multi-repo `--add-dir` ingestion + Atlassian Remote MCP / Notion MCP for cross-platform context
- **KT recording:** Fathom (recommended for action-item extraction quality) or Loom AI (recommended when screen-recording is the dominant format)
- **KT synthesis:** NotebookLM Enterprise (recommended for multi-source post-session ingestion) + Claude Code summary prompt
- **Credentials:** 1Password Business + 1Password CLI (`op run`) + Pulumi ESC handoff (transfers the Phase 6 secret-store posture intact)
- **Infrastructure handoff:** Pulumi Cloud (state + ESC environments) + GitHub repo ownership transfer + MCP-server access transfer (Pulumi MCP, observability MCP, Sentry MCP)
- **Knowledge base:** Atlassian Confluence (Atlassian Remote MCP, GA Feb 2026) or Notion (`mcp.notion.com/mcp`) — both seeded by Claude Code; Glean as the cross-platform discovery overlay when the org already has it
- **Post-handoff:** Sentry + Seer + Sentry MCP for inherited error backlog; incident.io or PagerDuty AIOps inherits the Phase 6 on-call rotation; Linear MCP carries over from Phase 3 for the support-period bug pipeline

> **Tool Philosophy.** Handoff is not a documentation exercise — it is a **transfer of an agentic operating model**. The recipient does not just inherit code and dashboards; they inherit the developers' Claude Code subagents, the `AGENTS.md`, the MCP server roster, the `@claude` PR-review workflow, and the secrets posture. AI accelerates every traditional handoff artefact (release notes, docs, handoff document, KT summaries, KB seeding) — but the larger win is that the recipient's first day looks identical to the delivery team's last day, because the same MCP servers and subagents are wired into their Claude Code. Five tool families, zero overlap, every credential transfer requires a human approval gate, every doc Claude generates is reviewed before it ships.

---

## Tool Stack

| Layer | Primary | Fallback | Cost |
|-------|---------|----------|------|
| **Versioning** | semantic-release + Conventional Commits | Manual semver + commitlint | Free OSS |
| **Release notes generation** | semantic-release raw → Anthropic Claude Code Action polish (auto on tag) | Claude Code one-shot via terminal | API usage-based; semantic-release free |
| **Release publish** | GitHub Releases via semantic-release plugin + GitHub MCP for review | GitLab Release CLI | Free |
| **Docs platform (customer-facing)** | Mintlify (MCP at `/mcp`, AI agent, auto llms.txt + skill.md) | GitBook (MCP at `/~gitbook/mcp`) or Docusaurus (OSS) | Mintlify Pro $300/mo; GitBook Pro $215/mo for 10 seats; Docusaurus free |
| **Docs platform (internal/dev)** | Docusaurus + Claude Code over the repo | Mintlify with auth-gated MCP at `/authed/mcp` | Docusaurus free |
| **API reference** | Mintlify OpenAPI integration / `docusaurus-plugin-openapi-docs` | Redoc, Scalar, Bump.sh | Free for OSS plugins |
| **Doc gen agent** | Claude Code with docs-platform MCP + repo `--add-dir` | Cursor over the docs repo | Already in Phase 3 stack |
| **Handoff doc synthesis** | Claude Code with `--add-dir` across PRD repo, code repo, IaC repo, docs site MCP | Claude Projects (claude.ai) with bulk file upload | API usage-based |
| **KT recording (meetings)** | Fathom (best action-item extraction; 9.4 vs Loom 7.2 in 2026 G2) | Otter, Granola, Jamie | Fathom Free covers most; Premium $19/mo |
| **KT recording (screen)** | Loom AI (auto chapters + auto CTA + summaries) | Fathom screen-recorder mode | Loom Business $15/user/mo |
| **KT post-processing** | NotebookLM Enterprise (multi-source ingest, 300 sources / 500K words each) | Claude Code with transcript paste | NotebookLM Enterprise via Gemini Enterprise |
| **Credential vault** | 1Password Business + 1Password CLI + Unified Access (Mar 2026 GA) | Bitwarden Teams | 1Password Business $7.99/user/mo |
| **CI secrets handoff** | Pulumi ESC environment ownership transfer (carries over from Phase 6) | GitHub Actions secrets re-export | Bundled with Pulumi Cloud Team+ |
| **MCP-credential injection** | `op run -- claude` pattern (no plaintext secrets in `mcp.json`) | Direct env vars (off-board risk) | Free with 1Password |
| **IaC handoff** | Pulumi Cloud workspace transfer (org → org) + Pulumi MCP access regrant | Terraform state migration | Free |
| **Knowledge base** | Atlassian Confluence (Remote MCP, 72+ tools, GA Feb 2026) or Notion (`mcp.notion.com/mcp`) | Docusaurus (if no separate KB) | Confluence Standard $6.05/user/mo; Notion Business $15/user/mo |
| **Cross-platform KB search** | Glean (when org already runs it; permissions-aware retrieval over Slack/Jira/Drive/Confluence) | Confluence/Notion native search | Enterprise pricing |
| **Inherited error triage** | Sentry + Seer + Sentry MCP (`mcp.sentry.dev/mcp`) | Bugsnag | Sentry Team from $26/mo |
| **Inherited on-call** | incident.io (Scribe transcribes incident calls) or PagerDuty AIOps | Grafana OnCall (archived 2026 — migrate) | incident.io from $20/responder/mo |

**Optional upgrades:**
- **Mintlify Enterprise** — when SSO + audit logs + custom domain at scale matter (~$5K+/yr) and the docs are a product surface in their own right.
- **NotebookLM Enterprise** — when KT corpora exceed Claude Code's context comfortably (multi-hour recordings + entire repos + slide decks); the 300-source / 500K-word-per-source ceiling absorbs almost any handoff payload.
- **Glean** — when the recipient already operates a multi-tool stack (Slack + Jira + Drive + Confluence + Linear) and wants permissions-aware AI search across all of it without writing a custom retrieval layer.

---

## Process Steps

### Step 0: One-Time Setup — Wire AI Tools into the Handoff Loop

> Visual: [Step 0 flowchart](./FLOWCHART.md#step-0-one-time-setup)

| Attribute | Detail |
|-----------|--------|
| **Input** | Phase 3 Linear MCP setup (carries over), Phase 6 Pulumi/observability MCP setup (carries over), GitHub org access, chosen docs-platform tenancy (Mintlify / GitBook / Docusaurus), chosen KB tenancy (Confluence / Notion), 1Password Business workspace, Anthropic API key already provisioned in Phase 6 |
| **Tools** | **Mintlify MCP** (or **GitBook MCP**), **Atlassian Remote MCP** (or **Notion MCP**), **GitHub MCP**, **Anthropic Claude Code Action**, **1Password CLI**, **AGENTS.md** extension |
| **Output** | Every delivery-team developer's Claude Code can read the docs site, the KB, and GitHub releases via MCP; release-note polish runs automatically on every tag; AGENTS.md carries handoff-specific context; a handoff subagent (optional) is committed under `.claude/agents/` for the recurring synthesis workflow |
| **Human** | Delivery Lead authorises org-level OAuth grants on the docs/KB/credential platforms; each developer authenticates once; recipient counterparts authorise on their side at handoff time |

This step is done **once per engagement**. Skipping it means hand-authoring release notes, hand-synthesising the handoff document, and chasing every dashboard URL by Slack — the same "AI tax" Phase 3 and Phase 6 already eliminated for development and ops.

#### 0.1 — Connect Claude Code to the docs-platform MCP

The docs platform owns the customer-facing surface. The team chose one in Step 2 of the engagement (or Phase 6 if the docs site shipped early). The MCP server lets Claude Code generate, refine, and completeness-check pages **against the live site** — no copy-paste of JSON or YAML.

Pick the path that matches the chosen platform:

```bash
# Path A — Mintlify (recommended for customer-facing API + product docs)
# Public docs site:
claude mcp add --transport http --scope user mintlify-docs https://<your-docs>.mintlify.app/mcp
# Authenticated docs site (private/internal):
claude mcp add --transport http --scope user mintlify-docs https://<your-docs>.mintlify.app/authed/mcp

# Path B — GitBook (recommended when content authors are non-technical)
claude mcp add --transport http --scope user gitbook-docs https://<your-site>.gitbook.io/docs/~gitbook/mcp

# Path C — Docusaurus (OSS, no hosted MCP — pair Claude Code with the docs repo directly)
# No MCP needed. Claude Code reads the docs repo via --add-dir, edits Markdown, commits.
```

After OAuth (Mintlify and GitBook both support MCP authorization spec + Dynamic Client Registration as of 2026), verify with `claude mcp list` — `mintlify-docs: connected` (or `gitbook-docs: connected`) must appear. Smoke test: `Via the docs MCP, search for "rate limit" and return the top 3 page titles with URLs.` Claude Code should return real pages from the live site.

> **Anti-pattern:** Don't auto-pick Mintlify just because it has the most polished AI story. Mintlify is the right call when the docs are a **product surface** (developer-facing API docs, public reference) and polish is a competitive feature. For internal handoff docs that primarily live alongside code, Docusaurus + Claude Code over the repo is faster, cheaper, and avoids a per-engagement SaaS bill.

#### 0.2 — Connect Claude Code to the knowledge-base MCP

The KB is where long-form troubleshooting, glossary, FAQ, onboarding, and architectural context live for the recipient — typically Atlassian Confluence (most enterprise) or Notion (most startup). Both have official remote MCP servers as of 2026.

```bash
# Path A — Atlassian Confluence + Jira (Atlassian Rovo MCP, GA Feb 2026, 72+ tools)
claude mcp add --transport http --scope user atlassian https://mcp.atlassian.com/v1/sse
# Note: the v1/sse endpoint is deprecated after 30 June 2026 — migrate to the new endpoint
# documented at https://support.atlassian.com/atlassian-rovo-mcp-server/ before that date.

# Path B — Notion (official, makenotion/notion-mcp-server)
claude mcp add --transport http --scope user notion https://mcp.notion.com/mcp
# Or use the bundled Claude Code Notion plugin which ships MCP + Skills + slash commands together:
# https://github.com/makenotion/claude-code-notion-plugin
```

Smoke test for Confluence: `Via Atlassian MCP, search Confluence space "PROJECT" for pages tagged "runbook" and list the top 5 with last-modified dates.` Smoke test for Notion: `Via Notion MCP, search the workspace for pages with "onboarding" in the title and return the page IDs and parent databases.`

#### 0.3 — Connect Claude Code to the GitHub MCP for release operations

Release management runs through GitHub Releases and the repo's tags. The GitHub MCP server (v1.0.2 as of April 2026) gives Claude Code direct access to draft releases, post comments on the release PR, attach artefacts, and inspect the merged-PR list since the last tag.

```bash
claude mcp add --transport http --scope user github https://api.githubcopilot.com/mcp/
# Authenticate via GitHub OAuth in the browser when /mcp prompts you in Claude Code.
```

Smoke test: `Via GitHub MCP, list the merged PRs in <org/repo> since the v2.4.0 tag and group them by Conventional Commits type (feat / fix / chore / docs / breaking).`

#### 0.4 — Install the Anthropic Claude Code Action for auto release-note polish

Phase 6 already committed `.github/workflows/claude.yml` for `@claude` PR review. Phase 7 adds a second workflow that triggers on every tag push and runs the [`Release Notes Polish`](./PROMPTS.md#release-notes-polish) prompt against the semantic-release output, then posts the polished draft as a comment on the release PR (or as a draft GitHub Release) for the Release Manager to merge.

The starter workflow lives under `.github/workflows/release-notes-polish.yml` and triggers on `release: created` or `push: tags: ['v*']`. It uses the same `anthropics/claude-code-action@v1` and `ANTHROPIC_API_KEY` (or Bedrock/Vertex/Foundry equivalents) the team already provisioned in Phase 6 Step 0.2 — no new secrets needed.

> **Anti-pattern:** Do **not** auto-publish the polished release notes. Always gate publish on a human merge or "publish release" click — release notes are the customer-visible voice of the engagement and Claude can be confidently wrong about which feature is the star.

#### 0.5 — Wire 1Password CLI for credential-handoff inventory generation

1Password's 2026 Unified Access platform (March 2026 GA) is the canonical place to discover, secure, and audit every credential the engagement uses. The 1Password CLI (`op`) lets Claude Code list vault items, generate inventory tables, and inject secrets at runtime — without putting plaintext secrets into `mcp.json`, `settings.json`, or transcript history.

Install the CLI and sign in once per developer:

```bash
brew install --cask 1password-cli   # macOS
# Windows: winget install AgileBits.1Password.CLI
op signin                           # interactive OAuth sign-in to your 1Password account
op whoami                           # verify the active account
```

At handoff time, generate the inventory by running Claude Code with `op run --` so the CLI brokers vault access:

```bash
op run -- claude    # opens Claude Code with op-resolved env; ask Claude to:
# "List all items in the 'Engagement Handoff' shared vault and produce a markdown
#  inventory grouped by service, including the field names but never the secret values."
```

> **Critical:** the secrets themselves never leave the vault — Claude Code receives item names and field names (`username`, `api_token`, `db_url`) so the inventory describes *what* the recipient will inherit without ever transcribing values. Plaintext secrets in a Claude Code transcript are a leak class we don't tolerate.

#### 0.6 — Extend `AGENTS.md` with handoff-specific context

The `AGENTS.md` already committed in Phase 6 covers stack naming, region defaults, mandatory tags, ADR pointers, and production-deploy approval. Phase 7 adds a `## Delivery & Handoff` section that captures the conventions every Claude Code session in this phase should respect:

- **Release-note tone** — "professional / friendly / technical / marketing"; matches the existing CHANGELOG voice. Claude defaults to the wrong register without this.
- **Customer-facing vs internal split** — what content goes to the public docs site vs the internal KB. Without this, Claude leaks internal-only information into customer docs.
- **Glossary location** — the canonical URL or file path for domain terms, so Claude resolves jargon consistently across release notes, docs, and KB.
- **Support-period defaults** — the agreed SLA (S0/S1/S2 response times), the escalation path, the support-period length. Drives the language Claude uses in the handoff document and the support agreement.
- **Recipient context** — who they are, their tech stack, their on-call posture, what they already have in place (e.g., "recipient already runs Confluence; do not propose Notion"). Stops Claude from recommending tools the recipient already has under a different name.
- **Forbidden content** — internal post-mortems, individual performance feedback, raw incident channel transcripts. Claude should never lift these into a customer artefact.

The Phase 6 entry pointer (`pulumi.yaml` → AGENTS.md) carries over; no new wiring needed.

#### 0.7 — (Recommended) Bundle the recurring handoff workflow into a Claude Code subagent

The handoff document, KT summaries, KB seeding, and post-handoff retrospective all share the same multi-step shape: ingest a heterogeneous bundle of artefacts (PRD, ADRs, code, runbooks, transcripts, dashboards), respect the AGENTS.md split, and emit a structured Markdown deliverable. This is exactly the shape a Claude Code subagent solves — same pattern Phase 3 uses for `linear-task-agent.md` and the four specialist subagents.

Recommend committing a single subagent at `.claude/agents/handoff-agent.md`. **Do not create the file in this task** — the team commits it when they are ready and aligned on the role's boundaries. The expected starter shape:

```yaml
---
name: "handoff-agent"
description: "Use for any handoff synthesis: handoff-document generation from Phase 1-6 artefacts, KT-session summary from a Fathom/Loom transcript, KB seeding (FAQ/glossary/troubleshooting) from PRD + Linear bug history, post-handoff retrospective. Embeds PROMPTS.md handoff-document, knowledge-base-seeding, kt-session-script, and post-handoff-retrospective prompts."
model: opus
---
```

Operating boundaries the system-prompt body should encode:

- **Read-only across all source artefacts** (PRD repo, code repo, IaC repo, docs site MCP, KB MCP, Linear MCP). Writes only the deliverable Markdown into `/handoff/` or the explicit destination the developer names.
- **Must consult `AGENTS.md` Delivery & Handoff section** before generating any customer-facing content; flag and stop when an artefact would breach the customer-facing/internal split.
- **Must cite sources inline** — every claim in the handoff document carries a link to the PRD section, ADR, runbook, or commit it came from. No source = the claim is removed.
- **Must never call `update_issue`, post Linear comments, transition state, push to GitHub, or publish to the docs site** — those writes belong to humans (or to `linear-task-agent` for the Phase 3-style flows that may still run during the support period).
- **Must escalate** when an artefact is missing or contradictory — surface the gap rather than fabricating bridging content.

Verify with `/agents` once committed; smoke-test by asking the subagent to dry-run a handoff document over a small subset of artefacts.

#### 0.8 — Verification checklist

- [ ] `claude mcp list` shows `mintlify-docs` (or `gitbook-docs`), `atlassian` (or `notion`), `github`, plus the carried-over `pulumi`, `sentry`, `grafana`/`datadog`, `linear` from Phases 3/6 — all `connected` for every developer
- [ ] `.github/workflows/release-notes-polish.yml` committed; a draft tag triggers the action and posts a polished draft within ~3 minutes
- [ ] `op signin` works for every delivery-team developer; `op run -- claude` resolves vault references without plaintext leakage
- [ ] `AGENTS.md` updated with the Delivery & Handoff section (release-note tone, customer-vs-internal split, glossary URL, support-period defaults, recipient context, forbidden content)
- [ ] Audit log on in the docs platform (Mintlify org settings or GitBook organization audit) and the KB (Confluence audit log or Notion workspace activity)
- [ ] (If adopted) `.claude/agents/handoff-agent.md` committed; `/agents` lists `handoff-agent`; subagent smoke test produces a handoff-document outline without writing any state
- [ ] Recipient counterparts have been told which MCP servers they will inherit and have requested matching OAuth grants on their side (so the day-1 transition is live, not deferred)

> **Permission inheritance.** Every MCP-driven action runs as the connecting developer — the docs MCP, KB MCP, and 1Password CLI cannot escalate beyond the connecting account's scopes. Off-board by revoking the OAuth grants in Mintlify/GitBook/Notion/Atlassian/1Password; the agent loses access automatically.

---

### Step 1: Release Management — Auto-Generated Release Notes

> Visual: [Step 1 flowchart](./FLOWCHART.md#step-1-release-management--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | Commits since the last release (Conventional Commits enforced from Phase 3), merged-PR titles and bodies, Linear issue identifiers per PR, the previous CHANGELOG entry as voice reference |
| **Tools** | **semantic-release** (auto-tag + initial markdown) + **Anthropic Claude Code Action** (auto-polish on tag) + **GitHub MCP** (review and comment on the release PR) |
| **Output** | Polished release notes posted as a draft GitHub Release + CHANGELOG.md entry; tag created; stakeholder notification draft ready for human send |
| **Human** | Release Manager reviews polished draft (~15 min), tweaks tone, highlights the star feature, clicks "publish" |

**Workflow:**

**1.1 — Conventional Commits is the floor (carries over from Phase 3).** Every commit follows the format (`feat(auth): add SSO support`, `fix(api): handle null user in /profile`, `BREAKING CHANGE: removed deprecated v1 endpoints`) and is enforced by the commitlint pre-commit hook + CI check. Without this, semantic-release cannot infer the version bump and Claude cannot infer change semantics from the commit list.

**1.2 — Push the release tag.** Either trigger semantic-release manually on the `release` branch or let it run on every merge to `main` (configurable). semantic-release: (a) analyses commit types since the last tag, (b) determines the next semver bump, (c) creates the git tag, (d) generates the initial Markdown changelog from commits, (e) creates a GitHub Release in draft state, (f) writes/updates `CHANGELOG.md` and pushes it.

**1.3 — Auto-polish via the Claude Code Action.** The `release-notes-polish.yml` workflow committed in Step 0.4 fires on the new tag. The action runs the [`Release Notes Polish`](./PROMPTS.md#release-notes-polish) prompt against the raw semantic-release output, the previous CHANGELOG entry (voice reference), and `AGENTS.md` (tone + customer-facing-vs-internal split). It posts the polished draft as a comment on the release PR (or directly into the draft GitHub Release body via GitHub MCP). The polish converts merge-PR titles ("merge PR #1847") into user-friendly bullets ("Added SSO support for enterprise accounts"), strips internal-only changes (`chore`, `refactor`, `test`), promotes the star feature to the top, and renders breaking changes prominently with migration guidance.

**1.4 — Release Manager review (~15 min).** The Release Manager opens the draft Release, reads the polished draft, and either accepts it, edits the star-feature pick, or asks Claude to redo specific sections inline (`@claude please rewrite the auth section in a more conservative tone — these are enterprise customers`). The Claude Code Action treats `@claude` mentions on the release PR the same way it treats them on a code PR (carry-over from Phase 6 Step 4.2).

**1.5 — Publish.** Release Manager clicks "Publish release" on GitHub. semantic-release's GitHub plugin: marks the release as latest, attaches build artefacts if configured, and updates `CHANGELOG.md` on `main`. If a public changelog page is part of the docs site (Mintlify and GitBook both support a CHANGELOG section), the docs build picks up the new entry on the next deploy.

**1.6 — Notify stakeholders.** Run the [`Release Notes Polish`](./PROMPTS.md#release-notes-polish) prompt a second time with `Audience: customers via email` to derive a shorter customer-email version from the published notes. Send via the team's customer-comms channel (Mailchimp / Customer.io / SendGrid). For internal stakeholders post a Slack/Teams message linking the GitHub Release.

> **Anti-pattern.** Don't conflate "AI polished" with "ready to publish." Every release goes through human eyes — the polished draft is the floor, not the ceiling. Release notes are also where AI confidently wrong tone (over-promising on incomplete features, under-emphasising breaking changes) does the most reputational damage.

> **Gate 1 — Release Automation Operational** must pass before treating releases as production-ready. See [QUALITY-GATES.md → Gate 1](./QUALITY-GATES.md#gate-1-release-automation-operational).

---

### Step 2: Documentation Finalisation

> Visual: [Step 2 flowchart](./FLOWCHART.md#step-2-documentation-finalisation--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | Codebase + OpenAPI spec (Phase 2/3) + ADRs + runbooks (Phase 6) + observability dashboards + AGENTS.md |
| **Tools** | **Mintlify** (or **GitBook** / **Docusaurus**) with the docs MCP from Step 0.1 + **Claude Code** (page generation, completeness check) + **OpenAPI auto-render** (Mintlify native or `docusaurus-plugin-openapi-docs`) |
| **Output** | Complete published documentation site with API reference, getting-started, architecture, ops, troubleshooting, glossary |
| **Human** | Tech Lead validates accuracy of every AI-generated page before publish |

**Workflow:**

**2.1 — Confirm the docs platform decision (one ADR, written once).** If the team committed to Mintlify in Phase 6 (or earlier), this step is already done. Otherwise: pick Mintlify when the docs are a customer-facing product surface and polish matters; pick GitBook when content authors are non-technical and live editing matters more than docs-as-code; pick Docusaurus when OSS / self-hosting / git-only is mandatory. Document the decision as an ADR.

**2.2 — Define the structure (Tech Lead).** Eight standard sections, in this order:

1. **Getting Started** — installation, quick start, first-run tutorial.
2. **User Guides** — how to use the product (tutorials, how-tos).
3. **API Reference** — auto-generated from the OpenAPI spec.
4. **Architecture** — overview + ADRs from Phase 2.
5. **Operations Runbook** — deployment, monitoring, incident response (from Phase 6).
6. **Developer Setup** — how to contribute, local dev environment.
7. **Troubleshooting** — FAQ, known issues, diagnostics.
8. **Glossary** — domain terms.

**2.3 — Generate page content with Claude Code.** For each section, run the [`Documentation Generation`](./PROMPTS.md#documentation-generation-per-page) prompt, with these inputs depending on the page:

```bash
# Example invocations from the repo root
claude --add-dir ../iac-repo --add-dir ../docs-repo
> Use the docs MCP (mintlify-docs) and the connected repos to generate the
>   "Local Development Setup" page from the README, the package.json scripts,
>   and the docker-compose.yml. Voice: imperative second-person. Output Markdown
>   ready to commit to /docs/getting-started/local-dev.md. Cite source files inline.
```

Claude Code reads the live docs site via the MCP server (so it picks up the existing tone and cross-link patterns) and writes new pages into the repo. **Always pair `Claude generates → human reviews`**; never publish without review. AI hallucinates domain specifics — repo names, environment variable defaults, release versions — confidently and consistently.

**2.4 — Auto-generate the API reference.** Two paths depending on the platform:

- **Mintlify:** add the OpenAPI spec to `mint.json` under `openapi`. Mintlify renders interactive API docs automatically and serves `llms.txt` + `llms-full.txt` + `skill.md` at the root for AI consumption. No manual API doc maintenance.
- **Docusaurus:** install `docusaurus-plugin-openapi-docs` (or `redocusaurus` for a Redoc-rendered version). The plugin reads `/docs/api/openapi.yaml` from Phase 2 and renders interactive reference at `/docs/api`. CI rebuilds on every spec change.

> **Rule.** **Never maintain hand-written API docs alongside the spec.** They drift within two sprints. The spec is the source of truth; docs render from it.

**2.5 — Run the completeness check via Claude Code + docs MCP.** Run the [`Doc Completeness Check`](./PROMPTS.md#doc-completeness-check) prompt. Claude reads the live docs site through the MCP server, cross-references the codebase via `--add-dir`, and emits a gap list ranked by severity (Critical / High / Medium / Low). Each gap row carries: missing item, priority, rationale, suggested source (ADR / code / Linear).

For Mintlify specifically, the auto-generated `llms-full.txt` at the docs root is a useful shortcut — Claude Code can ingest the entire docs site via `WebFetch https://<your-docs>.mintlify.app/llms-full.txt` and check coverage against the codebase in a single prompt.

**2.6 — Resolve gaps; loop back to 2.3.** Critical and High gaps are fixed before publish. Medium and Low can be queued as follow-up Linear issues with the `docs-debt` label so they are tracked, not forgotten.

**2.7 — Publish via CI.** The docs deploy pipeline runs on every merge to `main`. Mintlify deploys on git push; GitBook syncs from the connected GitHub branch; Docusaurus builds via GitHub Pages / Vercel / Netlify on merge. **Documentation stays in sync with code automatically** — that is the whole point of docs-as-code, and the reason the docs platform belongs in the same repo as the code (or in a tightly mirrored docs repo).

> **Gate 2 — Documentation Complete** must pass before the handoff meeting. See [QUALITY-GATES.md → Gate 2](./QUALITY-GATES.md#gate-2-documentation-complete).

---

### Step 3: Handoff Document Generation

> Visual: [Step 3 flowchart](./FLOWCHART.md#step-3-handoff-document-generation--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | PRD (Phase 1) + architecture & ADRs (Phase 2) + codebase structure + test coverage report (Phase 4) + security posture (Phase 5) + IaC + runbooks + observability dashboards (Phase 6) + release notes (Step 1) + the recipient's profile from `AGENTS.md` |
| **Tools** | **Claude Code** with `--add-dir` across multiple repos + **docs MCP** (for cross-linking) + **Linear MCP** (for known-issue extraction) + **Sentry MCP** (for known-error extraction) + (optional) **handoff-agent** subagent from Step 0.7 |
| **Output** | A comprehensive handoff document at `/handoff/handoff-document.md`, reviewed by Delivery Lead + Tech Lead, shared with the recipient for feedback |
| **Human** | Delivery Lead + Tech Lead review the synthesised draft for accuracy, fact-check across artefacts, edit for tone |

**Workflow:**

**3.1 — Gather inputs in one Claude Code session.** From the repo root, open Claude Code with multi-repo ingestion:

```bash
claude --add-dir ../prd-repo --add-dir ../iac-repo --add-dir ../docs-repo
```

The `--add-dir` flag lets Claude Code read across the engagement's separate repos in a single session — this is the equivalent of a multi-source Claude Project but kept in the developer's own session. For very large corpora (recordings, slide decks, multi-hour transcripts) consider exporting to **NotebookLM Enterprise** instead — its 300-source / 500K-words-per-source ceiling absorbs almost any handoff payload.

**3.2 — Run the handoff-document synthesis.** Either invoke the [`Handoff Document Generation`](./PROMPTS.md#handoff-document-generation) prompt directly, or invoke the `handoff-agent` subagent from Step 0.7 if the team committed it. The subagent encapsulates the multi-step ingestion + AGENTS.md respect + source-citation discipline; the bare prompt is the fallback for one-off invocations.

The deliverable structure (carries over from existing PROCESS):

- Executive summary (1 page) — what was built, current state, top 3 things the recipient must know
- Product overview — problem solved, users served, core features, success metrics
- Architecture — component diagram, data flow, technology choices with ADR pointers, scalability path
- Operations — local run, deploy (staging + prod), monitor, incident response
- Security — current posture, compliance status, known security debt
- Testing — strategy, coverage metrics, how to run, regression baseline
- Known limitations & tech debt — explicit, honest, prioritised
- Recommended next steps — first 30 days, first 90 days, next 12 months
- Contact & support — support terms, escalation path, key contacts
- Appendices — links to docs site, KT recordings, Linear export, Sentry handoff

**3.3 — Pull live operational signals at synthesis time, not from memory.** When generating "Known limitations" and "Recommended next steps", have Claude Code pull from Linear (`Via Linear MCP, list open issues labelled tech-debt OR bug, ordered by priority`) and from Sentry (`Via Sentry MCP, list the top 20 unresolved issues by event count over the last 30 days`). This prevents the document from being stale at the moment it is delivered.

**3.4 — Internal review.** Delivery Lead + Tech Lead read the synthesised draft together, fact-check claims against the source artefacts, edit for tone, and check the AGENTS.md customer-vs-internal split (no internal post-mortem language leaking into a recipient artefact). **Cross-check architecture diagrams and ADR cites** — Claude is most likely to hallucinate ADR numbers and component names when synthesising across artefacts.

**3.5 — Share the draft with the recipient for feedback.** They read it and ask questions; iterate based on confusion points. The questions themselves seed the FAQ in Step 7. Record the feedback in `/handoff/handoff-feedback.md` so the next engagement's handoff template improves.

**3.6 — Finalise and archive.** Lock the document under version control, link it from the docs site (Mintlify supports an internal `/handoff` route gated by `/authed/mcp`), and include the URL in the support-period agreement.

> **Time benchmark.** With the multi-repo `--add-dir` ingestion + the handoff-agent subagent, end-to-end synthesis is ~30 minutes of Claude time + ~1 hour of human review for a typical 6-month engagement. Without the AI loop the same artefact takes 2–3 days of focused human work.

> **Reference template:** [`templates/handoff-document-template.md`](../templates/handoff-document-template.md).

---

### Step 4: Knowledge Transfer (KT) Sessions

> Visual: [Step 4 flowchart](./FLOWCHART.md#step-4-knowledge-transfer-sessions--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | Handoff document (Step 3), recipient team's background, the 4–6 KT topics agreed in the support contract |
| **Tools** | **Fathom** (recommended for live-meeting recording + action-item extraction; 9.4/10 G2 vs Loom 7.2 in 2026) **or** **Loom AI** (recommended for asynchronous screen-walkthroughs with auto-chapter + auto-CTA); **Claude Code** for transcript-to-handoff-doc roll-up; **NotebookLM Enterprise** for cross-session synthesis on long engagements |
| **Output** | Recorded sessions with structured chapter-marked transcripts, action-item lists, updated handoff document, KB pages linked to recordings |
| **Human** | Delivery team presents; recipient drives Q&A; both teams capture decisions and follow-ups |

**Workflow:**

**4.1 — Plan the sessions.** Break into focused 30–60 minute topics — typical set:

1. **Architecture deep-dive** (60 min) — component model, data flow, key trade-offs, ADR walk-through
2. **Code walkthrough** (60 min, may split per service) — module by module, hot paths, how-to-add-a-feature reference
3. **Deployment demo** (45 min) — end-to-end staging deploy, blue/green or canary mechanics, rollback drill
4. **Incident response drill** (60 min) — simulate an alert, walk Bits AI SRE / Sift hypothesis, run the runbook, document gaps
5. **Q&A** (60 min, agenda-less) — recipient drives; this is the most valuable session

Generate a per-session script with the [`KT Session Script`](./PROMPTS.md#kt-session-script) prompt — gives the presenter timings, demo steps, anticipated questions, fallback if a demo breaks live.

**4.2 — Pick the recording tool per session shape.**

- **Fathom** is the recommended default for live calls (Zoom / Google Meet / Teams). 2026 G2 benchmarks put Fathom at 9.4/10 for Action Item Tracking vs Loom at 7.2 — for the deployment demo and incident drill where action items drive the support-period backlog, Fathom's quality matters.
- **Loom AI** is the recommended default for asynchronous screen walkthroughs (a 15-minute "how a service handles a request" recording the recipient watches at their own pace). Loom's auto-chapter and auto-CTA features make these recordings searchable reference material rather than 15 minutes of unindexed video.
- **Otter / Granola / Jamie** are reasonable fallbacks if the recipient already standardises on one — switching the recipient's transcription tool just for handoff is a friction antipattern.

**4.3 — Record every session.** The delivery team presents, the recipient asks questions and runs the demo on their own laptop where possible. Recording is **on by default** for every session — async retention is 3–4× better than written docs alone, and absent team members rely on the recording.

**4.4 — Generate structured summaries (Claude Code).** After each session, feed the transcript into Claude Code with the [`KT Session Summary`](./PROMPTS.md#kt-session-summary) prompt to produce: section headings with timestamps, key decisions per timestamp, action items per owner, open questions, and cross-link targets (runbooks/code/ADRs/dashboards mentioned). The prompt's `[REDACT-BEFORE-SHARE]` and `[AMBIGUOUS — review at mm:ss]` discipline keeps the summary honest rather than over-paraphrased. Commit the summary alongside the recording link.

**4.5 — Roll up summaries into the handoff document.** When the session reveals a clarification or a decision missing from the handoff document, append it. Run the `handoff-agent` subagent (Step 0.7) over the new transcript + the existing handoff doc to produce a clean diff:

```bash
claude  # (or via the handoff-agent subagent)
> Read the new KT-session summary at /handoff/kt-summaries/architecture-deep-dive.md
>   and propose updates to /handoff/handoff-document.md as a unified diff. Cite the
>   transcript timestamp for each proposed change.
```

Review the diff and apply.

**4.6 — Live Q&A is the most valuable session — treat it that way.** No agenda. Recipient drives. The questions surface gaps in the handoff document, the docs site, and the runbooks all in one go. Every question from the Q&A becomes a candidate for the FAQ in Step 7 and a candidate for a docs gap in Step 2.

**4.7 — For long engagements with many sessions, ingest into NotebookLM Enterprise.** If the engagement has more than ~6 hours of recordings + multiple slide decks, NotebookLM Enterprise's 300-source ceiling makes it the right place for cross-session search. Share the notebook with the recipient at handoff so they can ask `What did we decide about feature flagging across all five sessions?` and get an answer with timestamped citations.

> **Anti-pattern.** Don't skip the recording because docs exist. Video retention is 3–4× higher than text alone, and recordings preserve tacit knowledge ("the way Maria explains the rate-limit algorithm in 90 seconds") that no document captures.

---

### Step 5: Credential & Access Handoff

> Visual: [Step 5 flowchart](./FLOWCHART.md#step-5-credential--access-handoff--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | Every credential the engagement uses — cloud accounts, IAM roles, CI tokens, third-party SaaS, registrar, DNS, SSL, monitoring dashboards, AI/LLM API keys (Anthropic, Bedrock, Vertex), Pulumi access tokens, MCP-server OAuth grants |
| **Tools** | **1Password Business** (shared vault for handoff) + **1Password CLI** (`op`) + **Pulumi ESC** environment ownership transfer + cloud-native IAM (AWS IAM Identity Center, GCP IAM, Azure RBAC) |
| **Output** | Recipient owns every credential; delivery team's access is removed (or scheduled for removal at support-period end); shared credentials rotated by the recipient |
| **Human** | Security Champion + recipient jointly verify access for every service; both sign the access-transfer log |

**Workflow:**

**5.1 — Generate the credential inventory via 1Password CLI + Claude Code.** From a clean Claude Code session with `op run --` (so the CLI brokers vault access, no plaintext leaks):

```bash
op run -- claude
> List every item in the 'Engagement Handoff' shared vault and produce a markdown
>   inventory grouped by category (Cloud / CI / Third-party SaaS / Domain&DNS /
>   Monitoring / AI APIs / MCP OAuth grants). For each item include: service name,
>   item name, account/scope, role-transfer-possible (yes/no/unknown), preferred
>   transfer mechanism, post-handoff rotation required (yes/no). Never include
>   secret values.
```

Save as `/handoff/credential-inventory.md`. This is the working sheet for Steps 5.2–5.5.

**5.2 — Prefer role transfer over credential sharing — every time.** For every row in the inventory, classify:

- **Cloud (AWS / GCP / Azure):** add the recipient's principal to the IAM role / IAM Identity Center group / Cloud IAM binding. Do **not** share access keys. AWS specifically: cross-account role assumption is the canonical pattern.
- **SaaS admins (GitHub, Datadog, Sentry, Linear, Atlassian, Mintlify, etc.):** add the recipient as an org admin, then remove delivery-team admins post-handoff (or at support-period end if the contract requires continued access).
- **MCP OAuth grants:** the recipient authenticates against each MCP server (Pulumi, GitHub, Sentry, Atlassian/Notion, Linear, docs platform) on their own machine via the same `claude mcp add` flow the delivery team used in Step 0 / Phase 6. **No OAuth token is "transferred" — every MCP grant is per-developer per-account.**
- **Pulumi ESC environments:** transfer environment ownership (`pulumi env grant <env> <recipient-team>:admin`) so the recipient inherits the Phase 6 secrets posture without re-entering values. Revoke the delivery team's access at support-period end.
- **Only share raw credentials when role-based access isn't possible** — e.g., third-party API keys with no per-user model, registrar password (often single-account), pre-existing SSH keys for legacy systems.

**5.3 — For shared credentials: transfer via the 1Password Business shared vault.**

```bash
# Delivery-team Security Champion:
op vault create "Engagement Handoff"
op vault user grant --vault "Engagement Handoff" --user <recipient-email> --permissions allow_viewing,allow_editing
op item move <item-uuid> --vault "Engagement Handoff"  # for each shared item

# Recipient:
op vault list                                 # confirms vault visible
op item get <item-name> --vault "Engagement Handoff"
op item move <item-uuid> --vault "Recipient Vault"   # move to their own vault
```

The Security Champion **revokes the delivery team's access to "Engagement Handoff"** the same day the recipient confirms move-out.

**5.4 — Verify access for every row.** The recipient logs into every service with the new credentials / role. The Security Champion observes (screen-share) and signs off — no access is "verified by Slack message." Every confirmed row is marked `verified` with timestamp + observer.

**5.5 — Rotate every shared credential immediately after handoff.** Any credential that was transferred (not role-granted) is rotated by the recipient before the support-period ends. Why: even with a clean 1Password move, the credential existed in the delivery team's machines and CI environments at some point — rotation removes residual exposure.

**5.6 — Remove delivery-team access on a schedule.** Delivery-team admin access on each SaaS is removed at the agreed milestone — typically support-period end (Day 30), but for high-blast-radius services (cloud root, Pulumi org admin) revocation happens immediately after the recipient verifies access. Document the schedule in the support agreement.

> **Critical anti-patterns — never use for credential transfer.** Email. Slack. Teams DM. SMS. Unencrypted file shares. Screenshots in a document. PDF exports. Any of these is grounds for immediate rotation of the credential in question. The 1Password CLI + shared vault is the only path; if 1Password is unavailable for a specific recipient, fall back to Bitwarden Teams — never to messaging.

---

### Step 6: Infrastructure Handoff

> Visual: [Step 6 flowchart](./FLOWCHART.md#step-6-infrastructure-handoff--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | The full Phase 6 stack — Pulumi IaC repo + Pulumi Cloud workspace + Pulumi ESC environments + CrossGuard policy packs + GitHub repo + `.github/workflows/` + `.github/agentic-workflows/` + observability dashboards + runbooks + AGENTS.md + the MCP server roster (Pulumi, GitHub, Sentry, observability) |
| **Tools** | **Pulumi Cloud** workspace transfer (org → org) + **GitHub** repo ownership transfer + **MCP-server OAuth re-grant** on the recipient side + live reproducibility demo |
| **Output** | Recipient owns the IaC repo, the Pulumi state, the ESC environments, the CI/CD workflows, the dashboards, and **the same AI ops loop the delivery team ran** — Claude Code with Pulumi MCP, Sentry MCP, observability MCP, CrossGuard fixes, agentic workflows |
| **Human** | DevOps engineer demonstrates reproducibility live; recipient runs `pulumi preview` on their own machine before signoff |

**Workflow:**

**6.1 — Transfer the IaC repository.** Preferred: **move repo ownership** (GitHub: Settings → General → Transfer ownership). Fork-and-delete loses git history and breaks issue/PR continuity. After transfer the recipient receives all branches, tags, issues, PRs, Actions history, and the `.github/workflows/` + `.github/agentic-workflows/` directory intact.

**6.2 — Transfer the Pulumi Cloud workspace.** Pulumi Cloud supports org-to-org transfer for stacks; ESC environments can be re-homed via `pulumi env grant` followed by ownership transfer. The recipient's Pulumi org now owns the state, the stacks, the policy packs, and the audit log. **Coordinate the transfer window** — `pulumi up` against the transferring stacks must be paused for ~15 minutes during the handover.

**6.3 — Re-establish the MCP ops loop on the recipient's side.** This is the step that is most often forgotten. The Phase 6 Pulumi MCP, observability MCP, Sentry MCP, GitHub MCP, and Linear MCP wired into the delivery team's Claude Code sessions do not "move" — every recipient developer must re-run the `claude mcp add` commands from Phase 6 Step 0 + Phase 7 Step 0 against their own accounts. Walk them through it on the live session:

```bash
# Recipient side — re-establish the full Phase 6+7 MCP roster
claude mcp add --transport http --scope user pulumi    https://mcp.ai.pulumi.com/mcp
claude mcp add --transport http --scope user github    https://api.githubcopilot.com/mcp/
claude mcp add --transport http --scope user sentry    https://mcp.sentry.dev/mcp
claude mcp add --transport http --scope user grafana   https://<workspace>.grafana.net/api/mcp   # or datadog
claude mcp add --transport http --scope user linear    https://mcp.linear.app/mcp
claude mcp add --transport http --scope user mintlify-docs https://<your-docs>.mintlify.app/mcp   # or gitbook
claude mcp add --transport http --scope user atlassian https://mcp.atlassian.com/v1/sse           # or notion
```

Verify with `claude mcp list` on the recipient's machine — every server connected. **Smoke test:** `Via Pulumi MCP list_stacks for our org and run resource-search for tag:Environment=prod.` If the recipient's Claude Code returns real production stacks, the inheritance is live.

**6.4 — Demonstrate reproducibility live.** In a 60-minute session walk through:

1. **Provision a new dev environment from scratch** — `pulumi stack init dev-handoff-demo` → `pulumi config env add dev-handoff-demo acme/dev-ci` → `pulumi up`. Should complete in < 30 minutes from a clean clone (this is a Phase 6 handoff checklist requirement).
2. **Make a change via PR** — recipient edits a tag, opens a PR, the Anthropic Claude Code Action posts a review, CrossGuard runs, `pulumi preview` posts the cost-delta comment. Confirms the full Phase 6 PR loop works on the recipient side.
3. **Recover from a corrupted state simulation** — delete the dev-handoff-demo stack, recover via `pulumi refresh` + the documented procedure in `/docs/infrastructure.md`. Recipient runs the keystrokes; Delivery DevOps observes.
4. **Trigger an `@claude` mention on a PR** — confirms the recipient's `ANTHROPIC_API_KEY` (or Bedrock/Vertex/Foundry credential) is in place and the action runs against their own quota.

**6.5 — Subagents transfer with the repo.** All Phase 3 and Phase 6 subagents committed to `.claude/agents/` (`linear-task-agent`, `frontend-engineer`, `backend-engineer`, `code-reviewer`, `refactor-specialist`, plus any Phase 6 ops subagents) carry over with the repo transfer. The recipient inherits them automatically. Walk through `/agents` together — confirm each appears, smoke-test one (`Use the linear-task-agent to fetch my next assigned issue`).

**6.6 — Document any manual steps that aren't in IaC.** Should be rare — anything not in IaC by Phase 6 close was a gap. Capture residual manual steps in `/docs/infrastructure.md` under a "Manual Operations" section so the recipient can fix the gap on their own timeline.

**6.7 — Hand off the cloud accounts via Step 5's credential process.** The cloud accounts themselves (root credentials, billing, AWS Organizations master) follow Step 5's role-transfer-where-possible / vault-transfer-otherwise pattern.

> **Rule.** **IaC is the correct handoff artefact.** A recipient who cannot reproduce their own infrastructure is not a full owner. If Phase 6 didn't produce IaC, that's a gap to fix before handoff — not an excuse to ship a snowflake.

---

### Step 7: Knowledge Base Population

> Visual: [Step 7 flowchart](./FLOWCHART.md#step-7-knowledge-base-population--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | Handoff document (Step 3), KT session summaries (Step 4), Linear bug history, Sentry top issues, ADR set, glossary terms from PRD |
| **Tools** | **Atlassian Confluence** (via Atlassian Remote MCP) **or** **Notion** (via `mcp.notion.com/mcp`) — whichever the recipient already runs; **Claude Code** for content seeding; **Glean** as the cross-platform discovery overlay if the recipient has it |
| **Output** | KB seeded with FAQ, Glossary, Troubleshooting, Onboarding pages — every page cross-linked to code, runbooks, and ADRs; named owners assigned; quarterly review cadence agreed |
| **Human** | Recipient team designates content owners per area; both teams approve cross-link integrity |

**Workflow:**

**7.1 — Use the recipient's existing KB platform — don't introduce a new one.** Most recipients already run Confluence or Notion (Confluence for enterprises, Notion for younger orgs). Both have official remote MCP servers as of 2026 — Atlassian Rovo MCP went GA in February 2026 with 72+ tools across Jira/Confluence/Compass; Notion's MCP is published by makenotion and ships as the official `mcp.notion.com/mcp` endpoint. Connect Claude Code per Step 0.2 if you haven't already.

**7.2 — Seed the four core content types via Claude Code MCP writes.** Run the [`Knowledge Base Seeding`](./PROMPTS.md#knowledge-base-seeding-faq-glossary-troubleshooting) prompt for each:

- **FAQ** — 15–20 questions derived from KT-session Q&A transcripts + anticipated PRD questions. Group by Getting Started / Billing / Features / Technical / Troubleshooting.
- **Glossary** — domain terms from PRD + technical terms from architecture docs + internal naming from the codebase.
- **Troubleshooting** — symptom / likely cause / diagnosis / resolution / prevention rows derived from Linear bug history (`Via Linear MCP, list closed issues labelled bug from the last 6 months`) + Sentry top errors (`Via Sentry MCP, list top 30 unresolved issues by event count`).
- **Onboarding** — new-team-member runbook covering local dev, MCP setup, first PR, support-period contacts.

Example invocation (Confluence):

```bash
claude
> Via Atlassian MCP, in space "PROJECT-KB" parent page "Onboarding":
>   1. Generate the FAQ page using the kt-session transcripts at /handoff/kt-summaries/
>      and the PRD executive summary at ../prd-repo/PRD.md.
>   2. For each Q, link to the runbook, ADR, or code file that authoritatively answers it.
>   3. Output as a Confluence page draft for human review before publishing.
```

**7.3 — Cross-link discipline.** Every KB page links to:

- The **canonical code location** (file path or module name) it describes
- The **runbook** (if operational) at `/docs/runbooks/<alert>.md`
- The **ADR** (if architectural) at `/docs/adr/<n>.md`
- The **Linear label** for related ongoing work

No isolated KB pages. A page with no cross-links is a candidate for deletion at quarterly review — it has no source of truth and will rot.

**7.4 — Assign named owners per content area.** Recipient team designates one named human (not a team alias) per content area: FAQ owner, Glossary owner, Troubleshooting owner, Onboarding owner. Ownerless docs decay within months — this is the single most reliable predictor of KB health.

**7.5 — Set the review cadence — quarterly.** Every quarter the owners run the [`Quarterly KB Completeness Check`](./PROMPTS.md#quarterly-kb-completeness-check) prompt against the live KB + Linear backlog + Sentry top issues:

```bash
claude
> Via Atlassian MCP, fetch the Troubleshooting page in the KB.
> Via Sentry MCP, list top 20 unresolved issues by event count last 90 days.
> Via Linear MCP, list closed bugs labelled customer-impact last 90 days.
> Now run the Quarterly KB Completeness Check prompt — diff the KB against the live signal
>   and emit ADD/EDIT/DELETE rows in the prompt's table format. Only flag changes the
>   live signal supports.
```

The prompt's "only flag what the live signal supports" rule is what keeps the review honest — without it, an AI can fabricate gaps that look plausible but trace to nothing. This is the same shape as Step 2's completeness check; it works because the KB and the source-of-truth signals (Linear, Sentry) all expose MCP servers.

**7.6 — Layer Glean if the recipient already runs it.** Glean's permissions-aware retrieval over Slack + Confluence + Notion + Drive + Linear answers the cross-platform "where is this documented?" question that no single KB tool solves. If the recipient already runs Glean, ensure the new KB pages are indexed (Glean autodiscovers from connected sources) and link the Glean answer URLs into the handoff document. **Don't introduce Glean** at handoff — it's an org-level commitment, not a per-project tool.

---

### Step 8: Post-Handoff Support Period

> Visual: [Step 8 flowchart](./FLOWCHART.md#step-8-post-handoff-support-period--detail)

| Attribute | Detail |
|-----------|--------|
| **Input** | Completed handoff (Steps 1–7), agreed support terms (typically 30 days), inherited error backlog (Sentry), inherited on-call rotation (incident.io / PagerDuty) |
| **Tools** | **Linear MCP** (bug pipeline carries over from Phase 3) + **Sentry + Seer + Sentry MCP** (error triage) + **incident.io** or **PagerDuty AIOps** (on-call) + **Anthropic Claude Code Action** (`@claude` review on PRs) + the full inherited MCP roster from Step 6 |
| **Output** | Recipient operates the system independently using the same agentic loop the delivery team used; every issue raised during the support period traces back to a doc/runbook update |
| **Human** | Delivery team as on-call backup within agreed SLA; recipient owns the primary on-call rotation; both teams meet weekly during the support period |

**Workflow:**

**8.1 — Define the support period in writing.** Typical: 30 days of "on-call backup" where the delivery team helps with issues that arise. The support agreement names: SLAs (S0/S1 within 2 hours, S2 within 1 business day), the escalation path (recipient on-call → delivery on-call → delivery Tech Lead), the definition of "in scope" (production incidents, bugs introduced during handoff, doc gaps), and "out of scope" (new features, architectural changes — these become a separate engagement).

**8.2 — Recipient's Claude Code inherits the full agentic stack.** Confirmed in Step 6.3 + 6.4 + 6.5 — the recipient's developers can:

- Run `@claude` review on every PR via the inherited Claude Code Action workflow
- Query Pulumi MCP for infra changes, drift, cost
- Query Sentry MCP for inherited error backlog (`Via Sentry MCP, list top 10 issues by event count last 24h; for the top one, ask Seer for a root-cause hypothesis`)
- Query observability MCP (Grafana / Datadog) for dashboards and SLO breaches
- Query Linear MCP for the support-period bug pipeline
- Use the `linear-task-agent` and specialist subagents (carried over) for fixes during the support period

This is what "the support period is partly the recipient learning to run the AI loop themselves" means in practice. Schedule one paired-Claude-Code session per week during the first month so the delivery team can answer "how do I prompt this" questions in real time.

**8.3 — Inherited error backlog — triage with Sentry + Seer.** The recipient's first job week one is to scan the inherited Sentry issues. Seer auto-prioritises and proposes fixes; the recipient runs the [`Inherited Error Triage`](./PROMPTS.md#inherited-error-triage) prompt, which produces a Linear-import-ready CSV with explicit headers, severity, suggested owner from CODEOWNERS, and Seer confidence per row:

```bash
claude
> Via Sentry MCP, list top 30 unresolved issues in project <recipient-project>
>   ranked by event count last 30 days. Now run the Inherited Error Triage prompt
>   with our S0/S1/S2/S3 rubric, our CODEOWNERS, and our support-period bandwidth
>   so the fix-me-first cap stays inside what we can actually deliver.
```

The recipient imports the CSV into Linear with the `inherited-bug` label. Each is then a normal Phase 3 story — `linear-task-agent` picks it up, the appropriate specialist subagent implements the fix, the Claude Code Action reviews the PR. **The agentic loop is the support process.**

**8.4 — Inherited on-call — incident.io or PagerDuty AIOps.** Whichever incident-management tool the engagement set up in Phase 6 carries over. incident.io's Scribe transcribes incident calls in real time and extracts root-cause and follow-ups, which feed into post-mortems and the KB Troubleshooting page. PagerDuty AIOps groups noisy alerts before paging — both reduce the cognitive load on the recipient's first weeks of on-call. The Phase 6 runbook-per-alert convention carries over; on-call hits a runbook, runs the steps, files a Linear issue if the runbook is wrong.

**8.5 — Every issue reveals a documentation/runbook gap — close the loop.** The discipline that makes the support period valuable: every support request is a defect in the handoff package. After resolution, the on-call (or the Tech Lead) updates the relevant doc/runbook/KB page **the same day** — Claude Code can do the rewrite from the issue thread + the actual fix commit in ~10 minutes. Track "support-period issues that updated docs" as a metric — should be 80%+ in a healthy handoff.

**8.6 — Weekly checkpoint during the support period.** 30-minute sync between delivery Tech Lead and recipient Tech Lead. Agenda: open support tickets, recurring confusion points, doc gaps closed this week, AI-loop friction (where is Claude not helping yet). Ten minutes of conversation here saves hours of unnecessary escalation.

**8.7 — 30-day support-period closure.** Hold a formal closure meeting:

- Review every issue raised: resolved? doc updated?
- Confirm delivery-team admin access has been removed where the schedule says it should be
- Confirm rotated credentials are rotated
- Recipient signs off that they can operate the system independently
- File the [`Post-Handoff Retrospective`](./PROMPTS.md#post-handoff-retrospective) prompt output as `/handoff/post-handoff-retro.md`

After this meeting the delivery team transitions to **on-demand availability only** — paid engagement, defined scope, no implicit obligation.

**8.8 — 90-day debrief (final).** A second meeting at Day 90 — the recipient has now run the system without backup for 60 days. Review:

- What broke that the handoff didn't anticipate?
- Has the doc decayed already? Run the Step 7.5 quarterly check.
- Recipient satisfaction (1–5 score, narrative comments)
- Lessons that should feed into the next AI-DLC engagement's Phase 7 process

Document under `/handoff/90-day-debrief.md` and surface the lessons to the AI-DLC framework owner — **the framework only improves when this loop closes.**

---

## Phase Handoff

This is the final phase — there is no next phase. "Handoff" here means the complete transition of the project to its long-term owner, including the AI operating model.

| Artefact | Format | Location |
|----------|--------|----------|
| Release notes (all releases) | Markdown | GitHub Releases + CHANGELOG.md |
| Documentation site (live) | Web | Mintlify / GitBook / Docusaurus deployed URL |
| Auto-generated `llms.txt` + `llms-full.txt` + `skill.md` (if Mintlify) | Text | Docs site root |
| Handoff document | Markdown | `/handoff/handoff-document.md` (and shared copy with recipient) |
| KT session recordings | Video + transcript | Fathom / Loom links shared with recipient |
| KT structured summaries | Markdown | `/handoff/kt-summaries/<topic>.md` |
| (Optional) NotebookLM Enterprise notebook | Hosted | Shared with recipient with read access |
| Credential inventory | Markdown (no secrets) | `/handoff/credential-inventory.md` |
| Credential vault access | 1Password Business shared vault | Transferred to recipient, delivery access revoked |
| Pulumi Cloud workspace | Cloud-hosted | Org-transferred to recipient |
| Pulumi ESC environments | YAML in Pulumi Cloud | Ownership granted to recipient team |
| IaC repository | Git | Repo ownership transferred to recipient |
| `.github/workflows/` + `.github/agentic-workflows/` | YAML + Markdown | Transferred with the repo |
| `.github/workflows/claude.yml` (PR review) + `release-notes-polish.yml` | YAML | Transferred with the repo |
| `.claude/agents/` subagents (linear-task-agent, frontend-engineer, backend-engineer, code-reviewer, refactor-specialist, handoff-agent if committed) | Markdown | Transferred with the repo |
| `AGENTS.md` (with Phase 7 Delivery & Handoff section) | Markdown | Repo root |
| MCP server roster transferred | Per-developer OAuth grants | Re-established on recipient side via `claude mcp add` |
| Observability dashboards | Datadog / Grafana JSON + dashboards | `/observability/dashboards/` + transferred workspace access |
| SLO + alert definitions | YAML / Datadog config | `/observability/slos/` |
| Runbooks per alert | Markdown | `/docs/runbooks/` |
| Knowledge base | Atlassian Confluence / Notion | Seeded with FAQ + Glossary + Troubleshooting + Onboarding |
| Linear workspace access | Per-developer OAuth | Recipient-side `claude mcp add` |
| Sentry project access | Per-developer OAuth | Recipient-side `claude mcp add` |
| Post-handoff support agreement | Contract / MSA | Signed document |

**Handoff Complete Checklist:**

- [ ] Latest release tagged, polished, and published; release-notes-polish workflow runs on every tag
- [ ] Documentation site live, comprehensive, completeness-check clean
- [ ] (If Mintlify or GitBook) docs MCP responding from the recipient's Claude Code
- [ ] Handoff document reviewed by recipient, all feedback integrated
- [ ] KT sessions completed, recorded, and structured-summarised
- [ ] All credentials transferred via 1Password Business (NEVER email / Slack)
- [ ] Pulumi Cloud workspace transferred; ESC environments re-homed; recipient runs `pulumi preview` successfully on their machine
- [ ] IaC repository transferred; recipient provisions a fresh dev environment from scratch in < 30 minutes (live demo)
- [ ] `.github/workflows/` + `.github/agentic-workflows/` carried over; `@claude` mention works on a recipient PR
- [ ] All `.claude/agents/` subagents listed by `/agents` on the recipient's machine; smoke test passes
- [ ] `AGENTS.md` (with Phase 7 section) committed at repo root
- [ ] Recipient's Claude Code shows the full inherited MCP roster as `connected`: `pulumi`, `github`, `sentry`, `grafana`/`datadog`, `linear`, `mintlify-docs`/`gitbook-docs`, `atlassian`/`notion`
- [ ] Knowledge base seeded with named owners; quarterly review cadence agreed
- [ ] Sentry inherited-error triage CSV imported into Linear with `inherited-bug` label
- [ ] On-call rotation transferred (incident.io / PagerDuty AIOps); recipient is primary, delivery is backup for the support period
- [ ] Pulumi Cloud audit log enabled on recipient side; GitHub audit log enabled; KB audit log enabled
- [ ] Post-handoff support period defined in writing with SLAs and escalation
- [ ] Recipient confirms operational independence

**After 30 / 90 days:**

- [ ] **30-day support-period closure:** every issue traces to a doc update; delivery-team admin access removed per schedule; rotated credentials confirmed rotated; closure meeting held; retrospective filed
- [ ] **90-day debrief:** recipient satisfaction captured; doc decay measured (Step 7.5 quarterly check); lessons surfaced to AI-DLC framework owner; engagement formally concluded

---

## Risks & Guardrails

| Risk | Mitigation |
|------|------------|
| **AI-polished release notes ship with confidently-wrong feature emphasis** — Claude promotes a minor feature as the star, or under-emphasises a breaking change | Step 1.4 mandates Release Manager review of every polished draft. Breaking changes in the prompt are pinned to "render at the top with migration guidance"; the human verifies. |
| **Hallucinated facts in the handoff document** — wrong ADR number, wrong component name, wrong runbook path | Step 3.3 has Claude pull live signals from Linear/Sentry MCP at synthesis time, not from memory. Step 3.4 mandates fact-check across artefacts. The `handoff-agent` subagent's system prompt requires inline source citation — uncited claims are removed. |
| **Customer-facing docs leak internal information** — internal post-mortem language, performance feedback, raw incident channel quotes | `AGENTS.md` Phase 7 section explicitly lists forbidden content. The `handoff-agent` subagent must escalate when an artefact would breach the customer/internal split. Mintlify supports `/authed/mcp` for an internal-only docs surface — keep internal handoff docs there, not on the public site. |
| **Credential leak via plaintext in `mcp.json` / settings / Claude Code transcript** | Step 5.1 + 0.5: `op run -- claude` brokers vault access; Claude receives item names not values. Step 5 anti-pattern list: never email/Slack/SMS. |
| **MCP OAuth grants forgotten at off-boarding** — delivery developer leaves the engagement, retains access to the recipient's docs site / Pulumi / Sentry | Off-board checklist includes revoking OAuth grants in every MCP-issuing platform: Pulumi, GitHub, Sentry, Atlassian/Notion, Mintlify/GitBook, Linear, 1Password. Audit log on every platform makes orphan grants visible. |
| **MCP roster doesn't transfer** — recipient inherits the IaC repo but their Claude Code has no MCP servers wired, so they fall back to manual ops | Step 6.3 explicitly walks the recipient through the full `claude mcp add` sequence on their own machine during the live handoff session. Verification checklist at handoff-complete includes `claude mcp list` showing every server connected on a recipient developer's box. |
| **Subagents stop working after repo transfer** — `.claude/agents/` files reference paths or tool scopes that the recipient's environment does not have | Step 6.5 smoke-tests at least one subagent on the recipient's machine before signoff. Subagent system prompts use placeholders for repo-specific values (`{{TEST_RUNNER}}` etc.) so the recipient can refill without rewriting the boundary rules. |
| **KT recording becomes the only source of truth** — "watch the Loom" replaces a written runbook | Step 4.5 rolls every recording's structured summary into the handoff doc and a runbook update. Step 7.2 derives FAQ + Troubleshooting from KT transcripts so the searchable form lives in the KB, not just in the video. |
| **KB decays within 90 days** — pages have no owner, no cross-links, and no review cadence | Step 7.4 mandates named owners per content area (not team aliases). Step 7.5 requires the quarterly Claude Code completeness check. Pages with no cross-links are candidates for deletion. |
| **Inherited error backlog overwhelms the recipient in support week 1** | Step 8.3 has Seer + Sentry MCP triage the backlog into fix-me-first / watch / WONTFIX before the recipient touches it. The CSV import into Linear with `inherited-bug` label means the support-period queue is bounded and prioritised, not a flat list of 200 issues. |
| **Support-period issues don't close the doc-loop** — recipient resolves an issue but never updates the runbook/FAQ | Step 8.5 metric: "support-period issues that updated docs" tracked at weekly checkpoint. Falling below 80% triggers the Tech Lead to redo the doc Claude Code prompt for the affected area. |
| **Recipient never adopts the agentic loop** — they treat MCP servers as optional, fall back to manual ops, and the AI productivity gains evaporate post-handoff | Step 6.4 reproducibility demo includes an `@claude` PR review and a Pulumi MCP query on the recipient's machine. Step 8.2 schedules paired Claude Code sessions weekly for the first month — the recipient sees the loop in real-time on their codebase. |
| **Glean / NotebookLM Enterprise introduced for handoff that the recipient won't keep** — extra SaaS the recipient never adopts | Step 7.6 + Step 4.7 explicitly say: don't introduce them at handoff. They are recipient-side org commitments, not per-engagement tools. |

---

## Daily Delivery & Handoff Workflow (handoff week)

```
Day 1 (Mon):
  1. Pulumi Cloud workspace transfer + ESC re-home (~15 min downtime window)
  2. GitHub repo ownership transfer (instant)
  3. Recipient runs claude mcp add for full Phase 6+7 roster on their machine
  4. claude mcp list smoke test on recipient side — every server connected
  5. Live reproducibility demo: pulumi up against fresh dev stack (60 min)

Day 2 (Tue):
  6. KT session 1 — Architecture deep-dive (Fathom recording)
  7. Claude Code generates structured summary; appended to handoff doc

Day 3 (Wed):
  8. KT session 2 — Code walkthrough (per service)
  9. KT session 3 — Deployment demo + rollback drill
 10. Credential inventory walk via op run -- claude

Day 4 (Thu):
 11. KT session 4 — Incident response drill (live alert simulation)
 12. Credential transfer via 1Password Business shared vault
 13. Recipient verifies access service-by-service with Security Champion

Day 5 (Fri):
 14. KB seeding — Claude Code over Confluence/Notion MCP (FAQ, glossary, troubleshooting)
 15. KT session 5 — agenda-less Q&A (90 min)
 16. Day-1 of support period scheduled; weekly checkpoint cadence agreed

Days 6-30 (Support period):
 17. Recipient on-call primary; delivery on-call backup within SLA
 18. Inherited error backlog triaged via Seer + Sentry MCP; CSV imported to Linear
 19. Every issue → doc/runbook/KB update by EOD
 20. Weekly 30-min checkpoint Tech Lead ↔ Tech Lead
 21. Day-30: closure meeting + retrospective filed

Day 90:
 22. 90-day debrief; doc decay check; lessons to AI-DLC framework owner
```

---

## Related Documents

- [Prompt Templates →](./PROMPTS.md)
- [Quality Gates →](./QUALITY-GATES.md)
- [Process Flowchart →](./FLOWCHART.md)
- [Handoff Document Template →](../templates/handoff-document-template.md)
- [Release Notes Template →](../templates/release-notes-template.md)
- [ADR Template →](../templates/adr-template.md)
- [Phase 3 Linear MCP setup (carries over) →](../03-development/PROCESS.md#step-0-one-time-setup--connect-claude-code-to-linear-via-mcp)
- [Phase 6 Pulumi / GitHub / observability MCP setup (carries over) →](../06-cicd-devops/PROCESS.md#step-0-one-time-setup--wire-ai-tools-into-the-devops-loop)
- [Delivery & Handoff Tools Evaluation →](../../docs/tools-evaluation/7.Delivery_Handoff_Phase_Tools.md)

## External References

- [Mintlify MCP server documentation](https://www.mintlify.com/docs/ai/model-context-protocol) — auto-hosted MCP at `/mcp` and `/authed/mcp`, search + filesystem tools, llms.txt + skill.md
- [GitBook MCP server for published docs](https://gitbook.com/docs/ai-and-search/mcp-servers-for-published-docs) — auto-hosted MCP at `/~gitbook/mcp`, OAuth via DCR
- [Atlassian Rovo MCP Server (GA Feb 2026)](https://www.atlassian.com/platform/remote-mcp-server) — 72+ tools across Jira, Confluence, Compass; OAuth 2.1; v1/sse endpoint deprecated 30 June 2026
- [Notion MCP](https://developers.notion.com/guides/mcp/get-started-with-mcp) — official `mcp.notion.com/mcp`; bundled Claude Code Notion plugin available
- [GitHub MCP server (v1.0.2 April 2026)](https://github.com/github/github-mcp-server) — release ops, PR comments, Actions inspection
- [Anthropic Claude Code Action v1](https://github.com/anthropics/claude-code-action) — `anthropics/claude-code-action@v1`; auto-mode-detection in 2026
- [semantic-release](https://semantic-release.gitbook.io/) — auto-tag + initial changelog from Conventional Commits
- [Conventional Commits](https://www.conventionalcommits.org/) — commit format specification
- [Fathom AI](https://fathom.video/) — meeting recording with action-item extraction (9.4/10 G2 in 2026)
- [Loom AI](https://loom.com/) — screen-recording with auto-chapters + auto-CTAs + summaries
- [NotebookLM Enterprise](https://cloud.google.com/resources/notebooklm-enterprise) — multi-source ingestion, 300 sources × 500K words; April 2026 auto-source-labelling update
- [1Password Business + CLI](https://developer.1password.com/) — `op run` pattern, vault references, Unified Access (March 2026 GA)
- [Pulumi ESC environment ownership transfer](https://www.pulumi.com/docs/esc/) — `pulumi env grant` for re-homing CI secrets at handoff
- [Sentry Seer + MCP server](https://docs.sentry.io/product/sentry-mcp/) — `mcp.sentry.dev/mcp`; inherited error backlog triage
- [incident.io Scribe](https://incident.io/) — real-time incident-call transcription + post-mortem extraction
- [PagerDuty AIOps](https://www.pagerduty.com/platform/aiops/) — alert grouping and noise reduction for inherited on-call
- [Glean](https://www.glean.com/) — permissions-aware enterprise AI search overlay
- [docusaurus-plugin-openapi-docs](https://docusaurus-openapi.tryingpan.dev/) — interactive API reference rendering for Docusaurus
- [Phase 7 Tools Evaluation](../../docs/tools-evaluation/7.Delivery_Handoff_Phase_Tools.md)
