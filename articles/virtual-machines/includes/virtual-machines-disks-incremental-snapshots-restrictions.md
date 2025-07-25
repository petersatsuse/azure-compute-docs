---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 12/11/2024
 ms.author: rogarana
 ms.custom: include file
# Customer intent: "As a cloud administrator, I want to create and manage incremental snapshots of virtual disks, so that I can efficiently utilize storage and ensure data integrity while complying with the snapshot limitations and performance impacts."
---

- Incremental snapshots currently can't be moved between subscriptions.
- You can currently only generate SAS URIs of up to five snapshots of a particular snapshot family at any given time.
- You can't create an incremental snapshot for a particular disk outside of that disk's subscription.
- Incremental snapshots can't be moved to another resource group. But, they can be copied to another resource group or region.
- Up to seven incremental snapshots per disk can be created every five minutes.
- A total of 500 incremental snapshots can be created for a single disk. The 500 quota limit isn't over the lifetime of a disk, but at any given point in time. You can always delete older snapshots of a disk to make room for newer snapshots.
- You can't get the changes between snapshots taken before and after you changed the size of the parent disk across 4-TB boundary. For example, You took an incremental snapshot `snapshot-a` when the size of a disk was 2 TB. Now you increased the size of the disk to 6 TB and then took another incremental snapshot `snapshot-b`. You can't get the changes between `snapshot-a` and `snapshot-b`. You have to download the full copy of `snapshot-b` created after the resize. Subsequently, you can get the changes between `snapshot-b` and snapshots created after `snapshot-b`.
- When you create a managed disk from a snapshot, it starts a background copy process. You can attach a disk to a VM while this process is running but you'll experience [performance impact](/azure/virtual-machines/premium-storage-performance#latency). You can use CompletionPercent property to [check the status of the background copy](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot#performance-impact---background-copy-process) only for Ultra Disks and Premium SSD v2 disks.

### Incremental snapshots of Premium SSD v2 and Ultra Disks

Incremental snapshots of Premium SSD v2 and Ultra Disks have the following extra restrictions:

- Snapshots with a 512 logical sector size are stored as VHD, and can be used to create any disk type. Snapshots with a 4096 logical sector size are stored as VHDX and can only be used to create Ultra Disks and Premium SSD v2 disks, they can't be used to create other disk types. To determine which sector size your snapshot is, see [check sector size](#check-sector-size).
- Up to five disks may be simultaneously created from a snapshot of a Premium SSD v2 or an Ultra Disk.
- When an incremental snapshot of either a Premium SSD v2 or an Ultra Disk is created, a background copy process for that disk is started. While a background copy is ongoing, you can have up to three total snapshots pending. The process must complete before any more snapshots of that disk can be created.
- Incremental snapshots of a Premium SSD v2 or an Ultra disk can't be used immediately after they're created. The background copy must complete before you can create a disk from the snapshot. See [Check snapshot status](#check-snapshot-status) for details.
- When you increase the size of a Premium SSD v2 or an Ultra disk, any incremental snapshots that are under background copy will fail. 
- When you attach a Premium SSD v2 or Ultra disk created from snapshot to a running Virtual Machine while CompletionPercent property hasn't reached 100, the disk suffers performance impact. Specifically, if the disk has a 4k sector size, it may experience slower read. If the disk has a 512e sector size, it may experience slower read and write. To track the progress of this background copy process, see the check disk status section of either the Azure [PowerShell sample](/azure/virtual-machines/scripts/virtual-machines-powershell-sample-create-managed-disk-from-snapshot#performance-impact---background-copy-process) or the [Azure CLI](/azure/virtual-machines/scripts/virtual-machines-powershell-sample-create-managed-disk-from-snapshot#performance-impact---background-copy-process).
- You can have up to 50 currently active background copies per subscription. The following activities count against the total number of active background copies: 
    - Creating Premium SSD v2 and Ultra disks with snapshots
    - Creating Premium SSD v2 and Ultra Disks with restore points
    - Uploading VHDX files into Premium SSD v2 and Ultra Disks
    - Converting existing Premium SSD disks to Premium SSD v2 disks.

> [!NOTE]
> Normally, when you take an incremental snapshot, and there aren't any changes, the size of that snapshot is 0 MiB. Currently, empty snapshots of disks with a 4096 logical sector size instead have a size of 6 MiB, when they'd normally be 0 MiB.
