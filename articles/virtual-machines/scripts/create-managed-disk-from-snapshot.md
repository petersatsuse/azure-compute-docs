---
title: Create managed disk from snapshot (Linux) - CLI sample
description: Azure CLI Script Sample - restore a disk from a snapshot and learn about the performance impact of restoring managed disk snapshots
author: roygara
ms.service: azure-disk-storage
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.date: 01/19/2024
ms.author: rogarana
ms.custom: mvc, devx-track-azurecli, linux-related-content
# Customer intent: As a cloud engineer, I want to restore a virtual machine from disk snapshots using CLI, so that I can efficiently create managed disks and understand the performance implications during the restoration process.
---

# Create a managed disk from a snapshot with CLI (Linux)

This article contains two scripts for creating a managed disk from a snapshot. The first script is for a managed disk with platform-managed keys and the second script is for a managed disk with customer-managed keys. Use these scripts to restore a virtual machine from snapshots of OS and data disks. Create OS and data managed disks from respective snapshots and then create a new virtual machine by attaching managed disks. You can also restore data disks of an existing VM by attaching data disks created from snapshots.

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

## Sample script

[!INCLUDE [cli-launch-cloud-shell-sign-in.md](~/reusable-content/ce-skilling/azure/includes/cli-launch-cloud-shell-sign-in.md)]

### Disks with platform-managed keys

:::code language="azurecli" source="~/azure_cli_scripts/virtual-machine/create-managed-disks-from-snapshot/create-managed-disks-from-snapshot.sh" id="FullScript":::

### Disks with customer-managed keys

```azurecli
#Provide the subscription Id of the subscription where you want to create Managed Disks
subscriptionId="<subscriptionId>"

#Provide the name of your resource group
resourceGroupName=myResourceGroupName

#Provide the name of the snapshot that will be used to create Managed Disks
snapshotName=mySnapshotName

#Provide the name of the new Managed Disks that will be create
diskName=myDiskName

#Provide the size of the disks in GB. It should be greater than the VHD file size.
diskSize=128

#Provide the storage type for Managed Disk. Premium_LRS or Standard_LRS.
storageType=Premium_LRS

#Provide the name of the target disk encryption set
diskEncryptionSetName=myName

#Provide the target disk encryption set resource group
diskEncryptionResourceGroup=myGroup

#Required for Premium SSD v2 and Ultra Disks
#Provide the Availability Zone you'd like the disk to be created in, default is 1
zone=1

#Set the context to the subscription Id where Managed Disk will be created
az account set --subscription $subscriptionId

#Get the snapshot Id 
snapshotId=$(az snapshot show --name $snapshotName --resource-group $resourceGroupName --query [id] -o tsv)

#Get the disk encryption set ID
diskEncryptionSetId=$(az disk-encryption-set show --name $diskEncryptionSetName --resource-group $diskEncryptionResourceGroup)

#Create a new Managed Disks using the snapshot Id
#Note that managed disk will be created in the same location as the snapshot
#To change the location, add the --location parameter
#If you're creating a Premium SSD v2 or an Ultra Disk, add "--zone $zone" to the end of the command
az disk create -g $resourceGroupName -n $diskName --source $snapshotId --disk-encryption-set $diskEncryptionSetID --location eastus2euap
```

## Performance impact - background copy process

When you create a managed disk from a snapshot, it starts a background copy process. You can attach a disk to a VM while this process is running but you'll experience performance impact (4k disks experience read impact, 512e experience both read and write impact) with higher latency, lower IOPS and throughput until background copy completes. For Ultra Disks and Premium SSD v2, you can check the status of the background copy process with the following commands:

> [!IMPORTANT]
> You can't use the following sections to get the status of the background copy process for disk types other than Ultra Disk or Premium SSD v2. Other disk types will always report 100%.

```azurecli
subscriptionId=yourSubscriptionID
resourceGroupName=yourResourceGroupName
diskName=yourDiskName
az account set --subscription $subscriptionId
az disk show -n $diskName -g $resourceGroupName --query [completionPercent] -o tsv
```

## Clean up resources

Run the following command to remove the resource group, VM, and all related resources.

```azurecli-interactive
az group delete --name myResourceGroupName
```

## Sample reference

This script uses following commands to create a managed disk from a snapshot. Each command in the table links to command specific documentation.

| Command | Notes |
|---|---|
| [az snapshot show](/cli/azure/snapshot) | Gets all the properties of a snapshot using the name and resource group properties of the snapshot. ID property is used to create managed disk.  |
| [az disk create](/cli/azure/disk) | Creates a managed disk using snapshot ID of a managed snapshot |

## Next steps

[Create a virtual machine by attaching a managed disk as OS disk](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

For more information on the Azure CLI, see [Azure CLI documentation](/cli/azure).

More virtual machine and managed disks CLI script samples can be found in the [Azure Linux VM documentation](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
