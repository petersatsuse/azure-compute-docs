---
title: ECas_cc_v5 and ECads_cc_v5-series summary include
description: Include file containing a summary of the ECas_cc_v5 and ECads_cc_v5-series size family.
services: virtual-machines
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 04/18/2024
ms.author: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to understand the features and capabilities of ECas_cc_v5 and ECads_cc_v5-series VMs, so that I can effectively utilize their parent-child model for higher isolation and security in my deployments.
---
Confidential child capable VMs allow you to borrow resources from the parent VM you deploy, to create AMD SEV-SNP protected child VMs. The parent VM has almost complete feature parity with any other general purpose Azure VM (for example, E-series VMs). This parent-child deployment model can help you achieve higher levels of isolation from the Azure host and parent VM. These confidential child capable VMs are built on the same hardware that powers our Azure confidential VMs. Azure confidential VMs are now generally available.