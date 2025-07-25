---
title: Azure PowerShell Script Sample - Change the RDP port range | Microsoft Docs
description: Azure PowerShell Script Sample - Changes the RDP port range of a deployed cluster.
services: service-fabric
ms.custom: devx-track-azurepowershell
author: athinanthny
ms.author: atsenthi
ms.service: azure-service-fabric
ms.topic: sample
ms.date: 03/19/2018
# Customer intent: "As a cloud administrator, I want to modify the RDP port range of my deployed cluster using PowerShell, so that I can maintain network connectivity without disrupting the VMs in my environment."
---

# Update the RDP port range values

This sample script changes the RDP port range values on the cluster node VMs after the cluster has been deployed.  Azure PowerShell is used so that the underlying VMs do not cycle.  The script gets the `Microsoft.Network/loadBalancers` resource in the cluster's resource group and updates the `inboundNatPools.frontendPortRangeStart` and `inboundNatPools.frontendPortRangeEnd` values. Customize the parameters as needed.

[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

If needed, install the Azure PowerShell using the instruction found in the [Azure PowerShell guide](/powershell/azure/).

## Sample script

[!code-powershell[main](../../../powershell_scripts/service-fabric/change-rdp-port-range/change-rdp-port-range.ps1 "Update the RDP port range values")]

## Script explanation

This script uses the following commands. Each command in the table links to command specific documentation.

| Command | Notes |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Gets the `Microsoft.Network/loadBalancers` resource. |
|[Set-AzResource](/powershell/module/az.resources/set-azresource)|Updates the `Microsoft.Network/loadBalancers` resource.|

## Next steps

For more information on the Azure PowerShell module, see [Azure PowerShell documentation](/powershell/azure/).

Additional Azure PowerShell samples for Azure Service Fabric can be found in the [Azure PowerShell samples](../service-fabric-powershell-samples.md).
