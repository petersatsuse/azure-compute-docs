---
title: Falsv6-series summary include file
description: Include file for Falsv6-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/30/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to understand the capabilities of the Falsv6-series VMs, so that I can determine their suitability for high-performance workloads and optimize compute costs for licensed software in my organization."
---
The Falsv6-series utilizes AMD's fourth Generation EPYC™ 9004 processor that can achieve a boosted maximum frequency of 3.7 GHz with up to 320 MB L3 cache. The Falsv6 VM series comes without Simultaneous Multithreading (SMT), meaning a vCPU is now mapped to a full physical core, allowing software processes to run on dedicated and uncontested resources. These new full core VMs suit workloads demanding the highest CPU performance. Falsv6-series offers up to 64 full core vCPUs and 128 GiB of RAM. This series is optimized for scientific simulations, financial and risk analysis, gaming, rendering and other workloads able to take advantage of the exceptional performance. Customers running software licensed on per-vCPU basis can use these VMs to optimize compute costs within their infrastructure.
