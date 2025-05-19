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

### Create a public ip with dns

```hcl
resource "azurerm_public_ip" "this" {
  name                = "${var.resource_group_name}-publicip"
  resource_group_name = var.resource_group_name
  location            = var.location
  allocation_method   = "Static"
  sku                 = "Standard"
  domain_name_label   = "mygateway" # sets the prefix for the DNS.
  tags = {
    environment = "development"
  }
}
```

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

resource "azurerm_lb_probe" "this" {
  name                            = "${var.resource_group_name}-http-probe"
  loadbalancer_id                 = azurerm_lb.this.id
  protocol                        = "Http"
  port                            = 80
  request_path                    = "/health"  # HTTP path to check (app needs to respond to this)
  interval_in_seconds             = 15  # How often to check
}

## App Gateway

```hcl

resource "azurerm_application_gateway" "this" {
  name                = "${var.resource_group_name}-appgw"
  location            = var.location
  resource_group_name = var.resource_group_name
  sku {
    name     = "Standard_v2"   # Choose WAF_v2 later if needed
    tier     = "Standard_v2"
    capacity = 1               # Autoscaling can be configured later
  }

  gateway_ip_configuration {
    name      = "appgw-ip-config"
    subnet_id = azurerm_subnet.appgw_subnet.id
  }

  frontend_port {
    name = "frontend-port"
    port = 80
  }

  frontend_ip_configuration {
    name                 = "frontend-ip"
    public_ip_address_id = azurerm_public_ip.this.id
  }

  backend_address_pool {
    name = "default-backend-pool"
    ip_addresses = [azurerm_network_interface.nginx.private_ip_address]
  }

  backend_http_settings {
    name                  = "default-http-settings"
    cookie_based_affinity = "Disabled"
    port                  = 80
    protocol              = "Http"
    request_timeout       = 20
  }

  http_listener {
    name                           = "http-listener"
    frontend_ip_configuration_name = "frontend-ip"
    frontend_port_name             = "frontend-port"
    protocol                       = "Http"
  }

  request_routing_rule {
    name                       = "default-routing-rule"
    rule_type                  = "Basic"
    http_listener_name         = "http-listener"
    backend_address_pool_name  = "default-backend-pool"
    backend_http_settings_name = "default-http-settings"
    priority                   = 100
  }

  probe {
    name = "nginx-probe"
    protocol = "Http"
    path = "/"
    interval = 30
    timeout = 30
    unhealthy_threshold = 3
    pick_host_name_from_backend_http_settings = true
    match {
      status_code = ["200-399"]
    }
  }

  tags = {
    environment = "development"
  }
}
```

## Azure Bastion

```hcl
resource "random_id" "dns" {
  byte_length = 4
}

resource "azurerm_public_ip" "bastion_ip" {
  name                = "bastion-public-ip"
  location            = var.location
  resource_group_name = var.resource_group_name
  allocation_method   = "Static"
  sku                 = "Standard"
  domain_name_label = "bastion-${random_id.dns.hex}"
}

resource "azurerm_bastion_host" "bastion" {
  name                = "bastion-host"
  location            = var.location
  resource_group_name = var.resource_group_name

  ip_configuration {
    name                 = "bastion-config"
    subnet_id            = azurerm_subnet.bastion_subnet.id
    public_ip_address_id = azurerm_public_ip.bastion_ip.id
  }
}
```