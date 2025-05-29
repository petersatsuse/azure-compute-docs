---
title: Disable MSP
description: Disabling MSP
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: "As an IT administrator, I want to disable the Managed Service Proxy (MSP) for my virtual machines, so that I can manage security settings more effectively and allow requests to flow normally."
---

# Disable MSP

To disable MSP, set the `securityProfile.proxyAgentSettings.enabled` to `false`.

## Via REST API

```http
PATCH https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01
{
  "properties": {
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": false,
      }
    },
  }
}
```

## Via Azure portal

Portal doesn't allow disabling MSP completely at this time, but you can disable actual enforcement of security requirements to allow requests to flow normally:

![Screenshot of the Azure portal screen.](../images/disable-msp-portal.png)
