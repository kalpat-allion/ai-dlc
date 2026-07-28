# Claude PR Review Bot — CI Integration

Companion to [`PROCESS.md`](./PROCESS.md) → [Step 4: Code Review](./PROCESS.md#step-4-code-review). This document specifies the **repo-calibrated automated reviewer** that runs `anthropics/claude-code-action` inside GitHub Actions and posts an inline review on every pull request.

It is a documented, production-proven pattern — not a sketch. The reference implementation is the **Maestro Command Center** repo (NestJS + React 18 Nx monorepo).

**This is one of two AI PR reviewers the AI-DLC supports.** The other is CodeRabbit. They are peer options, not a primary and a fallback — a project configures whichever fits, or both. [Step 4.3](./PROCESS.md#step-4-code-review) holds the selection rule; this document is the build spec for teams that pick this one. Nothing below assumes CodeRabbit is or isn't also running.

> **Advisory, not a gate-keeper.** The bot never approves a PR. It posts findings; humans decide. Its worst failure mode is silence, not a bad merge — which is why every design decision below biases toward *fail-open and non-blocking*.

---

## 1. What this reviewer is for

This bot reviews every PR against a **checklist file committed in the repo**, so team conventions are enforced by the same artifact that documents them. That is its whole character, and it is what the configuration choice turns on:

- **Its rules are yours.** The error formatter every catch block must use, the dual-broadcast invariant, the room-naming convention, the config-service-not-`process.env` policy, the module that must never import from another — none of this is inferable from a diff. It has to be written down, and here it is written down in a file the team owns.
- **A wrong finding is a fixable bug.** Because the rules live in the repo, a bad review is a PR against the checklist, merged the same day. There is no vendor ticket and no waiting.
- **The same checklist drives the local pre-PR gate.** One file, two consumers (§2) — a rule agreed after an incident applies to both without being written twice.
- **You own the cost curve.** Spend is per-run, not per-seat, which means it scales with PR volume and needs explicit controls (§5). On a small team that is cheaper than a subscription; on a high-volume monorepo it may not be.

The trade is setup and upkeep. This reviewer is only as good as the checklist behind it — an empty or aspirational checklist produces a generic review, which is precisely what an off-the-shelf reviewer gives you for free. **If the team will not maintain the checklist, configure CodeRabbit instead**; see the selection rule in [Step 4.3](./PROCESS.md#step-4-code-review).

---

## 2. The three artifacts

The integration is three committed files. Keeping them separate is the point — the prompt is CI plumbing, the checklist is team policy, and they change at completely different rates.

| File | Role | Changes when |
|------|------|--------------|
| `.github/workflows/claude-pr-review.yml` | The workflow: triggers, size gate, prompt build, action invocation, failure handling | Rarely — CI plumbing |
| `.github/prompts/pr-review.md` | Prompt template with `__PLACEHOLDER__` tokens. Tells the agent how to gather context, which file paths map to which domain area, and how to post the review | Occasionally — when the review *procedure* changes |
| `.claude/prompts/pr-review-checklist.md` | **Single source of truth for what "good" means in this repo.** Sections A–Z, each a bullet list of rules with severities | Continuously — every convention the team agrees on lands here |

Optionally a fourth: `.github/workflows/claude-pr-review-manual.yml`, a `workflow_dispatch` variant for on-demand reviews (§5.4).

### The checklist is shared, not duplicated

The same checklist file drives **both** the CI bot and the local pre-PR review agent (Step 3.5 `code-reviewer`). One file, two consumers — so a rule added after a production incident immediately applies to both the pre-PR gate and the post-open review. Never fork it.

Structure the checklist by dimension, not by file type:

```markdown
# PR Review Checklist (Single Source of Truth)

Used by both:
- `.github/prompts/pr-review.md` — CI auto-reviewer (applies all sections)
- `.claude/agents/local-pr-review.md` — local pre-PR gate (each parallel agent owns one disjoint slice)

Edit this file to update both.

For each finding report: **file path**, **line number**, **severity**
(Critical / Important / Suggestion), **description**, **fix**.

### A. Silent Failures & Error Handling
### B. Async / Await
### C. Unit + Integration Test Coverage
### D. {{BACKEND_FRAMEWORK}} & Backend Conventions
### E. {{DATASTORE}} & Database
### F. TypeScript & Type Safety
### G. Security
### H. {{FRONTEND_FRAMEWORK}} & Frontend
...
### V. Idempotency & Duplicate Handling
```

The reference implementation runs 22 sections (A–V), including conditional domain sections that only apply when a specific module is touched. Two rules keep it maintainable:

- **Every section belongs to exactly one slice** of the local gate's dimension map, so parallel local agents never review the same rule twice.
- **Exemptions are explicit.** Section C ends with a "do not flag" list (type-only files, constants, generated files, pure presentational components). Without it, the bot flags missing tests on `types.ts` forever and the team learns to ignore it.

---

## 3. Trigger model

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review, labeled]

concurrency:
  group: claude-pr-review-${{ github.event.pull_request.number }}
  cancel-in-progress: true
```

| Event | Why |
|-------|-----|
| `opened` | First review pass |
| `synchronize` | Re-review on every push — the author fixes, the bot re-checks |
| `reopened` | A closed-then-reopened PR needs a fresh pass |
| `ready_for_review` | Draft → ready is the normal entry point |
| `labeled` | Lets a **draft** opt in early via a `bot-review` label |

**What is deliberately absent, and why.** Do not "helpfully" sync this list with the main CI workflow's triggers:

- **No `unlabeled`** — removing a label should not kick off a fresh review pass.
- **No `converted_to_draft`** — dropping to draft is a signal to stop reviewing, not to start.

`concurrency` with `cancel-in-progress` means a rapid push sequence produces one review of the final state, not four overlapping reviews racing to comment.

### Draft opt-in

Drafts are skipped by default — reviewing an unfinished branch burns tokens on code that will change. A developer who wants early feedback adds the `bot-review` label:

```yaml
if: >-
  needs.check-size.outputs.too_large == 'false'
  && (github.event.pull_request.draft == false
      || contains(github.event.pull_request.labels.*.name, 'bot-review'))
  && !(github.event.action == 'labeled' && github.event.label.name != 'bot-review')
```

The final clause is load-bearing. Without it, **every** label add — `bug`, `frontend`, a triage label — re-triggers a full review. That clause alone is the difference between predictable spend and a surprise bill.

---

## 4. Idempotency across passes

The bot re-runs on every push. Without explicit handling it re-raises findings the author already answered, and the review devolves into noise the team learns to scroll past. Two mechanisms prevent that.

**4a — Tag mode preserves GitHub context.** Setting `track_progress: true` makes the action auto-inject the PR's existing comments and review threads into the agent's context, so it sees prior passes and the author's replies.

One constraint: `track_progress` **hard-errors on the `labeled` event** (the action only supports it for `opened`, `synchronize`, `ready_for_review`, `reopened`). Gate on the action, not on draft status — draft pushes arrive as `synchronize` and should get tag mode:

```yaml
track_progress: ${{ github.event.action != 'labeled' }}
```

The `labeled` pass is a draft's *first* review, so there are no prior threads to preserve — the loss is harmless, and the prompt reads threads explicitly via `gh api` anyway.

**4b — The prompt reads prior threads directly.** Independent of tag mode, the context-gathering step fetches both:

```
gh api repos/__REPO__/pulls/__PR_NUMBER__/comments --paginate   # inline threads + author replies
gh api repos/__REPO__/pulls/__PR_NUMBER__/reviews  --paginate   # prior review summary bodies
```

Both are needed. Findings raised **inline** live in `/comments`; findings raised in the **summary body** (the aggregated `## Critical Issues` block) live on the review objects in `/reviews`. Fetching only the first re-raises every summary-level finding on every pass.

The prompt then instructs:

- If the same issue was raised at the same file + location, the author replied `❌ Disagreed` / `⏭ Deferred` / `Won't change` with reasoning, and **the code at that location is unchanged** → do not re-post. Re-raise only if the diff materially changed that code.
- Findings marked `✅ Fixed` and actually fixed are done.
- When unsure whether a thread is settled, **err toward not posting.** Human approval gates the merge, not the bot; silently re-flagging answered findings just churns the review.

Adopt the `❌ Disagreed` / `⏭ Deferred` / `✅ Fixed` reply convention as a team norm — the mechanism only works if authors actually mark threads.

---

## 5. Cost and scope controls

Per-run token spend is the one way this integration goes wrong quietly. Five controls, all worth having from day one.

**5.1 — Size gate.** A pre-job counts changed files and skips oversized PRs with an explanatory comment rather than failing silently:

```yaml
jobs:
  check-size:
    if: github.actor != 'dependabot[bot]'
    runs-on: ubuntu-latest
    permissions: { pull-requests: read }
    outputs:
      too_large: ${{ steps.count.outputs.too_large }}
      file_count: ${{ steps.count.outputs.file_count }}
    steps:
      - uses: actions/github-script@v9
        id: count
        with:
          script: |
            const { data: files } = await github.rest.pulls.listFiles({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.payload.pull_request.number,
              per_page: 100
            });
            core.setOutput('file_count', files.length);
            core.setOutput('too_large', files.length > 70 ? 'true' : 'false');
```

Calibrate the ceiling to your repo. The reference implementation started at 50, found it clipped legitimate medium PRs (refactors, dep bumps touching a lockfile plus several `package.json` files), and settled on **70**. Too low and the bot is useless on real work; too high and a single PR overflows the reviewer's context budget and produces a shallow review anyway.

**5.2 — Skip notice.** When the gate trips, a separate job comments on the PR. Never let an oversized PR pass without a review *and* without a message — the author will assume it was reviewed.

**5.3 — Bot-author exclusion.** `if: github.actor != 'dependabot[bot]'` on the size job short-circuits the whole workflow for dependency bumps.

**5.4 — Manual variant with a higher ceiling.** A `workflow_dispatch` twin (`claude-pr-review-manual.yml`) takes a PR number as input and runs the same prompt with a higher file ceiling (the reference uses **150**) and a longer timeout. Manual review is opt-in and deliberate, so the automatic ceiling doesn't apply.

The manual workflow has one extra requirement: it checks out the **PR head SHA**, not the merge ref, so it must fetch the prompt and checklist from `origin/main` explicitly. Otherwise a PR cut before the latest checklist update gets reviewed against a stale checklist:

```bash
git fetch --depth=1 origin main
git show origin/main:.github/prompts/pr-review.md > /tmp/pr-review-template.md
mkdir -p .claude/prompts
git show origin/main:.claude/prompts/pr-review-checklist.md > .claude/prompts/pr-review-checklist.md
```

The auto workflow side-steps this because `pull_request` events check out `refs/pull/N/merge`, which already contains base content.

**5.5 — Turn and tool budget.** `--max-turns 150` caps the agent loop, and an explicit `--allowedTools` allowlist stops it wandering into unbounded exploration:

```
--allowedTools "Bash(gh pr diff:*),Bash(gh pr view:*),Bash(gh api:*),Bash(git diff:*),Bash(git log:*),Read,Glob,Grep,mcp__github__create_pending_pull_request_review,mcp__github__add_comment_to_pending_review,mcp__github__submit_pending_pull_request_review"
```

Note what is **not** there: no `Write`, no `Edit`, no unrestricted `Bash`. The reviewer is read-only on the working tree and write-only through the three GitHub MCP review tools.

**Optional — self-hosted runners.** Teams already running a self-hosted fleet can move the review job onto it for roughly an order-of-magnitude cost reduction versus GitHub-hosted runners. The action self-installs its own runtime; the box needs only Node and git. On minimal images (e.g. AL2023) expect to install `libatomic` and best-effort-install the `gh` CLI. This is an optimisation, not a requirement — start on `ubuntu-latest`.

---

## 6. Output contract

The bot's review is only useful if its output is predictable. Fix the contract in the prompt.

**Severity buckets** — aggregate every finding into four sections, matching the Step 3.5 self-review prompt so developers read one vocabulary across the whole phase:

```
## Critical Issues — must fix before merge
## Important Issues — should fix
## Suggestions — nice to have
## Strengths — what is well done
```

Keep `## Strengths`. A reviewer that only ever criticises gets muted.

**Posting mechanism** — the action's GitHub MCP server splits review writes into three tools; there is no unified "write a review" call:

1. `mcp__github__create_pending_pull_request_review` — pass `owner`, `repo`, `pullNumber`, `commitID` (head SHA)
2. `mcp__github__add_comment_to_pending_review` — one call per finding that maps to a file + line **in the diff**; `side: "RIGHT"` for new/changed lines. Findings on lines outside the diff go in the summary body instead
3. `mcp__github__submit_pending_pull_request_review` — the aggregated summary plus the event

Pass `owner`, `repo`, and `pullNumber` **explicitly on every call**. Do not rely on event-context defaults: the same prompt runs from `workflow_dispatch`, where there is no PR event payload.

**Submit event:**

| Condition | Event |
|-----------|-------|
| Any Critical findings | `REQUEST_CHANGES` |
| Important / Suggestions only, or a clean pass | `COMMENT` |
| Ever | **never `APPROVE`** |

The bot is advisory. A clean review is still submitted as `COMMENT` stating that no blocking issues were found. Only humans approve — that is what keeps Gate 2 a human gate.

**No AI attribution.** No "Generated with Claude Code" in comments, inline notes, or the review body. Review comments are team communication; the tooling that produced them is not the point.

---

## 7. Hardening

Two classes of problem, both easy to get wrong.

### 7.1 — Untrusted input in shell

**PR titles and branch names are attacker-controlled.** Anyone who can open a PR controls them. Interpolating `${{ github.event.pull_request.title }}` directly into a `run:` block is a shell-injection vector — and breaks on the first title containing a quote.

Pass them through `env:` and substitute with **bash parameter expansion**, not `sed`:

```yaml
- name: Build review prompt
  env:
    REPO: ${{ github.repository }}
    PR_NUMBER: ${{ github.event.pull_request.number }}
    PR_TITLE: ${{ github.event.pull_request.title }}
    BASE_BRANCH: ${{ github.event.pull_request.base.ref }}
    HEAD_BRANCH: ${{ github.event.pull_request.head.ref }}
    FILES_CHANGED: ${{ needs.check-size.outputs.file_count }}
  run: |
    set -euo pipefail
    prompt=$(cat .github/prompts/pr-review.md)
    prompt=${prompt//__REPO__/$REPO}
    prompt=${prompt//__PR_NUMBER__/$PR_NUMBER}
    prompt=${prompt//__BASE_BRANCH__/$BASE_BRANCH}
    prompt=${prompt//__HEAD_BRANCH__/$HEAD_BRANCH}
    prompt=${prompt//__FILES_CHANGED__/$FILES_CHANGED}
    prompt=${prompt//__PR_TITLE__/$PR_TITLE}   # free-text — substitute LAST
    delim="PROMPT_EOF_$(openssl rand -hex 16)"
    {
      printf 'CLAUDE_PROMPT<<%s\n' "$delim"
      printf '%s\n' "$prompt"
      printf '%s\n' "$delim"
    } >> "$GITHUB_ENV"
```

Three details that each fix a real failure:

- **Parameter expansion, not `sed`.** The placeholder tokens are fixed literals with no glob metacharacters, so the match is literal — no regex escaping, no delimiter collision when a title contains `/` or `|`.
- **`__PR_TITLE__` substituted last.** It is free-text; substituting it first would let a title containing `__PR_NUMBER__` clobber a not-yet-expanded token.
- **Randomized heredoc delimiter.** A fixed sentinel like `EOF` can be matched by a line inside the prompt body, breaking out into arbitrary `GITHUB_ENV` entries. A per-run random delimiter cannot be collided with. Apply the same pattern to any free-text `GITHUB_OUTPUT` value (e.g. the PR title in the manual workflow).

Also pin the shell. On self-hosted runners the default `run:` shell can fall back to `dash`, where `set -o pipefail` is invalid:

```yaml
defaults:
  run:
    shell: bash
```

### 7.2 — Least privilege

```yaml
permissions:
  contents: read          # checkout + git diff
  pull-requests: write    # post the review
  actions: read
  id-token: write         # OIDC, if the action needs it
```

Nothing more. The credential lives in `secrets.CLAUDE_CODE_OAUTH_TOKEN` (or an API key / cloud-provider equivalent). Pin the action to an exact version — `anthropics/claude-code-action@v1.0.158`, not `@v1` — so a review-behaviour change arrives as a reviewed PR, not overnight.

---

## 8. Failure posture — fail open, always

**The review job must never block CI.** Provider rate limits and transient outages are normal operating conditions; a PR that cannot merge because a third-party LLM was busy is a worse outcome than a PR merged without an AI review pass.

```yaml
- name: Run Claude review
  id: claude_review
  continue-on-error: true
  uses: anthropics/claude-code-action@v1.0.158
```

`continue-on-error` alone is not enough — a silently-skipped review reads exactly like a clean review. Pair it with a **sticky degraded-mode comment**, keyed on an HTML marker so it updates in place instead of accumulating:

- **On failure** — find a comment containing `<!-- claude-pr-review-degraded -->`; update it if present, create it if not. Say plainly that the review did not complete, that the job was treated as best-effort and did not block CI, and that re-running after the provider recovers will produce a review.
- **On success** — find the same marker and **delete** it. A stale "review failed" banner on a PR that has since been reviewed is worse than no banner.
- **Always** — write the outcome to `$GITHUB_STEP_SUMMARY` so the failure is visible in the Actions UI without opening logs.

Use a distinct marker per workflow (`claude-pr-review-degraded` vs `claude-pr-review-manual-degraded`) so the auto and manual variants don't fight over the same comment.

---

## 9. Adoption checklist

1. **Write the checklist first.** `.claude/prompts/pr-review-checklist.md`. Start from the sections in §2 and fill each with rules your team has actually agreed on. An empty or aspirational checklist produces a generic review, and a generic review is not why you configured this reviewer. Seed it from real PR comments and post-incident findings.
2. **Wire the local gate to the same file.** Point the Step 3.5 `code-reviewer` subagent at the checklist before touching CI. This validates the checklist against real diffs at zero CI cost, and gets the team used to the severity vocabulary.
3. **Write the prompt template.** `.github/prompts/pr-review.md` with `__REPO__`, `__PR_NUMBER__`, `__PR_TITLE__`, `__BASE_BRANCH__`, `__HEAD_BRANCH__`, `__FILES_CHANGED__` placeholders. Include the context-gathering step (§4b), the domain-area map for your repo's module paths, a pointer to the checklist as the single source of truth, the severity buckets, and the posting instructions (§6).
4. **Add the secret.** `CLAUDE_CODE_OAUTH_TOKEN` in repository secrets.
5. **Commit the workflow.** `.github/workflows/claude-pr-review.yml` — triggers (§3), size gate (§5.1), skip notice (§5.2), prompt build (§7.1), the action with `continue-on-error` (§8), degraded-mode handling (§8).
6. **Create the `bot-review` label** for draft opt-in.
7. **Smoke-test on a small real PR.** Confirm: inline comments land on the right lines; `REQUEST_CHANGES` fires when a Critical is present and `COMMENT` when it is not; no `APPROVE` ever; no AI attribution in the body.
8. **Test the second pass.** Push a fix, reply `❌ Disagreed` on one finding, push again. The bot should not re-raise the disagreed finding or the fixed one. If it does, revisit §4 before rolling out — a bot that re-litigates settled threads gets muted within a week.
9. **Add the manual workflow** (§5.4) once the auto path is stable.
10. **Update branch protection** — the review job is not a required check (§8). Gate 2 stays: CI green + AI reviewer findings addressed + human approval.

---

## 10. Operating rules

- **Critical findings must be resolved before merge** — fixed, or dismissed with a one-line reason in the thread. Important and Suggestion findings are the author's judgement call.
- **Mark every thread** with `✅ Fixed`, `❌ Disagreed`, or `⏭ Deferred`. This is not politeness; §4 depends on it mechanically.
- **A finding the bot gets wrong is a checklist bug.** Fix the checklist in the same PR — that is the whole point of committing it. Do not add exceptions to the workflow YAML.
- **The bot never blocks the merge button.** If it is down, note it in the PR and proceed on human review (plus whatever else the project has configured).
- **Review the checklist quarterly.** Rules accumulate; some stop being true after a refactor. A checklist nobody prunes produces findings nobody reads.

---

## Reference implementation

**Maestro Command Center** — NestJS backend, React 18 frontend, Nx monorepo.

| File | Notes |
|------|-------|
| `.github/workflows/claude-pr-review.yml` | Auto workflow — 70-file ceiling, `bot-review` draft opt-in, conditional `track_progress`, self-hosted ARM64 fleet, 30-min timeout |
| `.github/workflows/claude-pr-review-manual.yml` | `workflow_dispatch` variant — 150-file ceiling, 60-min timeout, refreshes prompt + checklist from `origin/main` |
| `.github/prompts/pr-review.md` | 84-line prompt template with a 15-entry domain-area map |
| `.claude/prompts/pr-review-checklist.md` | 22 sections (A–V), ~285 lines, shared with the local pre-PR gate via a 6-slice dimension map |

---

## Related

- [PROCESS.md → Step 4: Code Review](./PROCESS.md#step-4-code-review)
- [PROCESS.md → `code-reviewer` subagent](./PROCESS.md#code-reviewer) — the local pre-PR gate that shares the checklist
- [PROMPTS.md → Self-review before PR](./PROMPTS.md#self-review-before-pr) — the severity vocabulary this bot matches
- [QUALITY-GATES.md → Gate 2: PR Merge](./QUALITY-GATES.md#gate-2-pr-merge)
- [Phase 6 → Step 4: CI/CD Pipeline Setup](../06-cicd-devops/PROCESS.md#step-4-cicd-pipeline-setup) — where `@claude` fix-on-mention and the rest of the pipeline live
- [Phase 5 → Security review on PRs](../05-security/PROCESS.md) — `anthropics/claude-code-security-review`, a separate action with a separate remit
