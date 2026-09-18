# VAL-PRE-001 value hypothesis gate — real CI/CD + OIDC demo

Temporary, standalone companion repo to
[doruit/forged-with-foundry](https://github.com/doruit/forged-with-foundry)'s
[`VAL-PRE-001_value_hypothesis_missing`](https://github.com/doruit/forged-with-foundry/tree/main/controls/value_adoption_and_finops/VAL-PRE-001_value_hypothesis_missing)
control. It exists only to run that control's optional GitHub Actions + OIDC
gate demo for real, without adding a real deployment-capable workflow to the
main repository. Everything here is a direct, unmodified copy of files
already implemented and documented in the main repo (see its README's
"Optional: run the real CI/CD + OIDC gate demo" section).

## What this proves

A real, manually triggered GitHub Actions workflow
(`.github/workflows/val-pre-001-value-gate-demo.yml`) authenticates to Azure
via Microsoft Entra Workload Identity Federation (OIDC, no client secret) and
demonstrates the same two independent gates as the main control:

1. **Gate 1 (CI/CD release gate)** — `scripts/validate_value_hypothesis.py --enforce`
   blocks a pipeline before any Azure call when `fixtures/agent.incomplete.yaml`
   is used.
2. **Gate 1 passes, Gate 2 (Azure Policy) allows** — the same validator passes
   against `fixtures/agent.yaml`, then a real `az deployment group create`
   against the shared `rg-forged-with-foundry` resource group succeeds because
   the policy-evaluated tags are `valueHypothesisStatus=approved` and a
   non-empty `businessCaseId`. The created resource is deleted at the end of
   the same job.
3. **Gate 2 backstop** — a deployment that skips Gate 1 entirely (tags built
   directly from the incomplete fixture) is still denied by Azure Policy with
   `RequestDisallowedByPolicy`.

No client secret is stored anywhere. The federated identity used here holds
only the built-in **Monitoring Contributor** role, scoped to the
`rg-forged-with-foundry` resource group, not `Contributor`.

## Setup

The "production" GitHub Environment in this repository (Settings →
Environments) has these Actions variables set:

| Variable | Value |
|---|---|
| `AZURE_CLIENT_ID` | Client ID of the `id-val-pre-001-github-oidc` managed identity |
| `AZURE_TENANT_ID` | The Azure AD tenant ID |
| `AZURE_SUBSCRIPTION_ID` | The Azure subscription ID |
| `AZURE_RESOURCE_GROUP` | `rg-forged-with-foundry` |

The federated credential trusts
`repo:doruit/forged-with-foundry-value-demo:environment:production`.

## Run it

Actions tab → "VAL-PRE-001: value hypothesis gate demo (optional, manual)" →
Run workflow.
