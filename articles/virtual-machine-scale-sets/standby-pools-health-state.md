---
title: Understand the health state of your standby pool
description: Learn how to get and understand the health state of your standby pool using the runtime view API.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: conceptual
ms.date: 4/23/2025
ms.reviewer: ju-shim
---

# Understand the health state of your standby pool

The health state of your standby pool provides critical insights into its operational status and helps ensure that your Virtual Machine Scale Sets are running efficiently. By using the Standby Pool runtime view API, you can retrieve the current health state of your standby pool, including details about instance counts, provisioning state, and overall health status. This information allows you to monitor the pool's performance and take proactive measures to address any issues.

## Health state overview

The health state of a standby pool is determined by analyzing various metrics, such as the number of instances in different states (e.g., running, deallocated, creating), provisioning status, and system health indicators. The runtime view API provides a detailed snapshot of these metrics, enabling you to:

- Monitor the current state of instances in the pool.
- Identify potential issues, such as insufficient capacity or provisioning delays.
- Ensure the pool is operating within expected parameters.

## Retrieve health state using the runtime view API

You can use the runtime view API to retrieve the health state of your standby pool. Below are examples of how to query the API using different tools.

### [CLI](#tab/cli)

Run the following Azure CLI command to get the health state of your standby pool:

```azurecli
az standby-vm-pool status --resource-group myResourceGroup --name myStandbyPool
```
**Output**
```azurecli
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool/runtimeViews/latest",
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 1,
          "state": "Running"
        },
        {
          "count": 3,
          "state": "Deallocating"
        },
        {
          "count": 7,
          "state": "Deallocated"
        },
         {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ]
   }
  "name": "latest",
  "resourceGroup": "myResourceGroup",

  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews"
}

```

The response will include details such as instance counts by state, provisioning state, and overall health status:

```
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool/runtimeViews/latest",
  "properties": {
    "instanceCountsByState": [
      {
        "state": "Running",
        "count": 5
      },
      {
        "state": "Deallocated",
        "count": 10
      }
    ],
    "provisioningState": "Succeeded",
    "status": {
      "code": "HealthState/healthy",
      "message": "The pool is healthy."
    }
  }
}
```
### [PowerShell](#tab/powershell)
Use the following PowerShell command to retrieve the health state:

```powershell
Get-AzStandbyVMPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool
```

The output will include similar details about the health state of the pool.


### [REST API](#tab/rest)
Make a GET request to the runtime view API endpoint:

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyPoolName}/runtimeViews/latest?api-version=2025-03-01
```

The response will include detailed health state information, such as instance counts, provisioning state, and health status.

---

### Interpreting the health state
The health state includes the following key components:

- Instance counts by state: Shows the number of instances in each state (e.g., running, deallocated, creating). This helps you understand the current capacity and utilization of the pool.
- Provisioning state: Indicates whether the pool is successfully provisioned or if there are issues that need attention.
- Health status: Provides an overall assessment of the pool's health, such as "healthy" or "unhealthy," along with a message explaining the status.


### Next steps
Learn how to get prediction results for your standby pool.
Review the frequently asked questions about standby pools for Virtual Machine Scale Sets.