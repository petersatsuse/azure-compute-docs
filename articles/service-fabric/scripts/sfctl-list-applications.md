---
title: List applications on a cluster in sfctl
description: Service Fabric CLI Script Sample - List the applications provisioned on a Service Fabric cluster.
services: service-fabric
author: athinanthny
manager: chackdan
ms.service: azure-service-fabric
ms.topic: sample
ms.date: 04/13/2018
ms.author: atsenthi
# Customer intent: As a cloud administrator, I want to use a script to list all applications on a Service Fabric cluster, so that I can manage and monitor the provisioned applications effectively.
---

# List applications running in a Service Fabric cluster

This sample script connects to a Service Fabric cluster and then lists all of the provisioned applications.

[!INCLUDE [links to azure cli and service fabric cli](../includes/service-fabric-sfctl.md)]

## Sample script

[!code-sh[main](../../../cli_scripts/service-fabric/list-application/list-application.sh "List provisioned applications from a cluster")]

## Next steps

For more information, see the [Service Fabric CLI documentation](../service-fabric-cli.md).

Additional Service Fabric CLI samples for Azure Service Fabric can be found in the [Service Fabric CLI samples](../samples-cli.md).
