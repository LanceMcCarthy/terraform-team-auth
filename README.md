# Actions: Terraform Team Auth

These workflows are a benefit to anyone who needs to dynamically create short-lived credentials to enact Terraform Plans. See the specific README for instructions and examples.

1. README => [Action: Generate Team Token](https://github.com/LanceMcCarthy/terraform-team-auth/blob/main/actions/generate-token/README.md)
2. README => [Action: Delete Team Token](https://github.com/LanceMcCarthy/terraform-team-auth/blob/main/actions/delete-token/README.md)

## Combined Example

Be sure to visit the above docs first, but here's a realistic scenario of using both actions.

```yaml
name: 'Terraform Plan'

on:
  workflow_dispatch:

permissions:  
  contents: read        # required by hashicorp/*
  pull-requests: write  # required by hashicorp/*

env:
  TF_CLOUD_ORGANIZATION: "my-org1"  # required by hashicorp/*
  TF_WORKSPACE: "my-tf-workspace"   # required by hashicorp/*
  CONFIG_DIRECTORY: "./"            # required by hashicorp/*
  # TF_API_TOKEN:                   # DO NOT SET THIS, it wil be done by step 1

jobs:
  terraform:
    name: "Terraform Plan"
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    # Phase 1 - Create a new team token (exports TF_API_TOKEN and TF_API_TOKEN_ID environment variables)
    - name: Generate a Team Token
      uses: LanceMcCarthy/terraform-team-auth/actions/generate-token@v1.0.0
      with:
        api-token: ${{secrets.ORG_OR_USER_API_KEY}}
        team-id: "team-ABdgNT7rDDiBadPi3"


    # Phase 2. Invoke Terraform Plan using Hashicorp's actions
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


    # Phase 3. Delete the Team Token created for this workflow
    - name: Delete a Team Token
      uses: LanceMcCarthy/terraform-team-auth/actions/delete-token@v1.0.0
      with:
        api-token: ${{secrets.ORG_OR_USER_API_KEY}}
        team-token-id: ${{env.TF_API_TOKEN_ID}} 
```