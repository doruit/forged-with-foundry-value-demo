# VAL-PRE-001 value hypothesis gate — real CI check + CD/OIDC demo

Temporary standalone companion repo for
[doruit/forged-with-foundry](https://github.com/doruit/forged-with-foundry)'s
[`VAL-PRE-001_value_hypothesis_missing`](https://github.com/doruit/forged-with-foundry/tree/main/controls/value_adoption_and_finops/VAL-PRE-001_value_hypothesis_missing)
control.

It exists only to prove the optional GitHub Actions + OIDC deployment path
against a real Azure subscription.

## What this proves

1. **CI check (Gate 1)** — `scripts/validate_value_hypothesis.py --enforce`
   validates the deployment input before any Azure call. An incomplete
   hypothesis fails with no Azure credentials used.
2. **Validated state flows into CD** — the CI job exposes
   `valueHypothesisStatus` and `businessCaseId` as job outputs. The deployment
   job has `needs: ci-check` and uses those exact outputs; it does not hardcode
   `complete` or re-create trusted metadata independently.
3. **Azure Policy is presence-only enforcement** — Azure evaluates the reduced
   deployment metadata and denies invalid/missing values. It never reads
   `agent.yaml`, cannot prove the CI check ran, and cannot distinguish genuine
   metadata from forged valid-looking metadata.

No client secret is stored. The Azure deployment identity is obtained through
the repository's `production` GitHub Environment and Microsoft Entra Workload
Identity Federation (OIDC). Environment protection rules are separate
production hardening and are not implied by the Environment name alone.

## Setup

The `production` GitHub Environment has these Actions variables:

| Variable | Value |
|---|---|
| `AZURE_CLIENT_ID` | Client ID of the demo managed identity |
| `AZURE_TENANT_ID` | Azure tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure subscription ID |
| `AZURE_RESOURCE_GROUP` | `rg-forged-with-foundry` |

The federated credential must trust the exact OIDC subject presented by this
repository/environment. Current GitHub repositories may use the immutable
owner/repository ID form; see the main control's `docs/OIDC-DEMO.md` for the
subject-format details.

## Run it

Actions → **VAL-PRE-001: CI check + CD deployment gate demo** → **Run workflow**.

Expected:

- `ci-check` validates the complete fixture and proves the incomplete fixture fails;
- `cd-deploy-gate-2-allowed` consumes the exact CI outputs and deploys successfully;
- `cd-deploy-gate-2-backstop` skips Gate 1 and proves Azure Policy still denies invalid metadata.
