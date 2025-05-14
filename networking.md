# Terraform Syntax - Networking

## Virtual Networks
### Create a Virtual Network

```hcl
resource "azurerm_virtual_network" "vnet" {
  name                = "learn-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.learn.location
  resource_group_name = azurerm_resource_group.learn.name
}
```

### Create a Subnet
```hcl
resource "azurerm_subnet" "vmss_subnet" {
  name                 = "vmss_subnet"
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.this.name
  address_prefixes     = ["10.0.2.0/24"]
}
```

### Create a Subnet with Delegation

```hcl
resource "azurerm_subnet" "aci_subnet" {
  name                 = "aci-subnet"
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.this.name
  address_prefixes     = ["10.0.1.0/24"]

  delegation {
    name = "aci-delegation"

    service_delegation {
      name    = "Microsoft.ContainerInstance/containerGroups"
      actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
    }
  }
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

### Create a public ip

resource "azurerm_public_ip" "lb_public_ip" {
  name                = "${var.resource_group_name}-lb-pip"
  resource_group_name = var.resource_group_name
  location            = var.location
  allocation_method   = "Static"
  sku                 = "Basic"
}

## Load Balancer

### Create a loadbalance with a public ip

resource "azurerm_lb" "this" {
  name                = "${var.resource_group_name}-lb"
  location            = var.location
  resource_group_name = var.resource_group_name
  sku                 = "Basic"

  frontend_ip_configuration {
    name                 = "PublicIPAddress"
    public_ip_address_id = azurerm_public_ip.this.id
  }
}

### Creating a loadbalncer health probe

# Creating Health Probes
resource "azurerm_lb_probe" "this" {
  name                            = "${var.resource_group_name}-http-probe"
  loadbalancer_id                 = azurerm_lb.this.id
  protocol                        = "Http"
  port                            = 80
  request_path                    = "/health"  # HTTP path to check (app needs to respond to this)
  interval_in_seconds             = 15  # How often to check
}

## App Gateway