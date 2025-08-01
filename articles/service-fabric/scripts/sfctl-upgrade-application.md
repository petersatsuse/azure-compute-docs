---
title: Update an application on a cluster in sfctl
description: Service Fabric CLI Script Sample - Update an application with a new version. This example also upgrades a deployed application with the new bits.
services: service-fabric
author: athinanthny
manager: chackdan
ms.service: azure-service-fabric
ms.topic: sample
ms.date: 12/06/2017
ms.author: atsenthi
# Customer intent: As a cloud administrator, I want to update an application on a cluster using a CLI script, so that I can efficiently deploy new versions and ensure my applications are up to date.
---

# Update an application using the Service Fabric CLI

This sample script uploads a new version of an existing application, and then upgrades a deployed application with the new bits.

[!INCLUDE [links to azure cli and service fabric cli](../includes/service-fabric-sfctl.md)]

## Sample script

[!code-sh[main](../../../cli_scripts/service-fabric/upgrade-application/upgrade-application.sh "Upload and update an application on a Service Fabric cluster")]

## Next steps

For more information, see the [Service Fabric CLI documentation](../service-fabric-cli.md).

Additional Service Fabric CLI samples for Azure Service Fabric can be found in the [Service Fabric CLI samples](../samples-cli.md).
