# Contribute to the GitHub Actions Security Knowledge Base

If you are the owner of a GitHub Action, please contribute information about the use of `GITHUB_TOKEN` and expected outbound calls for your Action. 

This will enable the community to:
1. Automatically calculate minimum token permissions for the `GITHUB_TOKEN` for their workflows. 
2. Restrict outbound traffic for their GitHub Actions workflows to allowed endpoints.

This will increase trust for your GitHub Action, more developers would be comfortable using it, and it will improve security for everyone's GitHub Actions workflows.

# How do I contribute to the knowledge base?

To contribute to the knowledge base:
1. Add a folder under the knowledge base folder for your GitHub Action.
2. In the folder for your GitHub Action, add an `action-security.yml` file. You can view existing files to understand the structure of these YAML files. See an example here [`knowledge-base/actions/checkout/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/actions/checkout/action-security.yml)
3. Add metadata in the `action-security.yml` file about the use of `GITHUB_TOKEN` and expected outbound traffic for your GitHub Action.

## Syntax for action-security.yml

The metadata filename must be `action-security.yml`. It must be located in a folder for your GitHub Action under the `knowledge-base` folder, e.g. the location for `actions/checkout` is [`knowledge-base/actions/checkout/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/actions/checkout/action-security.yml) The data in the metadata file defines the permissions needed for `GITHUB_TOKEN` for your Action and the outbound calls made by your Action.

## `name`

**Required** The name of your action, should be the same as in your GitHub Action's action.yml file

## `github-token`

**Optional** github-token allows you to specify where `GITHUB_TOKEN` is expected as input for your GitHub Action, what permissions it needs, and the reason for those permissions. If your Action does not use the `GITHUB_TOKEN`, you do not need to set this. 

## Example

This example is for `peter-evans/close-issue` GitHub Action. It shows that the Action expects GitHub token as an action input, the name of the input is `token`, and that it is set to `GITHUB_TOKEN` as the default value. It also shows that the permissions needed for the Action are `issues: write` and the reason for that permission is specified in the `issues-reason` key. 

[`knowledge-base/peter-evans/close-issue/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/peter-evans/close-issue/action-security.yml)

```
github-token:
  action-input:
    input: token
    is-default: true
  permissions:
    issues: write
    issues-reason: to close issues
```

## `github-token.action-input`

**Optional** github-token.action-input allows you to specify which input is used to accept the `GITHUB_TOKEN`. If your Action expects the token to be specified as an input to your Action, specify it in the `input` key. If you accept the `GITHUB_TOKEN` in environment variable, you do not need to set this. 

## `github-token.action-input.input`

**Required** github-token.action-input.input should be the name of the input in your `action.yml` file that accepts the `GITHUB_TOKEN`. This is required if `action-input` is set. 

## `github-token.action-input.is-default`

**Required** github-token.action-input.is-default This is required if `action-input` is set. It should be set to true, if `default` value of the input that expects the `GITHUB_TOKEN` is `${{ github.token }}`. 

## `github-token.environment-variable-name`

**Optional** If you expect the `GITHUB_TOKEN` to be set in an environment variable, specify the name of the environment variable in `environment-variable-name`. If you accept the `GITHUB_TOKEN` as an Action input, you do not need to set this. 

## Example

This example is for `github/super-linter` GitHub Action. It shows that the Action expects GitHub token as an environment variable, the name of the environment variable is `GITHUB_TOKEN`. It also shows that the permissions needed for the Action are `statuses: write` and the reason for that permission is specified in the `statuses-reason` key. 

[`knowledge-base/github/super-linter/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/github/super-linter/action-security.yml)

```
name: 'Super-Linter'
github-token:
  environment-variable-name: GITHUB_TOKEN
  permissions:
    statuses: write
    statuses-reason: to mark status of each linter run
```

## `github-token.permissions`

**Optional** If your Action uses the `GITHUB_TOKEN` provide information about the permissions needed using `github-token.permissions`. Each permission that is needed should be specified as a scope. If you only use the token to prevent rate-limiting, it needs `metadata` permissions, which the token has by default, so you do not need to provide additional scopes. 

## Example

This example is for `actions/setup-node` GitHub Action. It shows that the Action expects GitHub token as an Action input. The permissions key is set, but no scopes are defined, since it only uses it for rate-limiting. 

[`knowledge-base/actions/setup-node/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/actions/setup-node/action-security.yml)

```
name: 'Setup Node.js environment'
github-token:
  action-input:
    input: token
    is-default: true
  permissions: # Used to pull node distributions from node-versions. Metadata permissions is present by default
```

## `github-token.permissions.<scope>`

**Optional** If your Action uses the `GITHUB_TOKEN` and uses it for a scope other than `metadata`, provide what scope is needed. Valid scopes are documented [here](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token).

## `github-token.permissions.<scope>-reason`

**Optional** If your Action uses the `GITHUB_TOKEN` and uses it for a scope other than `metadata`, provide the reason you need the scope. This information gets added to a workflow when its permissions are calculated automatically using the knowledge base. This is required if the corresponding scope is set in the permissions. The reason must start with `"to "`. 

## Example

As an example, consider this `action-security.yml` for `peter-evans/close-issue` GitHub Action.

[`knowledge-base/peter-evans/close-issue/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/peter-evans/close-issue/action-security.yml)

```
github-token:
  action-input:
    input: token
    is-default: true
  permissions:
    issues: write
    issues-reason: to close issues
```

When a GitHub workflow uses `peter-evans/close-issue` GitHub Action, and it uses the knowledge base to set permissions automatically, this is the output. The `permissions` key is added to the job, and the reason is added in a comment. The reason is taken from the `github-token.permissions.<scope>-reason` value from the knowledge base. 

```
jobs:
  closeissue:
    permissions:
      issues: write # for peter-evans/close-issue@v1 to close issues
    runs-on: ubuntu-latest

    steps:
    - name: Close Issue
      uses: peter-evans/close-issue@v1
      with:
       issue-number: 1
       comment: Auto-closing issue
```

## `github-token.permissions.<scope>-if`

**Optional** If your Action uses the `GITHUB_TOKEN` but certain scopes are used only under certain conditions, the condition can be specified using `<scope>-if`.  

## Example

As an example, consider this `action-security.yml` for `dessant/lock-threads` GitHub Action. The `issues` scope only applies if either the `with` (action input) does not have `process-only` or `process-only` is set to `issues`. 

[`knowledge-base/dessant/lock-threads/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/dessant/lock-threads/action-security.yml)

```
github-token:
  action-input:
    input: github-token
    is-default: true
  permissions:
    issues: write
    issues-if: ${{ !contains(with, 'process-only') || with['process-only'] == 'issues' }}
    issues-reason: to lock issues
    pull-requests: write
    pull-requests-if: ${{ !contains(with, 'process-only') || with['process-only'] == 'prs' }}
    pull-requests-reason: to lock PRs
```

## `outbound-endpoints`

**Required** `outbound-endpoints` allows you to specify the outbound endpoints that your GitHub Action is expected to call and the reason for that call. If your GitHub Action does not make any outbound calls, you still need to set this, but it can be set to an empty value. This enables developers to restrict outbound traffic from their workflows using the [`Harden Runner`](https://github.com/marketplace/actions/harden-runner) GitHub Action. 

To know what outbound calls are made by your action, add the `Harden Runner` GitHub Action to a workflow that uses your GitHub Action. You can then see what outbound calls are being made. [Here](https://github.com/step-security/secure-workflows/blob/7a5b6afce236fa0634b8ecb46ebec180774c6aa2/.github/workflows/test.yml#L14) is an example workflow that uses the `actions/checkout` GitHub Action. The outbound calls made by `actions/checkout` GitHub Action are visible [here](https://app.stepsecurity.io/github/step-security/secure-workflows/actions/runs/1517151905). 

## Example

This example is for `actions/checkout` GitHub Action. It shows that the Action is expected to make outbount calls to `github.com` at port `443`, and the `reason` is `to fetch code from GitHub`. 

[`knowledge-base/actions/checkout/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/actions/checkout/action-security.yml)

```
name: 'Checkout'
outbound-endpoints:
  - fqdn: github.com
    port: 443
    reason: to fetch code from GitHub
```

## `outbound-endpoints[*].fqdn`

**Optional** A fully qualified domain name to which outbound calls are expected by your GitHub Action. The fqdn can have a wildcard `*`. 

## Example

This example is for `aws-actions/configure-aws-credentials` GitHub Action. It shows that the Action is expected to make outbount calls to `sts.*.amazonaws.com` at port `443`, and the `reason` is `to validate or fetch AWS credentials`. 

[`knowledge-base/aws-actions/configure-aws-credentials/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/aws-actions/configure-aws-credentials/action-security.yml)

```
outbound-endpoints:
  - fqdn: sts.*.amazonaws.com
    port: 443
    reason: to validate or fetch AWS credentials
```

## `outbound-endpoints[*].port`

**Required** For a domain name, the port to which outbound calls are expected by your GitHub Action.

## `outbound-endpoints[*].reason`

**Required** For a domain name, the reason the outbound call is being made. The reason must start with `"to "`

## Example

This example is for `actions/setup-node` GitHub Action. It shows that the Action is expected to make outbount calls to `nodejs.org` at port `443`, and the `reason` is `to download node distribution`. 

[`knowledge-base/actions/setup-node/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/actions/setup-node/action-security.yml)

```
name: 'Setup Node.js environment'
outbound-endpoints:
  - fqdn: nodejs.org
    port: 443
    reason: to download node distribution
```

When a GitHub workflow uses `actions/setup-node` and the `harden-runner` GitHub Action to discover and restrict outbound traffic, the reason is shown from this knowledge base. This helps users know that the outbound traffic is expected. 

<p align="left">
  <img src="https://step-security-images.s3.us-west-2.amazonaws.com/outboundkb.png" alt="Insights when running Harden Runner GitHub Action" >
</p>

## `harden-runner-link`

**Optional** If you discovered outbound traffic for your GitHub Action by using the `harden-runner` GitHub Action, add a link to the insights page here. This helps in confirming the outbound calls made by your Action. 

## Example

This example is for `ljharb/rebase` GitHub Action. It shows that the Action is expected to make outbount calls to `api.github.com` and `github.com` at port `443`. The `harden-runner-link` has a link to the insights when using the `harden-runner` GitHub Action in the workflow to discover outbound calls. The link can be found in the build logs.

[`knowledge-base/ljharb/rebase/action-security.yml`](https://github.com/step-security/secure-workflows/blob/main/knowledge-base/ljharb/rebase/action-security.yml)

```
name: 'GitHub action to automatically rebase PRs'
github-token:
  environment-variable-name: GITHUB_TOKEN
  permissions:
    contents: write
    contents-reason: to push code to rebase
    pull-requests: read
    pull-requests-reason: to get info about PR
outbound-endpoints:
  - fqdn: api.github.com
    port: 443
    reason: to call GitHub Pull Request API
  - fqdn: github.com
    port: 443
    reason: to push code to rebase
harden-runner-link: https://app.stepsecurity.io/github/varunsh-coder/actions-playground/actions/runs/1526918154
```
