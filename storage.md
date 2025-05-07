# Terraform - Azure Storage Syntax

## Create a Storage Account

```hcl
resource "azurerm_storage_account" "example" {
  name                     = "examplestorageacct"  # Must be globally unique
  resource_group_name      = var.resource_group_name
  location                 = var.location
  account_tier              = "Standard"
  account_replication_type = "LRS"  # Locally redundant storage for availability

  tags = {
    environment = "dev"
    project     = "AzureScale"
  }
}
```