---
layout: post
title:  "Terraform 13 - Some little samples"
date:   2020-08-26 07:26:18 +1000
categories: general

---

Today I'd like to write a little bit about Terraform 13 which was released on the 10/08/2020
You can find the official release here: 
[https://github.com/hashicorp/terraform/releases/tag/v0.13.0](https://github.com/hashicorp/terraform/releases/tag/v0.13.0)

**Some of the new features are as follows:**

- [count and for_each for modules](https://www.terraform.io/docs/configuration/modules.html#multiple-instances-of-a-module): Similar to the arguments of the same name in resource and data blocks, these create multiple instances of a module from a single module block. [#24461](https://github.com/hashicorp/terraform/issues/24461)

- [depends_on for modules](https://www.terraform.io/docs/configuration/modules.html#other-meta-arguments): Modules can now use the depends_on argument to ensure that all module resource changes will be applied after any changes to the depends_on targets have been applied. [#25005](https://github.com/hashicorp/terraform/issues/25005)

- [Automatic installation of third-party providers](https://www.terraform.io/docs/configuration/provider-requirements.html): Terraform now supports a decentralized namespace for providers, allowing for automatic installation of community providers from third-party namespaces in the public registry and from private registries. (More details will be added about this prior to release.)

- [Custom validation rules for input variables](https://www.terraform.io/docs/configuration/variables.html#custom-validation-rules): A new validation block type inside variable blocks allows module authors to define validation rules at the public interface into a module, so that errors in the calling configuration can be reported in the caller's context rather than inside the implementation details of the module. [#25054](https://github.com/hashicorp/terraform/issues/25054)

- [New Kubernetes remote state storage backend](https://www.terraform.io/docs/backends/types/kubernetes.html): This backend stores state snapshots as Kubernetes secrets. [#19525](https://github.com/hashicorp/terraform/issues/19525)

## Working samples of the above

Note that in my `version.tf` file I have the following which specifies Terraform version `0.13.0`:
```hcl
terraform {
  required_version = "~> 0.13.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 2.0"
    }
  }
}
```

Also, for completeness my `provider.tf` file looks like this as I am working with azure:

```hcl
provider "azurerm" {
  version = "~> 2.0"
  features {}
}
```

### count and for_each for modules

This feature is very simple. You can easily create multiple instances of a module from a **single** module block.

So here is a scenario and remember this is very simplistic for the purposes of this demo.

We have a requirement to create three resource groups. They are all in the same region and their names will only differ by an integer suffix.

So we will end up with three resources groups as follows:

- rg-1
- rg-2
- rg-3

Firstly let's write our resource group module. Here you can see the layout of my project. Note that I have placed my module in a local folder called `/modules/resource_group`:

![Layout](/assets/rg_module.png)


```hcl
resource "azurerm_resource_group" "example" {

  count = var.instance_count

  name     = "${var.resource_group_name}${count.index + 1}"
  location = "Australia East"
}
```

As you can see, this module takes a `count` from `instance_count` and it builds up its name using this `count`.

Now, in my `main.tf` file we will use this module to create three resources groups as mentioned in the scenario above.

I also have a couple of variables on my module that receive input from the consuming code:

```hcl
variable "resource_group_name" {
  type = string

  default = "rg-"
}

variable "instance_count" {
  type = number
}
```

```hcl
module "my_many_resource_groups" {
  source = "./modules/resource_group"

  instance_count = 3
}
```

now we can run:

- `terraform init`
- `terraform plan`

From the plan'2 output we can see that Terraform will create the three resources groups as described above:

```hcl
Terraform will perform the following actions:

  # module.my_many_resource_groups.azurerm_resource_group.example[0] will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "australiaeast"
      + name     = "rg-1"
    }

  # module.my_many_resource_groups.azurerm_resource_group.example[1] will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "australiaeast"
      + name     = "rg-2"
    }

  # module.my_many_resource_groups.azurerm_resource_group.example[2] will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "australiaeast"
      + name     = "rg-3"
    }

Plan: 3 to add, 0 to change, 0 to destroy.
```

It's not fancy, but I hope you get the idea.