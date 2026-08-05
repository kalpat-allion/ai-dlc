---
name: cicd-pipeline-bringup
description: Use when a repository needs its GitHub Actions delivery pipeline stood up or brought up to standard — generating only the workflow stages the project's stack justifies, wiring the @claude fix-on-mention workflow with an exact pinned action version and a bot-actor gate and a spend cap, committing and dry-running the four starter agentic workflows, resolving every secret through the secret manager with OIDC cloud authentication, and setting branch protection — then walking the pipeline checklist to a pass. Triggers on "set up the GitHub Actions workflow for this repo", "generate our build and test workflow", "wire branch protection", "our pipeline is missing stages", "add the coverage gate", "wire @claude fix-on-mention", "add the deploy gate to staging". Wires @claude fix-on-mention only — auto-review-on-PR-open is a separate recorded decision and is deliberately not committed here. Two rules are absolute and a generator will violate both: the AI review job is never a required status check, and no workflow that writes to a branch is ever granted pull_request_target. Do NOT use for: GitLab CI projects, choosing the AI pull-request reviewer or building its review policy, adding auto-review-on-PR-open to the @claude workflow, authoring application tests, the production deploy gate or rollback wiring (use deploy-and-rollback-bringup), the state backend (use iac-state-backend-bringup) or the OIDC trust policy and secret-manager wiring (use ci-identity-and-secrets-bringup), or the Infracost workflow (use cost-guardrails-bringup).
---

# Bring up the delivery pipeline

You are guiding the team through standing up the delivery pipeline. The source of truth is the repository's existing workflows, its conventions file, and its recorded stack decisions.

Extend what exists; never silently replace it.

## Procedure

1. Inventory `.github/workflows/`. Classify each as keep / extend / replace, and **never drop a stage someone added deliberately** without surfacing it.
2. Generate only the justified stages, in order: lint/format (< 2 min) → build → tests with the coverage gate on new code (`{{TEST_COMMAND}}`) → security scans matching the project's active tools → `plan` on infrastructure PRs → deploy to staging on merge → E2E against staging → load test on tagged release candidates. **Skip any stage the inputs do not justify and state why in a comment** — an unjustified stage is worse than a missing one, because it gets disabled and stays disabled. **Prove the coverage gate rather than trusting `{{TEST_COMMAND}}`:** confirm the filled command actually emits coverage and that the job **fails** below the threshold, by observing one failing run. Filled without its coverage flag, the command passes every PR and the gate reports green — a pass made reachable that should not be, with nothing downstream to contradict it.
3. Wire `@claude` **fix-on-mention only** as its own workflow: triggers on `issue_comment` and `pull_request_review_comment`, exact pin `{{CLAUDE_ACTION_VERSION}}`, narrowest `permissions:`, bot-actor gate, concurrency cancellation, timeout, spend cap. Never `pull_request_target`.
   **Do not commit an auto-review-on-PR-open trigger in this workflow.** Whether an AI reviews every PR — and which reviewer — is a recorded project decision, not part of pipeline bring-up. A review-mode block added here would be a bootstrap with no size gate, no re-review idempotency and no fail-open handling, and teams mistake it for the production reviewer precisely because it arrived with the pipeline. If the project has chosen an AI reviewer, wire it from its own policy document as a separate workflow.
4. Commit the four starter agentic workflows — stale-issue triage, release notes, flaky-test repair, lint-debt fix — adjusting only cron, limits and project phrasing. Dry-run each. (These four are shipped alongside this template rather than inlined, because inlining them would put the procedure past its length limit; paste them into `.github/agentic-workflows/` at instantiation.)
5. Resolve every secret through the secret manager; cloud authentication is OIDC. No credential literals in workflow YAML.
6. Configure caching and parallelism until the PR pipeline is under 15 minutes.
7. Set branch protection on the default branch: PR + passing checks + one human approval + the issue identifier in the title; two humans for auth, migration and IAM changes. **The AI review job is not among the required checks.**
8. Walk the pipeline checklist item by item and report each as pass or fail. → **the pipeline gate: CI/CD pipeline operational.**

## Refusal cases

- **Never make the AI review job a required status check.** It depends on a third-party provider; a rate-limit spike would freeze the merge queue. Acknowledging a review is a human responsibility, and enforcement by required-check is how teams end up merging with `--admin`.
- **Never grant `pull_request_target` to any workflow that writes to a branch or reads a secret.** No exception, no "just for forks".
- Never pin an action to a floating major tag. Exact version, so a behaviour change arrives as a reviewed PR rather than a Monday outage.
- Never add a scanner, stage or deploy step the inputs do not justify.
- Never write a credential literal into workflow YAML.
- `{{DEPLOY_TARGET}}` is still an unfilled placeholder, or is otherwise unknown → refuse; half the pipeline depends on it, and an unfilled placeholder means nobody chose.
- Asked to build the AI reviewer's *review policy*, or to add auto-review-on-PR-open to the `@claude` workflow → refuse. This skill wires the CI plumbing and fix-on-mention; a separate recorded decision picks the reviewer and a separate document owns what it looks for. Offer the fix-on-mention workflow and name the gap rather than shipping a bootstrap reviewer.
