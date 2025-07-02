# Setup CFS

Action that simplifies the setup for Azure DevOps Central Feed Service. It does several things to assist with using CFS in an Actions Workflow.

1. By default, it assumes you are using OIDC in your workflow to authenticate with Azure.
It will then obtain an access token for Azure DevOps that can be used to authenticate to CFS. It sets this token in an environment variable named `CFS_ACCESSTOKEN`. The name of this variable can be configured using the `variable-name` input option.
2. Optionally, you can pass an ADO PAT in the `cfs-auth-token` input and the value of the PAT will be stored in the `CFS_ACCESSTOKEN` variable.
3. If `nuget: true` is specified it will install [Artifacts Credential Provider](https://github.com/microsoft/artifacts-credprovider) and setup the environment variables it needs to authenticate your feed based on the previous items.
4. If `nuget-config: true` is specified it will setup a `nuget.config` file to use your feed. This should only be necessary if you do not have this file in version control.
5. If `npm-config: true` is specified it will configure `.npmrc` to use the access token based on the previous items.


## Example Usage for Nuget

```yaml
    - name: "Setup CFS"
      uses: microsoft/setup-cfs@main
      with:
        feed-url: https://pkgs.dev.azure.com/contoso/MyFeed/_packaging/MyFeed/
        nuget: true
        nuget-config: true
        nuget-config-path: nuget.config
```


## Example Usage for NPM

```yaml
    - name: "Setup CFS"
      uses: microsoft/setup-cfs@main
      with:
        feed-url: https://pkgs.dev.azure.com/contoso/MyFeed/_packaging/MyFeed/
        npm-config: true
```

## Example Usage for Repository Already Configured for CFS

This shows how to configure the environment variable that will contain the access token.
This example assumes you have a `.npmrc` already configured to read the access token from an environment variable named `NPM_TOKEN`. This value does not need to be encoded if your `.npmrc` is using the variable with `_authToken`. However if you need to use the token with `_password` then it does need to be Base64 encoded and there is an option provided to do this.

```yaml
    - name: "Setup CFS"
      uses: microsoft/setup-cfs@main
      with:
        variable-name: NPM_TOKEN
        encode-variable: true
```

## Complete Example with OIDC Authentication

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write   # Required for OIDC
      contents: read

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Azure Login
      uses: azure/login@v1
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Setup CFS
      uses: microsoft/setup-cfs@main
      with:
        feed-url: https://pkgs.dev.azure.com/myorg/myproject/_packaging/myfeed/
        cfs-auth-token: oidc
        nuget: true
        nuget-config: true
        npm-config: true

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'

    - name: Restore NuGet packages
      run: dotnet restore

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'

    - name: Install npm packages
      run: npm install
```

## Example with Personal Access Token

```yaml
    - name: Setup CFS with PAT
      uses: microsoft/setup-cfs@main
      with:
        feed-url: https://pkgs.dev.azure.com/myorg/_packaging/myfeed/
        cfs-auth-token: ${{ secrets.AZURE_DEVOPS_PAT }}
        nuget: true
        npm-config: true
```

## Input Parameters

| Parameter | Description | Required | Default | Example |
|-----------|-------------|----------|---------|---------|
| `feed-url` | Azure DevOps Artifacts feed URL | No | `https://pkgs.dev.azure.com/` | `https://pkgs.dev.azure.com/myorg/myproject/_packaging/myfeed/` |
| `variable-name` | Environment variable name for the auth token | No | `CFS_ACCESSTOKEN` | `NPM_TOKEN` |
| `encode-variable` | Base64 encode the token value | No | `false` | `true` |
| `cfs-auth-token` | Authentication method or PAT token | No | `oidc` | `oidc` or `${{ secrets.PAT }}` |
| `nuget` | Install NuGet Artifacts Credential Provider | No | `false` | `true` |
| `nuget-config` | Create nuget.config file | No | `false` | `true` |
| `nuget-config-path` | Path for nuget.config file | No | `nuget.config` | `src/nuget.config` |
| `npm-config` | Create .npmrc file | No | `false` | `true` |

## Environment Variables Set

| Variable | Description | Condition |
|----------|-------------|-----------|
| `CFS_ACCESSTOKEN` (or custom name) | Azure DevOps access token | Always |
| `VSS_NUGET_ACCESSTOKEN` | NuGet access token | When `nuget: true` |
| `VSS_NUGET_URI_PREFIXES` | NuGet URI prefixes | When `nuget: true` |

## Troubleshooting

### Common Issues

1. **OIDC Authentication Fails**
   - Ensure Azure CLI is available in the runner
   - Verify OIDC is configured correctly in your workflow
   - Check that the Azure identity has access to Azure DevOps

2. **NuGet Restore Fails**
   - Verify the feed URL format
   - Check that the Artifacts Credential Provider is installed
   - Ensure the token has package read permissions

3. **npm Install Fails**
   - Verify the .npmrc configuration
   - Check that the feed supports npm packages
   - Ensure the token is properly encoded for npm

### Getting Help

- Review the [Security Guide](SECURITY.md) for best practices
- Check the GitHub Actions logs for detailed error messages
- Verify your Azure DevOps permissions and feed configuration
