# terraform - Azure

## Create a Resource group

```h
resource "azurerm_resource_group" "learn" {
  name     = "learn-rg"
  location = "East US"
}
```