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
      ref: "refs/tags/v1.0.0"
```

Afterwards you can reference the needed template and adjust the parameters as appropriate.

```
- template: terraform-deploy.yml@iacwh-template-pipelines  # Template reference
parameters:
    repository: self                                       # if referencing this template multiple times, specify repo name
    AzureRMServiceConnectiontfState: 'ServiceConnectionForTfState'
    AzureRMServiceConnection: 'ServiceConnection'
    storage_account_resource_group_name: 'resource-group'
    terraformVersion: '1.1.2'
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
