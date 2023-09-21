---
layout: post
title:  "GitHub Actions with Terraform - simple"
date:   2020-08-15 11:26:18 +1000
categories: general

---

Note that you can get the markdown for a status badge to put in your README from the Action home page:

![Deploy Azure Resources](https://github.com/russellmccloy/github_actions_terraform/workflows/Deploy%20Azure%20Resources/badge.svg)

This repository demonstrates using GitHub Actions to deploy Azure resources with Terraform. It deploys a `resource group` and a `storage account`

It uses Github secrets to store the `client secret` for the `providers.tf` file and the `storage account key` for the `backend.tf` file.

![Secrets](/assets/secrets.jpg)

the terraform state is storage in a container in an already created storage account in Azure.

When you set up a GitHub action you need to put a `/.github/workflows/something.yaml` file in your solution

Following is an explanation of the `yaml` file in this solution.

## YAML

When a commit is pushed ot the `master` branch
```yaml
name: Deploy Azure Resources

on:
  push:
    branches:
      - master
```

then we have a `jobs` section which contains a collection of steps. 
We have chosen to run the GitHub Actions Runner on `ubuntu-latest`

```yaml

jobs:
  login-and-deploy-to-Azure:
    runs-on: ubuntu-latest
```

Beginning of steps section:
```yaml
    steps:
```

The action checks out the code from the `master branch` to the ubuntu agent's file system.

```yaml
      - uses: actions/checkout@master
```

As we mentioned github Actions Secrets above, this is where we replace tokens in our `Terraform` files with the real values from our secrets

```hcl
provider "azurerm" {
  version = "~> 2.0"
  features {}
  subscription_id = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
  client_id       = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
  client_secret   = "__CLIENT_SECRET__"
  tenant_id       = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
}
```

```yaml
      - uses: cschleiden/replace-tokens@v1
        with:
          tokenPrefix: '__'
          tokenSuffix: '__'
          files: '["*.tf"]'
        env:
          CLIENT_SECRET: ${{ secrets.CLIENT_SECRET }}
          STORAGE_FOR_STATE_KEY: ${{ secrets.STORAGE_FOR_STATE_KEY }}
```

The rest of the file sets up Terraform, runs INIT, PLAN and APPLY:

```yaml
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        if: github.ref == 'refs/heads/master' && github.event_name == 'push'
        run: terraform apply -auto-approve
```

the gitGub Action runs these in an easy to understand viewer:
![action_running](/assets/action_running.jpg)


the final deployed result is as follows:
![Deployed](/assets/az-sa-created.jpg)
