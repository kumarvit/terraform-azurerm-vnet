# Virtual Network resource creation example

[![Terraform](https://img.shields.io/badge/Terraform%20-0.12-brightgreen.svg?style=flat)](https://github.com/hashicorp/terraform/releases) [![License](https://img.shields.io/badge/License%20-MIT-brightgreen.svg?style=flat)](https://github.com/kumarvna/cloudascode/blob/master/LICENSE)

Configuration in this directory creates a set of Azure network resources. Few of these resources added/excluded as per your requirement.

## Create resource group

By default, this module will not create a resource group and the name of an existing resource group to be given in an argument `create_resource_group`. If you want to create a new resource group, set the argument `create_resource_group = true`.

*If you are using an existing resource group, then this module uses the same resource group location to create all resources in this module.*

## Using Custom names and VNet/subnet Address Prefix

This is optional. However, we recommend you to check with the network or cloud teams and give suitable CIDR block to configure these values. Provide proper values to following arguments to use your custom names and address prefix.

```
  resource_group_name   = "rg-demo-westeurope-01"
  vnetwork_name         = "vnet-demo-westeurope-001"
  location              = "westeurope"
  vnet_address_space    =  ["10.1.0.0/16"]
  private_subnets       = ["snet-app01","snet-app02","snet-gw01"]
  subnet_address_prefix = ["10.1.1.0/24","10.1.2.0/24","10.1.3.0/24"]

```

If you haven't provided with any values for the above arguments, then default values to apply to create the required resources. You can create as many subnets as you can!

To get the list of all locations with table format from CLI, run this command `"az account list-locations -o table"`

## Adding Network Watcher to your Subscription

This is a must value and note that you cannot create more than one network watcher resource per subscription in any region. You can exclude this from the Terraform plan using `create_network_watcher = false` argument in case you already have a network watcher available in your subscription.

By default, this enabled. Only "true" or "false" values accepted.

## Adding the Azure Network DDoS Protection Plan

This is an optional resource.  You can add a DDoS standard plan to VPC by providing appropriate value to the argument `create_ddos_plan = false`.  

By default, this not enabled. Only "true" or "false" values are accepted.

## Adding your own DNS servers

This is an optional and only applicable if you are using different DNS servers in Azure than its provided by default. Provide appropriate values to the argument `dns_servers = ["4.4.4.4"]`.

By default, this not enabled. Only IPs expected here. If you have multiple DNS servers, then values to the argument `dns_servers = ["4.4.4.4","8.8.8.8"]`

## Adding TAG's to your Azure resources

Use tags to organize your Azure resources and management hierarchy. You can apply tags to your Azure resources, resource groups, and subscriptions to logically organize them into a taxonomy. Each tag consists of a name and a value pair. For example, you can apply the name "Environment" and the value "Production" to all the resources in production. You can manage these values variables directly or mapping as a variable using `variables.tf`.

All network resources which support tagging can be tagged by specifying key-values in argument `tags`. Tag Name is added automatically on all resources. For example, you can specify `tags` like this as per environment:

```
module "vnet" {
  source        = "github.com/tietoevry-infra-as-code/terraform-azurerm-vnet?ref=v1.0.0"
  create_resource_group   = false

  # ... omitted

  tags = {
    Terraform   = "true"
    Environment = "dev"
    Owner       = "test-user"
  }
}  
```

## Module Usage

## Basic VNet

Following example to create a virtual network with subnets and network watcher resources.

```
module "vnet" {
  source                  = "github.com/tietoevry-infra-as-code/terraform-azurerm-vnet?ref=v1.0.0"
  create_resource_group   = false

# Using Custom names and VNet/subnet Address Prefix (Recommended)
  resource_group_name     = "rg-demo-westeurope-01"
  vnetwork_name           = "vnet-demo-westeurope-001"
  location                = "westeurope"

# Adding TAG's to your Azure resources (Required)
  tags = {
    Terraform   = "true"
    Environment = "dev"
    Owner       = "test-user"
  }
}
```

## VNet with all additional features

Following example to create a virtual network with subnets, DDoS protection plan, and network watcher resources.

```
module "vnet" {
  source                  = "github.com/tietoevry-infra-as-code/terraform-azurerm-vnet?ref=v1.0.0"
  create_resource_group   = false

# Using Custom names and VNet/subnet Address Prefix (Recommended)
  resource_group_name     = "rg-demo-westeurope-01"
  vnetwork_name           = "vnet-demo-westeurope-001"
  location                = "westeurope"
  vnet_address_space      = ["10.1.0.0/16"]
  private_subnets         = ["snet-app01","snet-app01"]
  subnet_address_prefix   = ["10.1.2.0/24","10.1.3.0/24"]

# Adding Network watcher, Firewall and custom DNS servers (Optional)
  create_ddos_plan        = false
  dns_servers             = []

# Adding TAG's to your Azure resources (Required)
  tags = {
    Terraform   = "true"
    Environment = "dev"
    Owner       = "test-user"
  }
}
```

## Terraform Usage

To run this example you need to execute following Terraform commands

```
$ terraform init
$ terraform plan
$ terraform apply
```

Run `terraform destroy` when you don't need these resources.

## Outputs

Name | Description
---- | -----------
`resource_group_name` | The name of the resource group in which resources are created
`resource_group_id` | The id of the resource group in which resources are created
`resource_group_location`| The location of the resource group in which resources are created
`virtual_network_name` | The name of the virtual network.
`virtual_network_id` |The virtual NetworkConfiguration ID.
`virtual_network_address_space` | List of address spaces that are used the virtual network.
`subnet_ids` | List of IDs of subnets
`subnet_address_prefixes` | List of address prefix for  subnets
`ddos_protection_plan` | Azure Network DDoS protection plan
`network_watcher_id` | ID of Network Watcher