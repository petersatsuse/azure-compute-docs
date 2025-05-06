---
title: Use Spot Instances in standby pools
description: Learn how to configure and use Spot Instances with standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: ju-shim
---

# Use Spot Instances in standby pools

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

Azure Spot Instances allow you to take advantage of unused Azure capacity at a significant cost savings. By combining Spot Instances with standby pools in Virtual Machine Scale Sets, you can optimize costs while maintaining scalability. However, there are specific considerations and limitations when using Spot Instances with standby pools:

- Standby pools only supports spot instances when used with scale sets configured to use 100% Spot Instances (not a mix of Spot and regular instances).
- Standby pools only supports spot instances when used with a scale set that has the spot eviction policy set as delete. Deallocate eviction policy is not currently supported.
- Standby pools using spot instances cannot use a hibernated VM state. Only running and deallocated pool state are supported.

This article explains how to configure and use Spot Instances with standby pools, including details about supported VM states and their behavior.

> [!NOTE]
> Creating and attaching a standby pool to a scale set using spot instances is not yet supported in the Azure portal. 

## Supported VM states for Spot Instances in standby pools

When using Spot Instances with standby pools, you can configure the pool to use either a **running** or **deallocated** VM state. Hibernate is not supported when using spot instances. Each state has different behaviors:

### Running state
- The VMs in the standby pool remain in a running state.
- If a VM in the pool is evicted due to Spot capacity constraints, it will be deleted and replaced with a new instance.
- This configuration ensures that the pool is always ready to provide running instances to the scale set as long as you have a minimum ready capacity configured or have not reached the max ready capacity configured on your standby pool. 

### Deallocated state
- The Spot Instances in the standby pool complete provisioning and are then shut down (deallocated).
- When the scale set requires new instances, they are automatically pulled from the pool and started in the scale set.
- This configuration reduces costs by releasing the compute resources associated with the VMs after they have finished all post provisioning steps. 

> [!NOTE]
> There are no guarantees or SLAs when using Spot Instances in standby pools. Spot Instances are subject to eviction based on Azure capacity availability.

## Configure a Virtual Machine Scale Set with Spot Instances

To use Spot Instances with standby pools, you must configure your scale set to use 100% Spot Instances and set the eviction policy to delete. 

### [Azure portal](#tab/portal)

> [!NOTE]
> Adding a standby pool to a Virtual Machine Scale Set in the Azure portal is not yet supported. Instead, create your scale set then use an alternative SDK to add the standby pool after scale set creation. 

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

Once your scale set is configured with Spot Instances, there is not additional parameters or settings you need to configure directly in your standby pool. Simply create and attach the standby pool to your scale set with spot instances and the instances within the pool will take on the properties configured in your scale set. For more information, see [create a standby pool](standby-pools-create.md).

## Next steps

Configure event monitoring and alertings using [Log Analytics](standby-pools-monitor-pool-events.md).