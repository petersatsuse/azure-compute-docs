---
 title: Include file
 description: Include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: Include
 ms.date: 05/28/2025
 ms.author: rogarana
 ms.custom: Include file, references_regions
---
> [!IMPORTANT]
> This limitation doesn't apply to Premium SSD v2 or Ultra Disks.
>
> If a Standard HDD, Standard SSD, or Premium SSD disk is 4 TiB or less, deallocate your VM and detach the disk before you expand it beyond 4 TiB. If one of those disk types is already greater than 4 TiB, you can expand it without deallocating the VM and detaching the disk.

- Only supported for data disks.
- Not supported for shared disks.
- Install and use either:
    - The [latest Azure CLI](/cli/azure/install-azure-cli).
    - The [latest Azure PowerShell module](/powershell/azure/install-azure-powershell).
    - The [Azure portal](https://portal.azure.com/).
    - Or an Azure Resource Manager template with an API version that's `2021-04-01` or newer.
- Not available on some classic VMs. Use [this script](#expanding-without-downtime-classic-vm-sku-support) to get a list of classic VM products that support expanding without downtime.

### Expand with Ultra Disk and Premium SSD v2

Expanding Ultra Disks and Premium SSD v2 disks without downtime has the following limitations:

- You can't expand a disk while a [background copy](../scripts/create-managed-disk-from-snapshot.md#performance-impact---background-copy-process) of data is also occurring on that disk. An example is when a disk is being backfilled from [snapshots](/azure/virtual-machines/disks-incremental-snapshots?tabs=azure-cli).
- You can expand VMs by using NVMe controllers with Ultra Disks or Premium SSD v2 disks without downtime only with this public preview. Use this preview only to test the functionality of expanding without downtime. Don't expand VMs in production.

In the following regions, you can expand VMs by using [NVMe controllers](../nvme-overview.md) for Ultra Disks or Premium SSD v2 disks without downtime. Use either the Azure portal, the Azure CLI, or an Azure PowerShell module:

- Southeast Asia
- Brazil South
- Canada Central
- Germany West Central
- Central India (not currently supported on V6 VMs)

In the following regions, you can expand VMs only by using [NVMe controllers](../nvme-overview.md) for Ultra Disks or Premium SSD v2 disks without downtime. Use only the Azure CLI or an Azure PowerShell module. You can't currently use the Azure portal:

- East Asia
- West Central US (not currently supported on V6 VMs)

Allow up to 10 minutes for the correct size to be reflected in Windows VMs and Linux VMs. For Linux VMs, you must perform a [Linux rescan function](/azure/virtual-machines/linux/expand-disks?tabs=ubuntu#detecting-a-changed-disk-size). For a Windows VM that doesn't have a workload, you must perform a [Windows rescan function](/windows-hardware/drivers/devtest/devcon-rescan). You can rescan immediately, but if the time is within 10 minutes, you might need to rescan again to display the correct size.
