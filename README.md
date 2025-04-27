# terraform - Azure

## Env variables

Tells Terraform: "Use the Azure CLI's current login credentials (az login) instead of client secrets, certificates, or service principals.
```shell
export ARM_USE_AZURECLI_AUTH=true 
```

Tells Terraform: "Which Azure subscription you want to target." It's about where the resources will be created.

```shell
export ARM_SUBSCRIPTION_ID=<subscription_id>
```

## CLI

Display the values of outputs that are explicitly defined in your configuration (either in the root module or within modules)

```shell
terraform output
```

Show all resources and their attributes, including any outputs defined in your configuration.

```shell
terraform show
```

updates the Terraform state to reflect the current state of your infrastructure, as seen from the cloud provider or other external systems.

```shell
terraform refresh
```


## Meta Arguments

### Count

The `count` meta-argument is a special argument in Terraform that allows you to manage the creation of multiple identical resources. This meta-argument enables you to create resources dynamically based on a specified number (`count`), which can simplify your infrastructure code when you need to create multiple instances of the same resource type.

### Basic Usage

The `count` meta-argument takes a numerical value that determines how many instances of a resource should be created. This value can be a literal number, a variable, or an expression that evaluates to a number.

```hcl
resource "azurerm_virtual_machine" "vm" {
  count               = 3   # Creates 3 VMs
  location           = var.location
  resource_group_name = var.resource_group_name
  name                = "learn-vm-${count.index}"  # Uses count.index to differentiate VMs
}
```



## Syntax

### Use Azure Resource Provider


```hcl
provider "azurerm"{
    features {}
}
```

### Create a Resource group

```hcl
resource "azurerm_resource_group" "learn" {
  name     = "learn-rg"
  location = "East US"
}
```

### Create a Virtual Network

```hcl
resource "azurerm_virtual_network" "vnet" {
  name                = "learn-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.learn.location
  resource_group_name = azurerm_resource_group.learn.name
}
```

### Create a Network Security Group

```hcl
resource "azurerm_network_security_group" "nsg" {
  name                = "learn-nsg"
  location            = azurerm_resource_group.learn.location
  resource_group_name = azurerm_resource_group.learn.name

  security_rule {
    name                       = "ssh"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}
```

### Create a Network Interface Card

```hcl
resource "azurerm_network_interface" "nic" {
  name                = "learn-nic"
  location            = azurerm_resource_group.learn.location
  resource_group_name = azurerm_resource_group.learn.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
  }
}
```

### Associate the NIC to the NSG

```hcl
resource "azurerm_network_interface_security_group_association" "nic_nsg_assoc" {
  network_interface_id     = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}
```