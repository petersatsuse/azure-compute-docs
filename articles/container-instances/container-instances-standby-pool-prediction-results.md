---
title: Get Prediction Results for Standby Pools for Azure Container Instances
description: Learn how to use prediction results to help right-size your standby pools for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: conceptual
ms.date: 5/6/2025
ms.reviewer: ju-shim
---

# Get Prediction Results for Standby Pools for Azure Container Instances

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, they require access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider. For detailed instructions, see **[configure role permissions for standby pools](container-instances-standby-pool-configure-permissions.md)**.

To effectively manage and optimize your standby pool for Azure Container Instances, you can use the Standby Pool runtime view APIs to retrieve prediction results. These results, available 2-3 weeks after creating the standby pool, provide insights into the predicted number of container instances that will be requested from the pool for each hour over a 12-hour period. The predictions include the accuracy of the forecast and a historical view of container instances requested from the pool over the past 12 hours, helping you make informed decisions to right-size your standby pool and improve operational efficiency.

While the prediction results provide valuable insights, they are not a guarantee and should be treated as a suggested size for your pool. The actual number of container instances requested from the pool may vary depending on the specific demands of your workload. Additionally, the longer the prediction engine monitors and analyzes your workload trends, the more accurate and reliable the prediction results will become over time.

## Retrieve Prediction Results

### [CLI](#tab/cli)

```azurecli
az standby-container-group-pool status --resource-group myResourceGroup --name myStandbyPool

{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest",
  "instanceCountsByState": [
    {
      "state": "Creating",
      "count": 0
    },
    {
      "state": "Starting",
      "count": 0
    },
    {
      "state": "Running",
      "count": 2
    },
    {
      "state": "Deallocating",
      "count": 3
    },
    {
      "state": "Deallocated",
      "count": 15
    },
    {
      "state": "Deleting",
      "count": 0
    }
  ],
  "prediction": {
    "forecastValues": {
      "instancesRequestedCount": [
        24,
        10,
        20,
        12,
        15,
        10,
        15,
        23,
        14,
        16,
        17,
        19
      ]
    },
    "forecastStartTime": "2025-02-14T01:34:59.228Z",
    "forecastInfo": "{\"forecastAccuracy\": 85, \"seriesUnitIntervalInMins\": 60, \"instancesRequestedCount_recentHistory\": \"[9, 4, 2, 8, 8, 2, 3, 6, 5, 3, 2, 6]\"}"
  },
  "provisioningState": "Succeeded",
  "status": {
    "code": "HealthState/healthy",
    "message": "The pool is healthy."
  }
}
```


### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzStandbyContainerGroupPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool

Id: /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest
InstanceCountSummary: {
  "instanceCountsByState": [
    {
      "state": "Creating",
      "count": 0
    },
    {
      "state": "Starting",
      "count": 0
    },
    {
      "state": "Running",
      "count": 2
    },
    {
      "state": "Deallocating",
      "count": 3
    },
    {
      "state": "Deallocated",
      "count": 15
    },
    {
      "state": "Deleting",
      "count": 0
    }
  ]
}
PredictionForecastInfo       : {"forecastAccuracy": 85, "seriesUnitIntervalInMins": 60, "instancesRequestedCount_recentHistory": "[9, 4, 2, 8, 8, 2, 3, 6, 5, 3, 2, 6]"}
PredictionForecastStartTime  : 2025-02-14T01:34:59.228Z
ProvisioningState            : Succeeded
StatusCode                   : HealthState/healthy
StatusMessage                : The pool is healthy.
```


### [REST](#tab/rest)

```HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyContainerPoolName}/runtimeViews/{runtimeView}?api-version=2025-03-01

{
  "properties": {
    "instanceCountSummary": [
      {
        "state": "Creating",
        "count": 0
      },
      {
        "state": "Starting",
        "count": 0
      },
      {
        "state": "Running",
        "count": 2
      },
      {
        "state": "Deallocating",
        "count": 3
      },
      {
        "state": "Deallocated",
        "count": 15
      },
      {
        "state": "Deleting",
        "count": 0
      }
    ],
    "provisioningState": "Succeeded",
    "status": {
      "code": "HealthState/healthy",
      "message": "The pool is healthy."
    },
    "prediction": {
      "forecastValues": {
        "instancesRequestedCount": [
          24,
          10,
          20,
          12,
          15,
          10,
          15,
          23,
          14,
          16,
          17,
          19
        ]
      },
      "forecastStartTime": "2025-02-14T01:34:59.228Z",
      "forecastInfo": "{\"forecastAccuracy\": 85, \"seriesUnitIntervalInMins\": 60, \"instancesRequestedCount_recentHistory\": \"[9, 4, 2, 8, 8, 2, 3, 6, 5, 3, 2, 6]\"}"
    }
  },
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyContainerPoolName}/runtimeViews/latest",
  "name": "pool",
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews"
}

```

---


## Next steps

Review the most [frequently asked questions](container-instances-standby-pools-faq.md) about standby pools for Azure Container Instances.