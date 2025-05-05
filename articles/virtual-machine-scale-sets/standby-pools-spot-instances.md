---
title: Use Spot Instances with Standby Pools in Azure
description: Learn how to configure and use Spot Instances with standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 5/5/2025
ms.reviewer: ju-shim
---

# Use Spot Instances with Standby Pools in Azure

Azure Spot Instances allow you to take advantage of unused Azure capacity at a significant cost savings. By combining Spot Instances with standby pools in Virtual Machine Scale Sets, you can optimize costs while maintaining scalability. However, there are specific considerations and limitations when using Spot Instances with standby pools:

- **Spot Instances can only be used with scale sets configured to use 100% Spot Instances** (not a mix of Spot and regular instances).
- **The scale set must have a delete eviction policy**.
- **Hibernated standby pools are not supported with Spot Instances**.

This article explains how to configure and use Spot Instances with standby pools, including details about supported VM states and their behavior.

## Prerequisites

Before you begin, ensure the following:

- You have an Azure subscription.
- You have a Virtual Machine Scale Set configured with 100% Spot Instances and a delete eviction policy.
- You have a standby pool created in the same region as your scale set.

## Supported VM States for Spot Instances in Standby Pools

When using Spot Instances with standby pools, you can configure the pool to use either a **running** or **deallocated** VM state. Each state has different behaviors:

### Running State
- The VMs in the standby pool remain in a running state.
- If a VM in the pool is evicted due to Spot capacity constraints, it will be deleted and replaced with a new instance.
- This configuration ensures that the pool is always ready to provide running instances to the scale set.

### Deallocated State
- The Spot Instances in the standby pool complete provisioning and are then shut down (deallocated).
- When the scale set requires new instances, they are automatically pulled from the pool and started in the scale set.
- This configuration reduces costs by not keeping the VMs running until they are needed.

> [!NOTE]
> There are no guarantees or SLAs when using Spot Instances in standby pools. Spot Instances are subject to eviction based on Azure capacity availability.

## Configure a Virtual Machine Scale Set with Spot Instances

To use Spot Instances with standby pools, you must configure your scale set to use 100% Spot Instances and set the eviction policy to delete. Follow these steps:

### [Azure Portal](#tab/portal)

1. Navigate to the [Azure portal](https://portal.azure.com/).
2. In the search bar, type **Virtual Machine Scale Sets** and select it from the results.
3. Click **+ Create** to create a new scale set.
4. Fill in the required fields:
   - **Instance type**: Select a Spot Instance type.
   - **Eviction policy**: Set the eviction policy to **Delete**.
   - **Spot allocation**: Ensure the scale set is configured to use 100% Spot Instances.
5. Complete the remaining configuration and click **Review + Create**, then **Create**.

### [Azure CLI](#tab/azurecli)
```azurecli
az vmss create \
  --name <scale-set-name> \
  --resource-group <resource-group-name> \
  --image UbuntuLTS \
  --instance-count 0 \
  --priority Spot \
  --eviction-policy Delete \
  --location <region>
```

### [PowerShell](#tab/powershell)
```azurepowershell
New-AzVmss -ResourceGroupName <resource-group-name> `
  -VMScaleSetName <scale-set-name> `
  -Location <region> `
  -Priority Spot `
  -EvictionPolicy Delete `
  -InstanceCount 0 `
  -ImageName "UbuntuLTS"
```

---

## Attach a standby pool


Once your scale set is configured with Spot Instances, you can attach a standby pool to it. For more information, see [create a standby pool](standby-pools-create.md)

## Next steps

Configure event monitoring and alertings using [Log Analytics](standby-pools-monitor-pool-events.md)