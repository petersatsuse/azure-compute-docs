---
title: Get Prediction Results for Standby pools for Virtual Machine Scale Sets (Preview)
description: Learn how to prediction standby pools to help right size your pool for Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: conceptual
ms.date: 5/6/2025
ms.reviewer: ju-shim
---

# Get prediction results for Standby pools (Preview)

> [!IMPORTANT]
> Prediction results for standby pools is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

To effectively manage and optimize your standby pool for Virtual Machine Scale Sets, you can use the Standby Pool runtime view APIs to retrieve prediction results. These results, **available 2-3 weeks after creating the standby pool**, provide insights into the predicted number of instances that will be requested from the pool for each hour over a 12-hour period. The predictions include the accuracy of the forecast and a historical view of instances requested from the pool over the past 12 hours, helping you make informed decisions to right-size your standby pool and improve operational efficiency.

While the prediction results provide valuable insights, they are not a guarantee and should be treated as a suggested size for your pool. The actual number of instances requested from the pool may vary depending on the specific demands of your workload. Additionally, the longer the prediction engine is able to monitor and analyze your workload trends, the more accurate and reliable the prediction results will become over time.

### [CLI](#tab/cli)

```azurecli
az standby-vm-pool status --resource-group myResourceGroup --name myStandbyPool

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
  "predictionForcastInfo":
  "predictionForcastStartTime":
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews"
}

```


### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzStandbyVMPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool

Id: /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/mmyStandbyPool/runtimeViews/latest
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
            "count": 1
        },
        {
            "state": "Deallocating",
            "count": 3
        },
        {
            "state": "Deallocated",
            "count": 7
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
            "state": "Deleting",
            "count": 0
        }
       ]
     }
Name                         : latest
PredictionForecastInfo       : 
PredictionForecastStartTime  : 2025-02-14T01:34:59.228Z
ProvisioningState            : Succeeded
ResourceGroupName            : myResourceGroup
StatusCode                   : HealthState/healthy
StatusMessage                :
SystemDataCreatedAt          :
SystemDataCreatedBy          :
SystemDataCreatedByType      :
Type                         : Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews

```


### [REST](#tab/rest)

```HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyVirtualMachinePoolName}/runtimeViews/{runtimeView}?api-version=2025-03-01

{
  "properties": {
        "instanceCountSummary": [
      {
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
            "state": "Hibernating",
            "count": 0
          },
          {
            "state": "Hibernated",
            "count": 0
          },
          {
            "state": "Deleting",
            "count": 0
          }
        ]
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
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyPoolName}/runtimeViews/latest",
  "name": "pool",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews",
  "systemData": {
    "createdBy": "pooluser@microsoft.com",
    "createdByType": "User",
    "createdAt": "2025-02-14T23:31:59.679Z",
    "lastModifiedBy": "pooluser@microsoft.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2025-02-14T23:31:59.679Z"
  }
}


```

---


## Next steps

Review the most [frequently asked questions](standby-pools-faq.md) about standby pools for Virtual Machine Scale Sets.