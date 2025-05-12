---
title: 
description: 
author: roygara
ms.author: vishalprayag
ms.date: 05/12/2025
ms.topic: how-to
ms.service: azure-disk-storage
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, innovation-engine
---
# Use Premium SSD v2 with VMs in Availability Set

## Introduction:

Premium SSD v2 managed disks are supported with VMs in availability sets to enhance the high availability and resilience of your applications. When VMs using Premium SSD v2 are part of an availability set, the platform ensures that their disks are automatically distributed across multiple storage Fault Domains (FDs). This distribution minimizes the risk of a single point of failure.

![A diagram of a computer  AI-generated content may be incorrect.](media/image1.png)

Availability sets have fault isolation for many possible failures, to minimize single points of failure and to offer high availability.  If there is a failure in one storage Fault Domain (FD), only the VM instances with Premium SSD v2 disks on that specific Fault Domain are affected. The other VM instances, whose disks are placed on separate Fault Domains, remain unaffected and continue to operate normally. Availability sets are still susceptible to certain shared infrastructure failures, like datacenter network failures, physical hardware failures or power interruptions which can affect multiple fault domains.

In a production scenario with three VMs deployed in an availability set using Premium SSD v2 with Fault Domain 3, VMs and disks are distributed across multiple fault domains. If one storage fault domain fails, only the VMs associated with that domain are affected, while the rest remain operational. The unaffected VMs will stay online, preserving the application's availability.

If a Premium SSD v2 disk originally resides in one fault domain but is attached to a VM in another fault domain, the system initiates a background copy process. This operation moves the disk to align with the VM's fault domain, ensuring consistent compute and storage fault domain alignment for improved reliability and availability.

![A diagram of a computer  AI-generated content may be incorrect.](media/image2.png)

For example, as shown in the diagram above, if a disk located in FD1 is attached to a VM in FD1 and is later detached and attached to a VM in FD2, the system will automatically trigger a background copy of the disk to move it from FD1 to FD2 for compute and storage fault domain alignment. This background move process can take up to 24 hours to complete. 

## Regional availability:

Premium SSD v2 support for VMs in an Availability Set is currently available only in following regions that do not support Availability Zones: 

- Australia Southeast
- Canada East
- North Central US
- West US

## Limitations:

-  A subscription must be registered for the required feature to use Premium SSD v2 with VMs in Availability Sets in non-zonal regions. Follow the instructions here to complete the registration.
- Only one background data copy per disk is allowed at a time. When attaching a disk to a VM in an Availability Set (AvSet), the system may initiate a background copy for Fault Domain (FD) alignment. If a detach-and-attach is attempted during an ongoing move, the operation fails with an error. To avoid this, wait for the move to complete, or set the [OptimizedForFrequentAttach](/dotnet/api/microsoft.azure.management.compute.models.diskupdate.optimizedforfrequentattach) disk property to skip FD-alignment background copies for future attachments. For more information on OptimizedForFrequentAttach, follow the instructions here.
- A disk created from a snapshot cannot be attached to VMs in an Availability Set while it is still undergoing background data copy from a snapshot. You must wait until the copy process is fully complete before attaching the disk. To check the status of background data copy from a snapshot, follow the instructions [here](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot).
- ***Disk size increase and changing customer-managed key (CMK) is not supported while a background data copy for Fault Domain alignment is in progress.***

## **AzureCLI:**

### Create new Availability Set with Premium SSD v2:

Create a resource group: 

_az group create --name myResourceGroup --location myLocation_ 

The number of Storage Fault domains varies by region. The following command retrieves a list of fault domains per region: 

_az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location,  MaximumFaultDomainCount:capabilities[0].value}' -o Table_ 

Create the availability set:  

_az vm availability-set create -n myAvSet -g myResourceGroup --platform-fault-domain-count 3 --platform-update-domain-count 20_ 

_Note: The value for 'platform-fault-domain-count' should be determined based on the number of_ available_ Storage Fault Domains in a given region. Please follow Step b to determine the number of available Fault Domains per region._ 

Create a VM: 

_az vm create -n myVMname -g myResourceGroupName --availability-set myAvSetName --image Win2016Datacenter --count MyCount_ 

_Attach a new premium SSD v2 disk to existing VMs in an availability set_ 

*az vm disk attach -g MyResourceGroupName --vm-name MyVMname --name MyDiskName --new --sku PremiumV2_LRS --size-gb MySize *

### Attach existing Premium SSD v2 disk to existing VMs in an Availability Set:

_az vm disk attach -g MyResourceGroupName --vm-name MyVMname --disks MyDiskName _

## **PowerShell:**

### Create new Availability Set with Premium SSD v2:

Create a resource group: 

_New-AzResourceGroup -Name myResourceGroup -Location myLocation_ 

The number of Storage Fault domains varies by region. The following command retrieves a list of fault domains per region: 

- Get*-AzComputeResourceSku | Where-Object {$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'} | Select-Object @{Name='Location'; Expression={$_.locationInfo[0].location}}, @{Name='MaximumFaultDomainCount'; Expression={$_.capabilities[0].value}}* 
- Create the availability set:  

_Note: The value for 'platform-fault-domain-count' should be determined based on the number of_ available_ Storage Fault Domains in a given region. Please follow Step b to determine the number of available Fault Domains per region._ 

- _Create a VM:_
- _Attach a new premium SSD v2 disk to existing VMs in an availability set:_

### Attach existing Premium SSD v2 disk to existing VMs in an Availability Set:

_$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName_

_$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName_

_$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun_

_Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName_

## **Portal:**

Sign in to the [Azure portal](https://portal.azure.com/).

Create an Availability Set with the 'Use managed disk' option set to 'Yes (Aligned)'

![A screenshot of a computer  AI-generated content may be incorrect.](media/image3.png)

Navigate to Virtual machines and follow the normal VM creation process.

On the Basics page, select a supported region and set Availability options to Availability set.

Select Availability set

![A screenshot of a computer  AI-generated content may be incorrect.](media/image4.png)

Fill in the rest of the values on the page as you like.

Proceed to the Disks page.

Under **Data disks** select **Create and attach a new disk**.

![A screenshot of a computer  AI-generated content may be incorrect.](media/image5.png)

Select the **Disk SKU** and select **Premium SSD v2**.

![A screenshot of a computer  AI-generated content may be incorrect.](media/image6.png)

Select whether you'd like to deploy a 4k or 512 logical sector size.

![A screenshot of a computer  AI-generated content may be incorrect.](media/image7.png)

Proceed through the rest of the VM deployment, making any choices that you desire.  

You've now deployed an Availability Set VM with a premium SSD v2.

## Register Premium SSD v2 with VMs in Availability Sets in supported Non-Zonal Regions:

To use Premium SSD v2 with VMs in Availability Sets in non-zonal regions, ensure your subscription is registered for the required feature:

- Command to register the feature:

  _az feature registration create --namespace Microsoft.Compute --name PV2WithAVSetRegionWithoutZone_

- Check registration status:

  _az feature registration show --provider Microsoft.Compute --name PV2WithAVSetRegionWithoutZone_

## Optimize Background Data Copy of the disk:

Change optimized-for-frequent-attach disk property:<br>If the workload involves frequent disk detach and attach operations between VMs of the same Availability Set or between VMs in different Availability Sets, we recommend enabling the optimized-for-frequent-attach property. Setting this property to true prevents the system from triggering a background copy of the disk for Fault Domain alignment during reattachments. optimized-for-frequent-attach can be set when creating a new unattached disk or update it later for an existing disk. If the disk is currently attached to a VM, first detach the disk. Update the optimized-for-frequent-attach disk property and then reattach the disk to the VM. 

#### Instructions to Set or Update Optimized-for-Frequent-Attach on Disks:

##### Set the property while creating a new unattached disk:

az disk create \ 

--name myDiskName \ 

--resource-group myResourceGroup \ 

--location myLocation \ 

--sku PremiumV2_LRS \ 

--size-gb myGB \ 

--optimized-for-frequent-attach true 

#####  Update the property on an existing unattached disk:

az disk update \ 

--name myDiskName \ 

--resource-group myResourceGroup \ 

--set optimizedForFrequentAttach=true
