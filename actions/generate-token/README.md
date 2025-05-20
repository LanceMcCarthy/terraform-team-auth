# Action: Terraform Generate Team Token

This action is a great start to any Terraform workflow. Using an Organization (or User) Token, it dynamically creates a short-lived Team Token that can be used with official Terraform plan workflows.

## Inputs

| Input       | Required |  Summary |
|-------------|----------|----------|
| `api-token` | ✅ | Your Organization Token (can be created at https://app.terraform.io/app/YOUR_ORG_ID/settings/authentication-tokens). |
| `team-id`   | ✅ | The team ID (e.g. `team-ABdgNT7rDDiBadPi3`). This can be found in the in URL of the team's page https://app.terraform.io/app/YOUR_ORG_ID/settings/teams/YOUR-TEAM-ID |
| `export`    |     | (default: true) Enables or disables the `TF_API_TOKEN` environment variable. |

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
    id: generate-token
    uses: LanceMcCarthy/terraform-team-auth/generate-token@v1
    with:
      api-token: ${{secrets.ORG_OR_USER_API_KEY}}
      team-id: "team-ABdgNT7rDDiBadPi3"

  - name: Confirm Result
    run: |
      echo "------ENV VAR------"
      echo "TF_API_TOKEN variable: ${{env.TF_API_TOKEN}}"
      echo "TF_API_TOKEN_ID variable: ${{env.TF_API_TOKEN_ID}}"

      echo "------OUTPUTS------"
      echo "team_token: ${{steps.generate-token.outputs.team_token}}"
      echo "team_token_id: ${{steps.generate-token.outputs.team_token_id}}"
      echo "description: ${{steps.generate-token.outputs.description}}"
      echo "createdat: ${{steps.generate-token.outputs.createdat}}"
      echo "expiredat: ${{steps.generate-token.outputs.expiredat}}"
```
