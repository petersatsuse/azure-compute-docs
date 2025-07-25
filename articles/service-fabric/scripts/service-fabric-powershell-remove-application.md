---
title: Remove application from a cluster in PowerShell
description: Azure PowerShell Script Sample - Remove an application from a Service Fabric cluster.
services: service-fabric
author: athinanthny
manager: chackdan
ms.service: azure-service-fabric
ms.topic: sample
ms.date: 01/18/2018
ms.author: atsenthi
ms.custom: mvc, devx-track-azurepowershell
# Customer intent: "As a cloud administrator, I want to remove an application from a Service Fabric cluster using PowerShell, so that I can manage application instances and free up resources in the environment."
---

# Remove an application from a Service Fabric cluster using PowerShell

This sample script deletes a running Service Fabric application instance and unregisters an application type and version from the cluster.  Deleting the application instance also deletes all the running service instances associated with that application. Customize the parameters as needed. 

If needed, install the Service Fabric PowerShell module with the [Service Fabric SDK](../service-fabric-get-started.md). 

## Sample script

[!code-powershell[main](../../../powershell_scripts/service-fabric/remove-application/remove-application.ps1 "Remove an application from a cluster")]

## Script explanation

This script uses the following commands. Each command in the table links to command specific documentation.

| Command | Notes |
|---|---|
| [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication) | Removes a running Service Fabric application instance from the cluster.  |
| [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype) | Unregisters a Service Fabric application type and version from the cluster. |

## Next steps

For more information on the Service Fabric PowerShell module, see [Azure PowerShell documentation](/powershell/azure/service-fabric/overview).

Additional PowerShell samples for Azure Service Fabric can be found in the [Azure PowerShell samples](../service-fabric-powershell-samples.md).
