# GitHub Action - Terraform Team Auth

This action is a great start to any Terraform workflow. Using a User or Org API token, it dynamically creates a short-lived Team Token that can be used with official Terraform workflows.

## Inputs

Below are the action's inputs that need to be defined in the Action's `with` block.

| Required | Input       |  Summary |
|----------|-------------|---------|
| [x] | `api-token` | An **organization** or **user** API token, see [Api Docs - Authentication](https://developer.hashicorp.com/terraform/cloud-docs/api-docs#authentication) |
| [x] | `team-name` | The name of the team to generate the Team Token for. |
| [] | `export` | (default: true) Enables or disables the `TF_API_TOKEN` environment variable. |

## Outputs

| Output | Summary |
|----------|--------|
| `team_token` | The value of the token to use for downstream requests. This needs to be used for `TF_API_TOKEN` environment variable in most scenarios. |
| `description` | The description used when requesting the team token, uses the format "GH Actions - $($env:GITHUB_REPOSITORY)" |
| `createdat` | Timestamp when the token was created. |
| `expiredat` | Timestamp of when the token expires. |

> [!INFORMATION]
> Unless disabled, the `team_token` value is automatically exported in the `TF_API_TOKEN` environment variable. 

```yaml
  - name: Generate a Team Token
    id: get-team-token
    uses: LanceMcCarthy/terraform-team-auth@v1
    with:
      api-token: ${{secrets.ORG_OR_USER_API_KEY}}
      team-name: "my-team"

  - name: Confirm Result
    run: |
      echo "------ENV VAR------"
      echo "TF_API_TOKEN variable: ${{env.TF_API_TOKEN}}"

      echo "------OUTPUTS------"
      echo "team_token: ${{steps.get-team-token.outputs.team_token}}"
      echo "description: ${{steps.get-team-token.outputs.description}}"
      echo "createdat: ${{steps.get-team-token.outputs.createdat}}"
      echo "expiredat: ${{steps.get-team-token.outputs.expiredat}}"
```

## Examples

Here is an example of the action's real usefulness in a workflow. Usign your Org API token, you can request a Team Token that is used in subsequent actions.

```yaml
name: 'Terraform Plan'

on:
  workflow_dispatch:
  pull_request:

permissions:  
  contents: read        # required by hashicorp/*
  pull-requests: write  # required by hashicorp/*

env:
  TF_CLOUD_ORGANIZATION: "my-org1"  # required by hashicorp/*
  TF_WORKSPACE: "my-tf-workspace"   # required by hashicorp/*
  CONFIG_DIRECTORY: "./"            # required by hashicorp/*
  # Automatically set by LanceMcCarthy/terraform-team-auth@v1
  # TF_API_TOKEN: !dont-set!        # required by hashicorp/*

jobs:
  terraform:
    name: "Terraform Plan"
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Generate a Team Token
      uses: LanceMcCarthy/terraform-team-auth@v1
      with:
        api-token: ${{secrets.ORG_OR_USER_API_KEY}}
        team-name: "my-team"

    # Continue with https://developer.hashicorp.com/terraform/tutorials/automation/github-actions

    - name: Upload Configuration
      uses: hashicorp/tfc-workflows-github/actions/upload-configuration@v1.0.0
      id: plan-upload
      with:
        workspace: ${{env.TF_WORKSPACE}}
        directory: ${{env.CONFIG_DIRECTORY}}
        speculative: true

    - name: Create Plan Run
      uses: hashicorp/tfc-workflows-github/actions/create-run@v1.0.0
      id: plan-run
      with:
        workspace: ${{ env.TF_WORKSPACE }}
        configuration_version: ${{ steps.plan-upload.outputs.configuration_version_id }}
        plan_only: true

    - name: Get Plan Output
      uses: hashicorp/tfc-workflows-github/actions/plan-output@v1.0.0
      id: plan-output
      with:
        plan: ${{ fromJSON(steps.plan-run.outputs.payload).data.relationships.plan.data.id }}
```
