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

## Create a Container

```hcl
resource "azurerm_storage_container" "this" {
  name                  = "mycontainer"  # Container name (must be lowercase)
  storage_account_name  = azurerm_storage_account.this.name
  container_access_type = "private"  # Options: "private", "blob", "container"
}
```

## Create a blob

```hcl
resource "azurerm_storage_blob" "example" {
  name                   = "examplefile.txt"  # Name of the blob
  storage_account_name   = azurerm_storage_account.example.name
  storage_container_name = azurerm_storage_container.example.name
  type                   = "Block"  # Block blobs for large files
  source                 = "localpath/examplefile.txt"  # Path to the local file
}
```