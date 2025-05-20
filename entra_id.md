# Azure Entra Id (Azure Ad)

## Register an Application

```hcl
data "azurerm_client_config" "current" {} # retrieves metadata about the currently authenticated identity being used to run Terraform

resource "azuread_application" "react_spa" {
  display_name = "MySecureReactApp"

  web {
    redirect_uris = ["http://localhost:3000/"] # The URI where Azure AD will send the user back after they log in

    implicit_grant { #Enables Implicit Flow, where tokens (access and ID tokens) are returned directly in the redirect URL.
      access_token_issuance_enabled = true
      id_token_issuance_enabled     = true
    }
  }

  sign_in_audience = "AzureADMyOrg"  # single-tenant
}

resource "azuread_service_principal" "react_spa_sp" {
  client_id = azuread_application.react_spa.client_id
}
```