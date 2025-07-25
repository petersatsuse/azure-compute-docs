---
title: Configure MSP via the Portal
description: View examples of configuring Metadata Security Protocol (MSP) for a new or existing virtual machine via the Azure portal.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: As an IT administrator, I want to configure Metadata Security Protocol for virtual machines via the portal, so that I can enhance the security and compliance of my deployments.
---

# Configure MSP via the portal

Azure portal has preview support for configuring some aspects of Metadata Security Protocol (MSP).

> [!NOTE]
> Currently, the portal is available only for subscriptions registered with the `Microsoft.Compute/ProxyAgentPreview` Azure Feature Exposure Control (AFEC) flag.

## Deploy a VM with MSP

On the **Create a virtual machine** pane, make selections in the **Metadata Security Protocol** area.

![Screenshot that shows selections for deploying a new virtual machine with Metadata Security Protocol.](../images/portal-greenfield.png)

> [!NOTE]
> The portal currently supports only [inline/basic](../configuration.md#inline-configuration) configuration.

## Enable MSP on an existing VM

Go to your virtual machine, and then go to **Settings** > **Configuration**. Make your selections in the **Metadata Security Protocol** area.

![Screenshot that shows selections for enabling Metadata Security Protocol on an existing virtual machine.](../images/portal-brownfield.png)
