---
name: iac-state-backend-bringup
description: Use when a repository needs its Terraform or OpenTofu state backend stood up before any resource is authored — a remote backend on the project's own cloud with object versioning, encryption at rest, public access denied and a version lifecycle policy; native state locking; two least-privilege IAM roles on the state store, one read-only and one read-write, so a plan cannot mutate state; one root module per environment with per-environment state keys so no environment can plan against another's state; state and lock files asserted out of version control and verified not already tracked; and the cloud audit log confirmed to cover the state store. Triggers on "set up terraform for this repo", "wire the terraform backend", "we need remote state", "where is our state stored", "our terraform has no backend", "the state file is in git", "we're all running terraform off our laptops". Hands CI authentication and secrets to ci-identity-and-secrets-bringup once the backend is green. Do NOT use for: authoring resource HCL or modules (use terraform-iac-engineer), running apply or destroy, choosing between Terraform and OpenTofu or which cloud (those are recorded architecture decisions, not procedures), the OIDC trust policy or secret manager (use ci-identity-and-secrets-bringup), cost comments or budget alerts (use cost-guardrails-bringup), the build and test pipeline (use cicd-pipeline-bringup), or recovering corrupted state and releasing a stuck lock during an incident.
---

# Bring up the Terraform state backend

You are guiding the team through a one-time, high-blast-radius bring-up. The source of truth is the repository's **recorded infrastructure decisions** (which CLI, which cloud) and its **conventions file**. A backend mistake is the one error in this phase with no cheap undo, so every step below is verified rather than assumed.

## Procedure

1. Read the recorded decisions for cloud provider and `{{TF_CLI}}`. **Refuse if either is unrecorded** — guessing the CLI commits the team to a licence, and guessing the cloud commits them to a backend.
2. Decide and **write down** how the backend storage itself gets created: a committed one-time bootstrap script, or an explicitly documented manual prerequisite. This is step 2 because it is the step everyone skips, and the result is a team that cannot rebuild its own backend.
3. Create the state store for `{{CLOUD_PROVIDER}}` with **object versioning on**, **encryption at rest with a customer-managed key**, **public access denied**, and a version lifecycle policy so the versioned bucket does not grow without bound.
4. Enable **native state locking** — the lock table, the native lockfile, or the blob lease, per `{{CLOUD_PROVIDER}}`. Verify it by attempting two concurrent applies and observing the second block. An unverified lock is an assumption.
5. Create **two least-privilege IAM roles** on the state store: read-only for `plan`, read-write for `apply`. Record which principal assumes which. Never one role for both.
6. Write the `backend` block under `{{IAC_DIR}}`. → **Confirm before writing** (see below).
7. Establish one root module per environment with per-environment state keys. Verify no environment can read another's state.
8. Assert `*.tfstate`, `*.tfstate.backup`, `.terraform/` and `*.tfvars` in `.gitignore`, then run `git ls-files` to verify none is **already tracked**. A tracked state file is a credential leak, not a hygiene issue — if you find one, say so plainly and stop for the human.
9. Confirm the cloud audit log covers the state store. → **Gate 1: the state and repo-layout items pass.** Hand CI authentication and secrets to `ci-identity-and-secrets-bringup`.

## Confirm-before-write

Echo the before and after, and require a literal `go`, for:
- The `backend` block, and **any change to an existing one** — a backend edit re-homes state.
- The `.gitignore` change, if a state file is already tracked.

## Refusal cases

- The CLI or cloud decision is unrecorded → refuse; both are architecture decisions, not procedure steps.
- **Never create the backend by hand in a console and call the foundation reproducible.** Step 2's answer goes in writing or the step is not done.
- **Never create a single role with write access to state and hand it to plan-time workloads.** Two roles, always — a plan running with write credentials is an accident waiting for a typo.
- **Never enable a backend without object versioning.** Versioning is the entire state-recovery story, and retrofitting it does not recover the history already lost.
- A state file is already tracked in git → stop for the human. Removing it from the index does not remove it from history, and the credentials in it are already exposed.
- Asked to run `apply`, `destroy`, or state surgery → refuse; this skill builds the backend, it does not use it.
