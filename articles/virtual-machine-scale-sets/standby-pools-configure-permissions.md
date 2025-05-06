---
title: Configure role permissions for standby pools
description: Learn how to configure role-based access control (RBAC) permissions for standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 5/6/2025
---

# Configure role permissions for standby pools

Standby pools require specific permissions to create and manage resources in your subscription. Without the correct permissions, standby pools will not function properly. This article explains how to configure role-based access control (RBAC) permissions for standby pools and provides guidance for scenarios where additional permissions may be required.

> [!IMPORTANT]
> These permissions may not fully encompass all scenarios. If your standby pool uses specific resources, such as compute gallery images in other subscriptions, ensure the standby pool resource provider has access to those resources.

## Feature registration

> [!NOTE] 
> The StandbyPoolVMPoolPreview feature flag is required to enable standby pool functionality in the Azure portal. This requirement is temporary and will be removed in the future.

Before configuring permissions, register the standby pool resource provider with your subscription. Use Azure Cloud Shell to run the following commands:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.StandbyPool
Register-AzProviderFeature -FeatureName StandbyVMPoolPreview -ProviderNameSpace Microsoft.StandbyPool
```

Registration can take up to 30 minutes to complete. You can rerun the commands to check the registration status.

## Basic permissions 
To allow standby pools to create and manage virtual machines in your subscription, assign the appropriate permissions to the standby pool resource provider.

1) In the Azure portal, navigate to your subscriptions.
2) Select the subscription you want to adjust permissions.
3) Select **Access Control (IAM)**.
4) Select **Add** and **Add role assignment**.
5) Under the **Role** tab, search for **Virtual Machine Contributor** and select it.
6) Move to the **Members** Tab.
7) Select **+ Select members**.
8) Search for **Standby Pool Resource Provider** and select it.
9) Move to the **Review + assign** tab.
10) Apply the changes. 
11) Repeat the above steps and assign the **Network Contributor** role and the **Managed Identity Operator** role to the Standby Pool Resource Provider. If you're using images stored in Compute Gallery assign the **Compute Gallery Sharing Admin** and **Compute Gallery Artifacts Publisher** roles as well.

For more information on assigning roles, see [assign Azure roles using the Azure portal](/azure/role-based-access-control/quickstart-assign-role-user-portal).

## Cross-subscription resources
If your standby pool uses resources, such as compute gallery images, stored in a different subscription, ensure the standby pool resource provider has access to those resources. You may need to assign roles in the other subscription to grant the required permissions.

## Next steps
For more information on assigning roles, see Assign Azure roles using the Azure portal. 