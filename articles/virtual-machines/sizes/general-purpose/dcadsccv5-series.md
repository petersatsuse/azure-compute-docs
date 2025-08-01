---
title: DCads_cc_v5 size series
description: Information on and specifications of the DCads_cc_v5-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to review the specifications and features of the DCads_cc_v5 VM sizes, so that I can select the appropriate virtual machine configuration for my workloads.
---

# DCads_cc_v5 sizes series
>[!NOTE]
>This VM series is currently in **Preview**. See the [Preview Terms Of Use | Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability. 

[!INCLUDE [dcads_cc_v5-summary](./includes/dcadsccv5-series-summary.md)]

## Host specifications
[!INCLUDE [dcads_cc_v5-series-specs](./includes/dcadsccv5-series-specs.md)]

## Feature support
[Trusted Launch](../../trusted-launch.md): Not Supported <br>
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_DC4ads_cc_v5 | 4 | 16 |
| Standard_DC8ads_cc_v5 | 8 | 32 |
| Standard_DC16ads_cc_v5 | 16 | 64 |
| Standard_DC32ads_cc_v5 | 32 | 128 |
| Standard_DC48ads_cc_v5 | 48 | 192 |
| Standard_DC64ads_cc_v5 | 64 | 256 |
| Standard_DC96ads_cc_v5 | 96 | 384 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) |
| --- | --- | --- |
| Standard_DC4ads_cc_v5 | 1 | 150 |
| Standard_DC8ads_cc_v5 | 1 | 300 |
| Standard_DC16ads_cc_v5 | 1 | 600 |
| Standard_DC32ads_cc_v5 | 1 | 1200 |
| Standard_DC48ads_cc_v5 | 1 | 1800 |
| Standard_DC64ads_cc_v5 | 1 | 2400 |
| Standard_DC96ads_cc_v5 | 1 | 3600 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) |
| --- | --- | --- | --- |
| Standard_DC4ads_cc_v5 | 8 | 6400 | 144 |
| Standard_DC8ads_cc_v5 | 16 | 12800 | 200 |
| Standard_DC16ads_cc_v5 | 32 | 25600 | 384 |
| Standard_DC32ads_cc_v5 | 32 | 51200 | 768 |
| Standard_DC48ads_cc_v5 | 32 | 76800 | 1152 |
| Standard_DC64ads_cc_v5 | 32 | 80000 | 1200 |
| Standard_DC96ads_cc_v5 | 32 | 80000 | 1600 |

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

| Size Name | Max NICs (Qty.) |
| --- | --- |
| Standard_DC4ads_cc_v5 | 2 |
| Standard_DC8ads_cc_v5 | 4 |
| Standard_DC16ads_cc_v5 | 4 |
| Standard_DC32ads_cc_v5 | 8 |
| Standard_DC48ads_cc_v5 | 8 |
| Standard_DC64ads_cc_v5 | 8 |
| Standard_DC96ads_cc_v5 | 8 |

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


