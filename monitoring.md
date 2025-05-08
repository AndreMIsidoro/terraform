# Terraform - Monitoring

## Create Log Analytics Workspace

```hcl
resource "azurerm_log_analytics_workspace" "example" {
  name                = "azurescale-workspace" # Unique name for the workspace
  location            = "East US" # Region where the workspace will reside
  resource_group_name = var.resource_group_name # The resource group name
  sku                 = "PerGB2018" # Pricing SKU for the workspace
  retention_in_days   = 30 # Number of days to retain logs and data
}
```

## Create Monitor Diagnositc

```hcl
resource "azurerm_monitor_diagnostic_setting" "vmss_diag" {
  name                       = "vmss-diagnostic"
  target_resource_id         = var.vmss_id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.this.id

  metric {
    category = "AllMetrics"
    enabled  = true
  }

  enabled_log {
    category = "AuditEvent"
  }

}
```

## Create An Action Group

```hcl
resource "azurerm_monitor_action_group" "alerts" {
  name                = "${var.resource_group_name}-ActionGroup"
  resource_group_name = var.resource_group_name
  short_name          = "asg"

  email_receiver {
    name          = "admin-alerts"
    email_address = var.alert_email
  }
}
```

## Create A Metric Alert

```hcl
resource "azurerm_monitor_metric_alert" "high_cpu_alert" {
  name                = "HighCPUAlert"
  resource_group_name = var.resource_group_name
  scopes              = [var.vmss_id]  # VMSS resource
  description         = "Alert when average CPU > 80% for 5 minutes"
  severity            = 2
  frequency           = "PT1M"
  window_size         = "PT5M"
  enabled             = true

  criteria {
    metric_namespace = "Microsoft.Compute/virtualMachineScaleSets"
    metric_name      = "Percentage CPU"
    aggregation      = "Average"
    operator         = "GreaterThan"
    threshold        = 80
  }

  action {
    action_group_id = azurerm_monitor_action_group.alerts.id
  }
}
```