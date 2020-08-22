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

### count and for_each for modules

This feature is very simple. You can easily create multiple instances of a module from a **single** module block.

So here is a scenario and remember this is very simplistic for the purposes of this demo.

We have a requirement to create three resource groups. They are all in the same region and their names will only differ by an integer suffix.

So we will end up with three resources groups as follows:

- rg-1
- rg-2
- rg-3

Firstly let's write our resource group module. Here you can see the layout of my project:
![Layout](/assets/rg_module.png)


```hcl
resource "azurerm_resource_group" "example" {

  count               = var.instance_count
  
  name     = "${var.resource_group_name}${count.index + 1}"
  location = "Australia East"
}
```

