---
title: Use Azure Log Analytics to monitor standby pool events
description: Learn how to use Azure Log Analytics to monitor and analyze events from standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 4/23/2025
ms.reviewer: ju-shim
---

# Use Azure Log Analytics to monitor standby pool events

Azure Log Analytics provides a powerful platform for monitoring and analyzing events from standby pools in Virtual Machine Scale Sets. By integrating your standby pools with a Log Analytics workspace, you can track key metrics, analyze trends, and set up alerts for critical events.

## Available metrics and tables

| Table name | Description | 
|---|---|
| `SVMPoolRequestLog` | Contains logs for user-initiated events, such as updates to pool settings (e.g., changes to minimum or maximum ready capacity). |
| `SVMPoolExecutionLog` | Contains logs for system-initiated events, such as standby pool operations like degraded mode, VM reuse, and pool refills. |


| Event name | Description | 
|---|---|
| `StandbyPoolExhaustedPool` | Triggered when the standby pool instance count reaches zero and cannot create more VMs because the pool's max ready capacity is less than or equal to the Virtual Machine Scale Set (VMSS) instance count. This typically occurs when no minimum ready capacity is configured. For example, if the max ready capacity is set to 3, the min ready capacity is 0, and the VMSS requests 3 instances, the standby pool will not refill until the scale set scales down. In this scenario, the pool is considered exhausted, and the event is triggered. Note: If a minimum ready capacity is configured, this event will not be triggered as the pool will always refill to maintain the minimum ready capacity. |
| `StandbyPoolReuseSuccess` | Triggered when a virtual machine has been successfully moved from the standby pool into the scale set. |
| `StandbyPoolReuseFailure` | Triggered when the scale set requests a VM from the standby pool but is unable to provide one, causing the scale set to create a new VM directly. |
| `StandbyPoolSettingsUpdated` | Triggered when a setting is changed on the standby pool resource, such as adjusting the min/max ready capacity or the VM state. |
| `StandbyPoolMaxReadyPool` | Triggered when the number of instances in the standby pool has replenished enough to meet the maximum ready capacity set by the customer. |
| `StandbyPoolDegradedPool` | Triggered when the instances within the standby pool are unable to successfully provision the requested resources, causing the pool to enter a degraded mode for 30 seconds. |
| `StandbyPoolExitDegradedPool` | Triggered when the timeout on degraded mode has expired, and the pool is now attempting to create resources again. |

## Configure Log Analytics for standby pools

> [!NOTE]
> Enabling diagnostic settings for a standby pool resource is not yet available from the Azure portal. 

### [CLI](#tab/cli)
```azurecli
az monitor diagnostic-settings create \
  --name "standbyPoolLogs" \
  --resource "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/<standby-pool-name>" \
  --workspace "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<log-analytics-workspace-name>" \
  --logs '[{"categoryGroup": "allLogs", "enabled": true}]'
```


### [PowerShell](#tab/powershell)
```powershell
# Create log settings object
$log = New-AzDiagnosticSettingLogSettingsObject -Enabled $true -CategoryGroup allLogs  

# Create diagnostic setting
New-AzDiagnosticSetting -Name 'standbyPoolLogs' `
  -ResourceId "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/<standby-pool-name>" `
  -WorkspaceId "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<log-analytics-workspace-name>" `
  -Log $log
```

### [REST](#tab/rest)
```rest
https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/<standby-pool-name>/providers/microsoft.insights/diagnosticSettings/standbyPoolLogs?api-version=2021-05-01-preview

{
  "properties": {
    "workspaceId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<log-analytics-workspace-name>",
    "logs": [
      {
        "categoryGroup": "allLogs",
        "enabled": true
      }
    ]
  }
}
```


## Query standby pool events 



### Example: View all standby pool events in the last 24 hours

## Setup alerts for specific events

## Next steps
