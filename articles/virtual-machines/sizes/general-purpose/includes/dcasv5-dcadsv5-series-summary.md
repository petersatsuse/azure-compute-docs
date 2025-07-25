---
title: DCadsv5-series summary include
description: Include file containing a summary of the DCadsv5-series size family.
services: virtual-machines
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 04/11/2024
ms.author: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to understand the capabilities of the DCadsv5-series virtual machines, so that I can assess their suitability for running confidential workloads with enhanced security features."
---

These confidential VMs use AMD's third-Generation EPYC<sup>TM</sup> 7763v processor in a multi-threaded configuration with up to 256 MB L3 cache. These processors can achieve a boosted maximum frequency of 3.5 GHz. Both series offer Secure Encrypted Virtualization-Secure Nested Paging (SEV-SNP). SEV-SNP provides hardware-isolated VMs that protect data from other VMs, the hypervisor, and host management code. Confidential VMs offer hardware-based VM memory encryption. These series also offer OS disk pre-encryption before VM provisioning with different key management solutions. 
