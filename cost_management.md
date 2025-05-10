# Terraform - Cost Management

## Setup a cost Management of a subscription

```hcl
resource "azurerm_consumption_budget_subscription" "this" {
  name           = "monthly-budget"
  subscription_id = "/subscriptions/${var.subscription_id}"

  amount = 1 # in USD, set your monthly limit
  time_grain = "Monthly"

  time_period {
    start_date = "2025-05-01T00:00:00Z"
  }

  notification {
    enabled        = true
    threshold      = 80.0  # percent
    operator       = "GreaterThan"
    contact_emails = var.contact_emails
  }

  notification {
    enabled        = true
    threshold      = 100.0
    operator       = "GreaterThan"
    contact_emails = var.contact_emails
  }
}
```