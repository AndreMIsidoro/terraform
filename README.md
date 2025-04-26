# terraform - Azure

## Use Azure Resource Provider


```hcl
provider "azurerm"{
    features {}
}
```

## Create a Resource group

```hcl
resource "azurerm_resource_group" "learn" {
  name     = "learn-rg"
  location = "East US"
}
```

## Create a Virtual Network

```hcl
resource "azurerm_virtual_network" "vnet" {
  name                = "learn-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.learn.location
  resource_group_name = azurerm_resource_group.learn.name
}
```

## Create a Network Security Group

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

## Create a Network Interface Card

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

# # Associate the NIC to the NSG

```hcl
resource "azurerm_network_interface_security_group_association" "nic_nsg_assoc" {
  network_interface_id     = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}
```