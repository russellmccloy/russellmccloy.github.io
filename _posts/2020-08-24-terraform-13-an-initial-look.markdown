---
layout: post
title:  "Terraform 0.13.0 - An Initial Look"
date:   2020-08-26 06:26:18 +1000
categories: general

---

Today I'd like to write a little bit about Terraform `0.13.0` which was released on the 10/08/2020
You can find the official release here: 
[https://github.com/hashicorp/terraform/releases/tag/v0.13.0](https://github.com/hashicorp/terraform/releases/tag/v0.13.0)

Unlike many other blogs on this subject, I have tried and tested all the new features below apart from the **New Kubernetes remote state storage backend** feature.

**Some of the new features are as follows:**

- [count and for_each for modules](https://www.terraform.io/docs/configuration/modules.html#multiple-instances-of-a-module): Similar to the arguments of the same name in resource and data blocks, these create multiple instances of a module from a single module block. [#24461](https://github.com/hashicorp/terraform/issues/24461)

- [depends_on for modules](https://www.terraform.io/docs/configuration/modules.html#other-meta-arguments): Modules can now use the depends_on argument to ensure that all module resource changes will be applied after any changes to the depends_on targets have been applied. [#25005](https://github.com/hashicorp/terraform/issues/25005)

- [Automatic installation of third-party providers](https://www.terraform.io/docs/configuration/provider-requirements.html): Terraform now supports a decentralized namespace for providers, allowing for automatic installation of community providers from third-party namespaces in the public registry and from private registries. (More details will be added about this prior to release - *Obviously they forgot to update this part after the release!* :))

- [Custom validation rules for input variables](https://www.terraform.io/docs/configuration/variables.html#custom-validation-rules): A new validation block type inside variable blocks allows module authors to define validation rules at the public interface into a module, so that errors in the calling configuration can be reported in the caller's context rather than inside the implementation details of the module. [#25054](https://github.com/hashicorp/terraform/issues/25054)

- [New Kubernetes remote state storage backend](https://www.terraform.io/docs/backends/types/kubernetes.html): This backend stores state snapshots as Kubernetes secrets. [#19525](https://github.com/hashicorp/terraform/issues/19525)

## Working samples of the above list

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

Also, for completeness my `provider.tf` file looks like this as I am working with Azure:

```hcl
provider "azurerm" {
  version = "~> 2.0"
  features {}
}
```

### count and for_each for modules

#### count

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

As you can see, this module takes a `count` from `instance_count` variable and it builds up its name using this `count`.

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

From the `plan` output we can see that Terraform will create the three resources groups as described above:

```ssh
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

#### for_each

This feature is a little more complex but gives you much more control over the data in the map you iterate over to produce multiple resources from a single module.

In this scenario we will create a variable on our module that receives a map of objects that describes our resource group's name and location:

```hcl
variable "resource_groups" {
  type = map(object({
    name      = string
    location  = string
  }))
}
```

The code for our resource group within our module now looks like this:

```hcl
resource "azurerm_resource_group" "example" {

  for_each = var.resource_groups

  name     = each.value.name
  location = each.value.location
}
```

And in our code that consumes the module we have this:

```hcl
module "my_many_resource_groups" {
  source = "./modules/resource_group"

  resource_groups = local.resource_groups
}
```

and for simplicity I have hard coded the map into a local:

```hcl
locals {
  resource_groups = {
    rg1 = {
      name     = "rg-1"
      location = "Australia East"
    }

    rg2 = {
      name     = "rg-2"
      location = "Australia SouthEast"
    }

    rg3 = {
      name     = "rg-3"
      location = "Australia Central"
    }
  }
}
```

So in summary:

- we have three resource group properties defined in our map
- we pass this map into our module
- the module receives this data through the above mentioned variable
- the module iterates through the passed in data and deploys three resource groups with three different names into three different regions

```ssh
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.my_many_resource_groups.azurerm_resource_group.example["rg1"] will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "australiaeast"
      + name     = "rg-1"
    }

  # module.my_many_resource_groups.azurerm_resource_group.example["rg2"] will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "australiasoutheast"
      + name     = "rg-2"
    }

  # module.my_many_resource_groups.azurerm_resource_group.example["rg3"] will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "australiacentral"
      + name     = "rg-3"
    }

Plan: 3 to add, 0 to change, 0 to destroy.
```

That's it for now!

### depends_on for modules

Modules can now use the `depends_on` property to ensure that all module resource changes will be applied after any changes to the depends_on targets have been applied.

This is what version `0.12.29` would have done if you attempted to use depends on with modules:

```ssh
Error: Reserved argument name in module block

  on main.tf line 9, in module "my_storage_account":
   9:   depends_on = [module.my_many_resource_groups]

The name "depends_on" is reserved for use in a future version of Terraform.
```

But now with Terraform `0.13.0` you can do this:

![Depends On](/assets/depends_on.png)

> Admittedly, the above `depends_on` example is not great but you get the idea.

### Automatic installation of third-party providers

In this section I will describe how to automatically install 3rd party providers.

We would like to automatically install the following two providers:

- Azure DevOps provider - [https://registry.terraform.io/providers/ellisdon-oss/azuredevops/latest](https://registry.terraform.io/providers/ellisdon-oss/azuredevops/latest)
- Terraform Random provider - [https://registry.terraform.io/providers/hashicorp/random/latest](https://registry.terraform.io/providers/hashicorp/random/latest)

from the Terraform Registry which is here: [https://registry.terraform.io/](https://registry.terraform.io/)

So, in my `versions.tf` file I will do the following:
```hcl
terraform {
  required_version = "~> 0.13.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 2.0"
    }
    azuredevops = {
      source = "ellisdon-oss/azuredevops"
      version = "0.0.2"
    }
    random = {
      source = "hashicorp/random"
      version = "2.3.0"
    }
  }
}
```

Then when we run `terraform init -upgrade` we see the providers being downloaded: 

> Note: `terraform init -upgrade` blows any previous downloads away and gets fresh ones.

```ssh
Upgrading modules...
- my_many_resource_groups in modules/resource_group

Initializing the backend...

Initializing provider plugins...
- Finding ellisdon-oss/azuredevops versions matching "0.0.2"...
- Finding hashicorp/random versions matching "2.3.0"...
- Finding hashicorp/azurerm versions matching "~> 2.0, ~> 2.0"...
- Installing ellisdon-oss/azuredevops v0.0.2...
- Installed ellisdon-oss/azuredevops v0.0.2 (self-signed, key ID 1033ECE171B90D34)
- Installing hashicorp/random v2.3.0...
- Installed hashicorp/random v2.3.0 (signed by HashiCorp)
- Installing hashicorp/azurerm v2.24.0...
- Installed hashicorp/azurerm v2.24.0 (signed by HashiCorp)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/plugins/signing.html

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

You can see that it downloaded the required providers for you.

I won't go into the Azure DevOps provider but I will however show you an example of why I downloaded the Terraform Random provider.

You may remember that originally, the resource groups I am deploying were named like this: `rg-1, rg-2` etc.

now we will use a randomly generated string instead:

```hcl
resource "azurerm_resource_group" "example" {

  count = var.instance_count

  name     = "${random_string.random.result}${count.index + 1}"
  location = "Australia East"
}

resource "random_string" "random" {
  length  = 24
  special = false
}
```

That's all there is to it.

### Custom validation rules for input variables

This section describes the new custom validation rules that you can add to your input variables. The idea here is you can control what is allowed / not-allowed to be passed in as a variable.

you may remember our `resource_group_name` variable from above. Here we have added some validation to it to ensure the name starts with `rg-`:

```hcl
variable "resource_group_name" {
  type = string

  validation {
    condition     = can(regex("^rg-", var.resource_group_name))
    error_message = "Hey, we have naming standards around here! Please prefix your resource group name with:  \"rg-\"."
  }

  default = "nope"
}
```

And when we run `terraform plan` you can see that it stops you in your tracks:

```ssh
Error: Invalid value for variable

  on modules/resource_group/variables.tf line 1:
   1: variable "resource_group_name" {

Hey, we have naming standards around here! Please prefix your resource group
name with:  "rg-".

This was checked by the validation rule at
modules/resource_group/variables.tf:4,3-13.
```

You could also check the minimum length:

```hcl
variable "resource_group_name" {
  type = string

  validation {
    condition     = length(var.resource_group_name) > 4 && substr(var.resource_group_name, 0, 4) == "rg-"
    error_message = "Hey, we have naming standards around here! Please prefix your resource group name with:  \"rg-\" and make sure it is more than 4 characters long."
  }

  default = "nope"
}
```

Obviously, don't use validation as the only control around naming standards, maybe introduce a `naming` module to enforce naming standards. 

### New Kubernetes remote state storage backend

As mentioned above this feature allows you to store your Terraform state in a Kubernetes secret with locking done using a Lease resource.

Here are the details:
[https://www.terraform.io/docs/backends/types/kubernetes.html](https://www.terraform.io/docs/backends/types/kubernetes.html)

I have not worked with this to it is hard for me to comment but I think the gist is that, Terraform has given us a new `backend` type.

![Backend Types](/assets/backend_types.png)

I presume the benefit here is that you can store all your assets close the the product you are working on instead of having your state stored somewhere else such as an Azure Storage Account, for example.

Please feel free to let me know more about the Kubernetes remote state storage backend and I will update the information here.

Cheers
Russ