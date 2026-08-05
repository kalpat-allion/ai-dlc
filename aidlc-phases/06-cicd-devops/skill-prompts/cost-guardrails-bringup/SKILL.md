---
name: cost-guardrails-bringup
description: Use when a repository needs its cost guardrails wired so infrastructure spend cannot surprise anyone — Infracost running on every infrastructure pull request and posting a single monthly-delta comment, cloud-native budget alerts at 50, 80 and 100 percent of the monthly target routed to the channel the team treats as on-call, a unit-economics metric (cost per active user, per request, or per inference) emitted to the existing dashboard, and a reviewer checklist for the three things Infracost cannot judge: a cost-bearing change with no justification in its linked issue, an instance sized past the stated performance target, and a resource missing the mandatory tags. Triggers on "wire cost comments on our PRs", "set up infracost", "we need budget alerts", "how much will this PR cost us", "nobody knows what we're spending", "set up cost per user tracking". Does not produce the cost estimate itself — that is a human-run estimation with a cited price source, and this skill refuses to invent one. Do NOT use for: producing or revising the monthly cost estimate, the quarterly cost-optimisation review, authoring infrastructure code (use terraform-iac-engineer), the state backend (use iac-state-backend-bringup) or the OIDC and secret-manager wiring (use ci-identity-and-secrets-bringup), Kubernetes-specific cost tooling, or approving a cost-increasing pull request.
---

# Bring up cost guardrails

You are guiding the team through wiring the four cost controls that are otherwise audited by hand weeks after the fact. The source of truth is the **recorded monthly budget target** and the conventions file's mandatory tag list.

Infracost owns the number. You wire the plumbing around it and the judgement it cannot make.

## Procedure

1. Confirm the monthly budget target exists and is recorded. **Refuse if not** — thresholds against an unknown denominator are decoration.
2. Commit the Infracost workflow scanning `{{IAC_DIR}}`: `infracost breakdown` on the base branch, `infracost diff` on the PR, one comment per PR with the monthly delta. Its credential resolves through the secret manager, never a literal.
3. Create `{{CLOUD_PROVIDER}}` budget alerts at 50 / 80 / 100% of target, routed to `{{ALERT_CHANNEL}}`. Surprise bills are incidents; route them where incidents go.
4. Emit one unit-economics metric to the existing dashboard — cost per active user, per request, or per inference. Pick **one**; it is the figure that survives a board conversation.
5. Commit the three-line reviewer checklist for what Infracost cannot judge:
   - a cost-bearing change with **no justification in its linked issue**;
   - an instance **sized past the stated performance target**;
   - a resource **missing the mandatory tags** (which also makes it invisible to cloud-native inventory).
   → **the infrastructure-foundation gate: the four cost items pass.** The monthly cost estimate is **not** one of them — it stays a human exercise against a cited price source, and this skill does not produce it.

## Refusal cases

- **Never produce, adjust, or "sanity-check" a cost figure.** Infracost prices the diff; the monthly estimate is a human-run exercise against a cited price source. A number you invent here would look gate-approved.
- **Never post two cost comments on one PR.** Infracost owns the number; the anomaly checklist is a reviewer aid, not a second comment. The wrong number is the one a reviewer will occasionally read.
- Never set a budget alert threshold above 100%, and never route one anywhere other than `{{ALERT_CHANNEL}}`.
- Asked to approve a cost-increasing PR → refuse; you wire the guardrail, a human makes the call.
