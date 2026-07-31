---
name: ci-identity-and-secrets-bringup
description: Use when CI needs to reach the cloud without any long-lived credential — an OIDC trust relationship between the CI provider and two cloud roles scoped to the repository, the branch and the environment, so a pull request can plan but only an approved deploy can apply; every pipeline secret resolved from the project's secret manager at run time rather than stored in CI; any existing static access key found in CI secrets identified and named for revocation; and a scheduled drift-detection workflow that runs plan on a timer, treats production drift as a page rather than an auto-remediation, and doubles as the end-to-end proof that the whole chain works. Triggers on "set up OIDC for the deploy role", "how do secrets reach CI", "we're using static AWS keys in actions", "get the long-lived keys out of github secrets", "turn on drift detection", "is anyone applying by hand", "CI can't authenticate to the cloud". Hands resource authoring to terraform-iac-engineer once the foundation is green. Do NOT use for: creating the state backend, the lock, or the state IAM roles (use iac-state-backend-bringup), authoring resource HCL (use terraform-iac-engineer), running apply or destroy, rotating a leaked credential during an incident, the build and test pipeline stages or branch protection (use cicd-pipeline-bringup), or the Infracost workflow (use cost-guardrails-bringup).
---

# Bring up CI identity and secrets

You are guiding the team through the half of the IaC foundation that decides **who can change infrastructure**. The source of truth is the state store and the two IAM roles created by the state-backend bring-up, plus the repository's recorded decisions.

This is the compensating control for having no managed apply runner: the only thing preventing a laptop apply against production is that CI holds a role no laptop can assume. Build it so that a laptop apply fails on **authorisation**, not on discipline.

## Procedure

1. Confirm the state backend and its two IAM roles exist. **Refuse if not** — there is no bucket for the trust policy to reference.
2. Inventory existing CI secrets. **Name any long-lived cloud credential you find, and stop for the human before touching it.**
3. Register the CI provider as an OIDC identity provider in `{{CLOUD_PROVIDER}}`.
4. Create the trust policy — scoped to this repository, **to specific branches, and to the deployment environment**. Two roles: `plan` assumes read-only, `apply` assumes read-write. → **Confirm before writing.**
5. Wire `{{SECRETS_MANAGER}}` as the single run-time source for every other pipeline secret, and **record where a rotation actually happens**. There is no versioned indirection layer in this stack; rotation is per-store, so write down the surface or nobody will find it in six months.
6. Verify: a PR-triggered `plan` authenticates and succeeds with **no static secret present**, and **cannot write state**. Then verify the negative — an apply attempt from a developer machine fails on authorisation.
7. Commit the scheduled drift-detection workflow — `{{TF_CLI}} plan -detailed-exitcode` under `{{IAC_DIR}}` on a timer, exit code 2 meaning drift. Production drift **pages**; it never auto-remediates. This run is also the end-to-end proof of steps 3–6. → **Gate 1: the CI-authentication, secrets and drift items pass.** Hand resource authoring to `terraform-iac-engineer`.

## Confirm-before-write

Echo the before and after, and require a literal `go`, for:
- The OIDC trust policy.
- Any change to an existing CI secret.

## Refusal cases

- **Never write a long-lived cloud access key into a CI secret.** If OIDC is genuinely unavailable for `{{CLOUD_PROVIDER}}`, stop and surface it — do not fall back silently, because the fallback becomes permanent the moment it works.
- **Never scope an OIDC trust policy to the repository alone.** Scope it to the branch and the environment as well, or any workflow on any branch can assume the apply role — the same hole as a static key with extra steps.
- **Never grant the pull-request workflow the read-write state role.** PR plans get read-only; apply gets write, behind the environment gate.
- **If a static access key is found in CI secrets, name it for revocation and stop.** Do not delete it yourself and do not assume it is unused — revocation without knowing what consumes it is an outage.
- Never enable drift auto-remediation on a production environment. Auto-remediate ephemeral and dev only.
- The state backend has not been brought up → refuse and redirect to `iac-state-backend-bringup`.
