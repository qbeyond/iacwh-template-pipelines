# iacwh-template-pipelines

Template pipelines to deploy and destroy Terraform infrastructure via Azure DevOps.

## Overview

This repository provides reusable Azure DevOps YAML pipeline templates for managing Terraform infrastructure lifecycles. Templates are designed to be referenced as a resource in your pipeline and support Workload Identity Federation, Entra ID authentication, flexible job injection, and multi-environment deployments.

## Usage

Reference this repository in your pipeline:

```yaml
resources:
  repositories:
    - repository: iacwh-template-pipelines
      type: github
      name: qbeyond/iacwh-template-pipelines
      endpoint: <your-github-service-connection>
      environment: "refs/tags/v1.3.1"
```

Then extend a template:

```yaml
stages:
  - template: terraform-deploy.yml@iacwh-template-pipelines
    parameters:
      AzureRMServiceConnection: "sc-my-subscription"
      AzureRMServiceConnectiontfState: "sc-terraform-backend"
      storage_account_name: "stterraformbackend01"
      container_name: "tfstate"
      key: "my-project/terraform.tfstate"
      environment: "dev"
      repository: self
```

---

## Templates

### `terraform-deploy.yml`

Runs a two-stage pipeline: **Plan** → **Deploy**. The Plan stage creates a speculative plan and publishes it as an artifact. The Deploy stage downloads the artifact and applies it against the target environment.

#### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `AzureRMServiceConnection` | string | — | Service Connection used for `terraform plan` and `terraform apply` |
| `AzureRMServiceConnectiontfState` | string | — | Service Connection used to authenticate against the Terraform remote state storage account |
| `storage_account_resource_group_name` | string | `rg-TerraformBackend-01` | Resource group of the Terraform state storage account |
| `storage_account_name` | string | — | Name of the storage account holding remote state |
| `container_name` | string | — | Blob container name for the remote state file |
| `key` | string | — | Path to the state file inside the container (e.g. `env/terraform.tfstate`) |
| `environment` | string | — | Azure DevOps environment name. Used as stage suffix and deployment gate |
| `repository` | string | — | Repository to check out. Use `self` for the calling repo |
| `variables` | object | `{}` | Terraform input variables passed to `terraform plan` |
| `terraformVersion` | string | `1.1.2` | Version of Terraform to install |
| `commandOptions` | string | `""` | Additional arguments appended to `terraform plan` and `terraform apply` |
| `commandOptionsInit` | string | `""` | Additional arguments appended to `terraform init` |
| `workingDirectory` | string | `$(System.DefaultWorkingDirectory)` | Directory containing Terraform configuration files |
| `backendAzureRmUseEntraIdForAuthentication` | boolean | `false` | Use Entra ID (OIDC) for backend authentication instead of storage account key |
| `azureRmUseIdTokenGeneration` | boolean | `false` | Required when using AzureRM provider version > 3.80 with Workload Identity Federation |
| `allowedBranches` | object | `["refs/heads/master","refs/heads/main"]` | List of branches allowed to trigger the Deploy stage. Set to `[]` to allow all branches |
| `initStepsPlan` | stepList | `[]` | Steps injected before `terraform init` in the Plan stage |
| `initStepsApply` | stepList | `[]` | Steps injected before `terraform init` in the Deploy stage |
| `beforePlanJobs` | jobList | `[]` | Jobs injected before the Plan job |
| `beforeDeployJobs` | jobList | `[]` | Jobs injected before the Deploy job |
| `afterDeployJobs` | jobList | `[]` | Jobs injected after the Deploy job |
| `preApplySteps` | stepList | `[]` | Steps injected before `terraform apply` |
| `afterApplySteps` | stepList | `[]` | Steps injected after `terraform apply` |
| `useCLIApply` | boolean | `false` | When `true`, runs `terraform apply` via an `AzureCLI@2` task instead of the AzureRM service connection. Useful for Workload Identity Federation setups |
| `timeoutInMinutes` | number | `60` | Timeout in minutes for both the Plan job and Deploy deployment |
| `templateVariables` | object | `{}` | Stage-level variables injected into the Plan stage (e.g. subscription ID, environment tag) |
| `planDependsOnStage` | string | `""` | Name of a prior stage the Plan stage should depend on. Leave empty for no dependency |

---

### `terraform-destroy.yml`

Runs a two-stage pipeline: **Destroy Plan** → **Destroy Apply**. The Plan stage creates a speculative destroy plan (`-destroy`) and publishes it as an artifact. The Apply stage downloads the artifact and executes the destroy. Both stages are gated by the `allowedBranches` condition and the Azure DevOps environment approval gates.

#### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `AzureRMServiceConnection` | string | — | Service Connection used for `terraform plan -destroy` and `terraform apply -destroy` |
| `AzureRMServiceConnectiontfState` | string | — | Service Connection used to authenticate against the Terraform remote state storage account |
| `storage_account_resource_group_name` | string | `rg-TerraformBackend-01` | Resource group of the Terraform state storage account |
| `storage_account_name` | string | — | Name of the storage account holding remote state |
| `container_name` | string | — | Blob container name for the remote state file |
| `key` | string | — | Path to the state file inside the container |
| `environment` | string | — | Azure DevOps environment name. Used as stage suffix and deployment gate |
| `repository` | string | — | Repository to check out. Use `self` for the calling repo |
| `variables` | object | `{}` | Terraform input variables passed to `terraform plan -destroy` |
| `terraformVersion` | string | `1.1.2` | Version of Terraform to install |
| `commandOptions` | string | `""` | Additional arguments appended to `terraform plan -destroy` and `terraform apply -destroy` |
| `commandOptionsInit` | string | `""` | Additional arguments appended to `terraform init` |
| `workingDirectory` | string | `$(System.DefaultWorkingDirectory)/$(Build.Repository.Name)` | Directory containing Terraform configuration files |
| `backendAzureRmUseEntraIdForAuthentication` | boolean | `false` | Use Entra ID (OIDC) for backend authentication instead of storage account key |
| `azureRmUseIdTokenGeneration` | boolean | `false` | Required when using AzureRM provider version > 3.80 with Workload Identity Federation |
| `allowedBranches` | object | `["refs/heads/master","refs/heads/main"]` | List of branches allowed to trigger destroy stages. Set to `[]` to allow all branches |
| `cleanWorkspace` | string | `resources` | Workspace clean strategy (`outputs`, `resources`, `all`, or `none`) |
| `initStepsPlan` | stepList | `[]` | Steps injected before `terraform init` in the Destroy Plan stage |
| `initStepsApply` | stepList | `[]` | Steps injected before `terraform init` in the Destroy Apply stage |
| `beforePlanJobs` | jobList | `[]` | Jobs injected before the Destroy Plan job |
| `beforeDeployJobs` | jobList | `[]` | Jobs injected before the Destroy deployment job |
| `afterDeployJobs` | jobList | `[]` | Jobs injected after the Destroy deployment job |
| `preApplySteps` | stepList | `[]` | Steps injected before `terraform apply -destroy` |
| `useCLIApply` | boolean | `false` | When `true`, runs `terraform apply -destroy` via an `AzureCLI@2` task instead of the AzureRM service connection |
| `afterApplySteps` | stepList | `[]` | Steps injected after `terraform apply -destroy` |
| `timeoutInMinutes` | number | `60` | Timeout in minutes for both the Plan job and Destroy deployment |
| `templateVariables` | object | `{}` | Stage-level variables injected into the Destroy Plan stage |
| `planDependsOnStage` | string | `""` | Name of a prior stage the Destroy Plan stage should depend on. Leave empty for no dependency |

---

## Advanced Usage

### Workload Identity Federation (OIDC)

When using Workload Identity Federation with AzureRM provider version > 3.80, set both flags:

```yaml
- template: terraform-destroy.yml@iacwh-template-pipelines
  parameters:
    azureRmUseIdTokenGeneration: true
    backendAzureRmUseEntraIdForAuthentication: true
    useCLIApply: true   # recommended for WIF environments
    ...
```

### Injecting Steps and Jobs

Use `initStepsPlan`, `initStepsApply`, `preApplySteps`, `afterApplySteps`, `beforePlanJobs`, `beforeDeployJobs`, and `afterDeployJobs` to extend the pipeline without forking the template:

```yaml
- template: terraform-deploy.yml@iacwh-template-pipelines
  parameters:
    preApplySteps:
      - script: echo "Running pre-apply validation"
        displayName: Pre-Apply Check
    afterApplySteps:
      - script: echo "Notifying Slack"
        displayName: Post-Deploy Notification
    ...
```

### Restricting Destroy to Specific Branches

By default, both stages require the pipeline to run from `refs/heads/master` or `refs/heads/main`. Override this for stricter control:

```yaml
- template: terraform-destroy.yml@iacwh-template-pipelines
  parameters:
    allowedBranches:
      - "refs/heads/destroy/approved"
    ...
```

Set `allowedBranches: []` to allow any branch (useful for ephemeral environments).

### Stage Dependencies

Use `planDependsOnStage` to chain the Destroy Plan stage after another stage, such as an approval or notification gate:

```yaml
stages:
  - stage: RequestApproval
    jobs:
      - job: Notify
        steps:
          - script: echo "Approval requested"

  - template: terraform-destroy.yml@iacwh-template-pipelines
    parameters:
      planDependsOnStage: RequestApproval
      environment: "prod"
      ...
```

---

## Step Templates

The following step templates are used internally and can also be referenced directly:

| Template | Description |
|---|---|
| `steps/terraform-init.yml` | Runs `terraform init` with remote backend configuration |
| `steps/terraform-plan.yml` | Runs `terraform plan` with variable injection |
| `steps/terraform-apply.yml` | Runs `terraform apply` using an AzureRM service connection |

---

## License

MIT





# Introduction

This repository contains Templates to be used in Azure DevOps Pipelines.
Use the templates in subfolders only if you know what you are doing!

# Getting Started

Reference this repository in your pipeline. Please make sure to use a recent tag from this repository as reference. 

Since this repository is located in GitHub, a [corresponding service connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#github-service-connection) needs to be set up in order to access the pipeline definition.

```
resources:
  repositories:
    - repository: iacwh-template-pipelines                 # name is referenced in template, don't change this
      type: github
      name: qbeyond/iacwh-template-pipelines
      endpoint: MyGithubServiceConnection
      ref: "refs/tags/v1.3.1"
```

Afterwards you can reference the needed template and adjust the parameters as appropriate.

```
- template: terraform-deploy.yml@iacwh-template-pipelines  # Template reference
parameters:
    repository: self                                       # if referencing this template multiple times, specify repo name
    AzureRMServiceConnectiontfState: 'ServiceConnectionForTfState'
    AzureRMServiceConnection: 'ServiceConnection'
    storage_account_resource_group_name: 'resource-group'
    terraformVersion: '1.14.5'
    storage_account_name: 'storageaccount'
    container_name: 'container'
    key: 'key.tfstate'
    environment: 'DEVCU'
    variables:
      state: 'dev'
```

## Using the Deploy Template

To use the deploy Template you must create an Environment and configure Approvals!
The environment is used as id of stages so it should only contain `[A-Za-z0-9]` or underscores. Spaces and dashes are replaces by underscrores and can be used.
