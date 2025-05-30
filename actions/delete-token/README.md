# Action: Terraform Delete Team Token

This deletes a previously generated Team Token, intended to be used in conjunction with [LanceMcCarthy/terraform-generate-team-token](https://github.com/LanceMcCarthy/terraform-generate-team-token), but can be used in isolation.

## Inputs

| Input       | Required |  Summary |
|-------------|----------|----------|
| `api-token` | ✅ | Your Organization Token (can be created at https://app.terraform.io/app/YOUR_ORG_ID/settings/authentication-tokens) |
| `team-token-id`   | ✅ | The ID of the team token (usually provided by an earlier action) |

## Examples

```yaml
  # 1. Create token (see /actions/generate-token.README.md)

  # 2. do work with env.TF_API_TOKEN

  # 3. Delete step 1's token 
  - name: Delete a Team Token
    uses: LanceMcCarthy/terraform-team-auth/actions/delete-token@v1.0.0
    with:
      api-token: ${{secrets.ORG_OR_USER_API_KEY}}
      team-token-id: ${{env.TF_API_TOKEN_ID}} 
```

> IF you do not want to use the environment variable, you can use step 1's output `team-token-id: ${{steps.generate-token.outputs.team_token_id}}` 
