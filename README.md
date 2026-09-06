# reusable-workflows

Reusable GitHub Actions workflows, called across repositories.

Each workflow here is generic: every value identifying a project, an identity
or a cloud provider is an input, so a workflow describes a mechanism and knows
nothing about the repositories that call it.

## `terraform-job.yaml`

Runs a Terraform plan on a pull request, comments the plan on it, and applies
on a push to the default branch. Authenticates to Google Cloud with Workload
Identity Federation — no long-lived keys.

```yaml
jobs:
  terraform:
    uses: OWNER/reusable-workflows/.github/workflows/terraform-job.yaml@COMMIT_SHA
    with:
      working-directory: environments/prod
      workload-identity-provider: ${{ vars.WORKLOAD_IDENTITY_PROVIDER }}
      deploy-service-account: ${{ vars.DEPLOY_SERVICE_ACCOUNT }}
      plan-service-account: ${{ vars.PLAN_SERVICE_ACCOUNT }}
      terraform-version: "1.15.8"
```

| Input | Required | Purpose |
| --- | --- | --- |
| `working-directory` | yes | Terraform root to plan or apply |
| `workload-identity-provider` | yes | Full WIF provider resource name |
| `deploy-service-account` | yes | Identity for apply, and for plan when no plan identity is given |
| `plan-service-account` | no | Read-only identity for plans on a pull request; also switches the plan to `-lock=false` |
| `project-id` | no | Cloud project id, for logging and auth context |
| `terraform-version` | no | Defaults to `1.15.8` |
| `label` | no | Heading for the plan comment; defaults to the working directory |

**Pin to a commit SHA, not a branch.** A mutable ref means a change here
reaches every caller at once with none of their CI having run against it.

**Set the plan identity where the deploy identity cannot be impersonated from a
pull request** — for example where its trust policy is pinned to the default
branch — or wherever a pull request should not hold write credentials.

**No access configuration is needed.** Workflows in a public repository are
callable from any repository, so the access policy that a private repository
would require does not apply here — GitHub rejects setting one at all.
