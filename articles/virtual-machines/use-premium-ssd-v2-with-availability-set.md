---
title: Use Premium SSD v2 with VMs in availability set
description: Instructions on how to use Premium SSD v2 with VMs in availability set
author: vishalprayag
ms.author: roygara 
ms.date: 05/12/2025
ms.topic: how-to
ms.service: azure-disk-storage
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, innovation-engine
---
# Use Premium SSD v2 with VMs in Availability Set

## Introduction:

Premium SSD v2 managed disks are supported with VMs in availability sets to enhance the high availability and resilience of your applications. When VMs using Premium SSD v2 are part of an availability set, the platform ensures that their disks are automatically distributed across multiple storage Fault Domains (FDs). This distribution minimizes the risk of a single point of failure. 

:::image type="content" source="media/avset-main-figure.png" alt-text="Diagram Showing AvSet with Managed Disk FD alignment Setup." lightbox="media/avset-main-figure.png":::

Availability sets have fault isolation for many possible failures, to minimize single points of failure and to offer high availability.  If there is a failure in one storage Fault Domain (FD), only the VM instances with Premium SSD v2 disks on that specific Fault Domain are affected. The other VM instances, whose disks are placed on separate Fault Domains, remain unaffected and continue to operate normally. Availability sets are still susceptible to certain shared infrastructure failures, like datacenter network failures, physical hardware failures or power interruptions which can affect multiple fault domains. 

In a production scenario with three VMs deployed in an availability set using Premium SSD v2 with Fault Domain 3, VMs and disks are distributed across multiple fault domains. If one storage fault domain fails, only the VMs associated with that domain are affected, while the rest remain operational. The unaffected VMs will stay online, preserving the application's availability. 

If a Premium SSD v2 disk originally resides in one fault domain but is attached to a VM in another fault domain, the system initiates a background copy process. This operation moves the disk to align with the VM's fault domain, ensuring consistent compute and storage fault domain alignment for improved reliability and availability. 

:::image type="content" source="media/avset-disk-move-figure.png" alt-text="Diagram Showing AvSet with Managed Disk FD alignment Setup." lightbox="media/avset-disk-move-figure.png":::

For example, as shown in the diagram above, if a disk located in FD1 is attached to a VM in FD1 and is later detached and attached to a VM in FD2, the system will automatically trigger a background copy of the disk to move it from FD1 to FD2 for compute and storage fault domain alignment. This background move process can take up to 24 hours to complete.  

## Regional availability:

Premium SSD v2 support for VMs in an Availability Set is currently available only in following regions that do not support Availability Zones: 

- Australia Southeast
- Canada East
- North Central US
- UK West
- West Central US
- West US

## Limitations:

-  A subscription must be registered for the required feature to use Premium SSD v2 with VMs in Availability Sets in non-zonal regions. Follow the instructions [here](/https://github.com/MicrosoftDocs/azure-compute-docs-pr/edit/vishalprayag24-patch-1/articles/virtual-machines/use-premium-ssd-v2-with-availability-set.md#register-premium-ssd-v2-with-vms-in-availability-sets-in-supported-non-zonal-regions) to complete the registration.
- Only one background data copy per disk is allowed at a time. When attaching a disk to a VM in an Availability Set (AvSet), the system may initiate a background copy for Fault Domain (FD) alignment. If a detach-and-attach is attempted during an ongoing move, the operation fails with an error. To avoid this, wait for the move to complete, or set the [OptimizedForFrequentAttach](/dotnet/api/microsoft.azure.management.compute.models.diskupdate.optimizedforfrequentattach) disk property to skip FD-alignment background copies for future attachments. For more information on OptimizedForFrequentAttach, follow the instructions [here](/https://github.com/MicrosoftDocs/azure-compute-docs/edit/vishalprayag24-patch-1/articles/virtual-machines/use-premium-ssd-v2-with-availability-set.md#change-optimized-for-frequent-attach-disk-property).
- A disk created from a snapshot cannot be attached to VMs in an Availability Set while it is still undergoing background data copy from a snapshot. You must wait until the copy process is fully complete before attaching the disk. To check the status of background data copy from a snapshot, follow the instructions [here](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot).
- Disk size increase and changing customer-managed key (CMK) is not supported while a background data copy for Fault Domain alignment is in progress.



### [Azure CLI](#tab/CLI)
 
* Create a resource group:
 
```azurecli-interactive

az group create --name myResourceGroup --location myLocation 

```
 
The number of Storage Fault domains varies by region. The following command retrieves a list of fault domains per region:

```azurecli-interactive

az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location,  MaximumFaultDomainCount:capabilities[0].value}' -o Table 

```
 
* Create the availability set:  
 
```azurecli-interactive

az vm availability-set create -n myAvSet -g myResourceGroup --platform-fault-domain-count 3 --platform-update-domain-count 20

```
 
> [!Note]
> The value for *platform-fault-domain-count* should be determined based on the number of available Storage Fault Domains in a given region. Please follow Step b to determine the number of available Fault Domains per region.
 
* Create a VM:
 
```azurecli-interactive

az vm create -n myVMname -g myResourceGroupName --availability-set myAvSetName --image Win2016Datacenter --count MyCount 

```
 
* Attach a new premium SSD v2 disk to existing VMs in an availability set
 
```azurecli-interactive

az vm disk attach -g MyResourceGroupName --vm-name MyVMname --name MyDiskName --new --sku PremiumV2_LRS --size-gb MySize

```
 
* Attach existing Premium SSD v2 disk to existing VMs in an Availability Set:
 
```azurecli-interactive

az vm disk attach -g MyResourceGroupName --vm-name MyVMname --disks MyDiskName
 

### [PowerShell](#tab/PowerShell)
 
* Create a resource group:
 
```powershell
New-AzResourceGroup -Name myResourceGroup -Location myLocation
```
 
The number of Storage Fault domains varies by region. The following command retrieves a list of fault domains per region:
 
```powershell
Get*-AzComputeResourceSku | Where-Object {$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'} | Select-Object @{Name='Location'; Expression={$_.locationInfo[0].location}}, @{Name='MaximumFaultDomainCount'; Expression={$_.capabilities[0].value}}* 
```
* Create the availability set:  
 
> [!Note]
> The value for *platform-fault-domain-count* should be determined based on the number of available Storage Fault Domains in a given region. Please follow Step b to determine the number of available Fault Domains per region.
 
* Create a VM:
```powershell
New-AzVm 
ResourceGroupName $resourceGroupName 
Name $vmName 
Location $region  
Image $vmImage `
Size $vmSize 
AvailabilitySetName $AvSetName` 
Credential $credential
```
* Attach existing Premium SSD v2 disk to existing VMs in an Availability Set:
 
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

### [Portal](#tab/Portal)
 
* Sign in to the [Azure portal](https://portal.azure.com/).
 
* Create an Availability Set with the 'Use managed disk' option set to 'Yes (Aligned)'
 
:::image type="content" source="media/portal-step-two.png" alt-text="Diagram showing the Azure Portal screen to create an availability set." lightbox="media/portal-step-two.png":::
 
* Navigate to virtual machines and follow the normal VM creation process.
 
* On the **Basics** page, select a supported region and set availability options to Availability set.
 
* Select an Availability set
 
:::image type="content" source="media/portal-step-five.png" alt-text="Diagram showing the Azure Portal screen to select an availability set." lightbox="media/portal-step-five.png":::
 
* Fill in the rest of the values on the page as you like.
 
* Proceed to the Disks page.
 
* Under **Data disks** select **Create and attach a new disk**.
 
:::image type="content" source="media/portal-step-eight.png" alt-text="Diagram showing the Azure Portal screen to create and attach a new disk." lightbox="media/portal-step-eight.png":::
 
* Select the **Disk SKU** and select **Premium SSD v2**.
 
:::image type="content" source="media/portal-step-nine.png" alt-text="Diagram showing the Azure Portal screen to select disk SKU." lightbox="media/portal-step-nine.png":::
 
* Select logical sector of your choice to deploy a 4k or 512.
 
:::image type="content" source="media/portal-step-ten.png" alt-text="Diagram showing the Azure Portal screen to select logical sector." lightbox="media/portal-step-ten.png":::
 
* Proceed through the rest of the VM deployment, making any choices that you desire.
  
You've now deployed an Availability Set VM with a premium SSD v2.

## Register Premium SSD v2 with VMs in Availability Sets in supported Non-Zonal Regions:

To use Premium SSD v2 with VMs in Availability Sets in non-zonal regions, ensure your subscription is registered for the required feature:

- Command to register the feature:

  _az feature registration create --namespace Microsoft.Compute --name PV2WithAVSetRegionWithoutZone_

- Check registration status:

  _az feature registration show --provider Microsoft.Compute --name PV2WithAVSetRegionWithoutZone_

## Optimize Background Data Copy of the disk:
### Change optimized-for-frequent-attach disk property:
If the workload involves frequent disk detach and attach operations between VMs of the same Availability Set or between VMs in different Availability Sets, we recommend enabling the optimized-for-frequent-attach property. Setting this property to true prevents the system from triggering a background copy of the disk for Fault Domain alignment during reattachments. optimized-for-frequent-attach can be set when creating a new unattached disk or update it later for an existing disk. If the disk is currently attached to a VM, first detach the disk. Update the optimized-for-frequent-attach disk property and then reattach the disk to the VM. 

#### Instructions to Set or Update Optimized-for-Frequent-Attach on Disks:

##### Set the property while creating a new unattached disk:
az disk create --name myDiskName --resource-group myResourceGroup --location myLocation --sku PremiumV2_LRS --size-gb myGB --optimized-for-frequent-attach true 

#####  Update the property on an existing unattached disk:
az disk update --name myDiskName --resource-group myResourceGroup --set optimizedForFrequentAttach=true
