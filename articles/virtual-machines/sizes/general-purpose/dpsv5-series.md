---
title: Dpsv5 size series
description: Information on and specifications of the Dpsv5-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to review the specifications and features of the Dpsv5 size series, so that I can select the appropriate virtual machine sizes for my organization's workloads."
---

# Dpsv5 sizes series

[!INCLUDE [dpsv5-summary](./includes/dpsv5-series-summary.md)]

## Host specifications
[!INCLUDE [dpsv5-series-specs](./includes/dpsv5-series-specs.md)]

## Feature support

Premium Storage: Supported<br>
Premium Storage caching: Supported<br>
Live Migration: Supported<br>
Memory Preserving Updates: Supported<br>
VM Generation Support: Generation 2<br>
Accelerated Networking: Supported<br>
Ephemeral OS Disks: Not supported<br>
Nested Virtualization: Not supported<br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2ps_v5 | 2 | 8 |
| Standard_D4ps_v5 | 4 | 16 |
| Standard_D8ps_v5 | 8 | 32 |
| Standard_D16ps_v5 | 16 | 64 |
| Standard_D32ps_v5 | 32 | 128 |
| Standard_D48ps_v5 | 48 | 192 |
| Standard_D64ps_v5 | 64 | 208 |

#### VM Basics resources
- [What are vCPUs](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series. For similar sizes with local storage, see the [Dpdsv6-series](./dpdsv6-series.md).
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).



### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_D2ps_v5 | 4 | 3750 | 85 | 10000 | 1200 |
| Standard_D4ps_v5 | 8 | 6400 | 145 | 20000 | 1200 |
| Standard_D8ps_v5 | 16 | 12800 | 290 | 20000 | 1200 |
| Standard_D16ps_v5 | 32 | 25600 | 600 | 40000 | 1200 |
| Standard_D32ps_v5 | 32 | 51200 | 865 | 80000 | 2000 |
| Standard_D48ps_v5 | 32 | 76800 | 1315 | 80000 | 3000 |
| Standard_D64ps_v5 | 32 | 80000 | 1735 | 80000 | 3000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_D2ps_v5 | 2 | 12500 |
| Standard_D4ps_v5 | 2 | 12500 |
| Standard_D8ps_v5 | 4 | 12500 |
| Standard_D16ps_v5 | 4 | 12500 |
| Standard_D32ps_v5 | 8 | 16000 |
| Standard_D48ps_v5 | 8 | 24000 |
| Standard_D64ps_v5 | 8 | 40000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]


