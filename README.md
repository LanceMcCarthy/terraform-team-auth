# GitHub Action - Terraform Team Auth

This action is a great start to any Terraform workflow. Using a User or Org API token, it dynamically creates a short-lived Team Token that can be used with official Terraform workflows.

## Inputs

| Required | Input       |  Summary |
|----------|-------------|---------|
| ✅ | `api-token` | Your Organization Token (can be created at https://app.terraform.io/app/YOUR_ORG_ID/settings/authentication-tokens) |
| ✅ | `team-id` | The team ID (e.g. `team-ABdgNT7rDDiBadPi3`). This can be found in the in URL of the team's page https://app.terraform.io/app/YOUR_ORG_ID/settings/teams/YOUR-TEAM-ID |
|  | `export` | (default: true) Enables or disables the `TF_API_TOKEN` environment variable. |

> [!WARNING]
> Treat your api-token like you would a password. While it does not have permission to perform plans and applies in workspaces. For more information, see the [organization service account documentation](https://developer.hashicorp.com/terraform/cloud-docs/users-teams-organizations/api-tokens#organization-api-tokens).

## Outputs

| Output | Summary |
|----------|--------|
| `team_token` | The value of the token to use for downstream requests. This needs to be used for `TF_API_TOKEN` environment variable in most scenarios. |
| `description` | The description used when requesting the team token, uses the format "GH Actions - (Repo Name)" |
| `createdat` | Timestamp when the token was created. |
| `expiredat` | Timestamp of when the token expires. |

```yaml
  - name: Generate a Team Token
    id: get-team-token
    uses: LanceMcCarthy/terraform-team-auth@v1
    with:
      api-token: ${{secrets.ORG_OR_USER_API_KEY}}
      team-id: "team-ABdgNT7rDDiBadPi3"

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

## Example

Here is an example of the action's initial position in a workflow; it gets a new Team Token for you, then use that token for all subsequent Terraform actions. 

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

    # Sets the TF_API_TOKEN environment variable
    - name: Generate a Team Token
      uses: LanceMcCarthy/terraform-team-auth@v1
      with:
        api-token: ${{secrets.ORG_OR_USER_API_KEY}}
        team-id: "team-ABdgNT7rDDiBadPi3"


    # Continue with Terraform actions
    # https://developer.hashicorp.com/terraform/tutorials/automation/github-actions

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
        workspace: ${{env.TF_WORKSPACE}}
        configuration_version: ${{steps.plan-upload.outputs.configuration_version_id}}
        plan_only: true

    - name: Get Plan Output
      uses: hashicorp/tfc-workflows-github/actions/plan-output@v1.0.0
      id: plan-output
      with:
        plan: ${{fromJSON(steps.plan-run.outputs.payload).data.relationships.plan.data.id}}
```
