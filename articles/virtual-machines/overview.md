---
title: Overview of virtual machines in Azure
description: Overview of virtual machines in Azure.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.collection: linux
ms.topic: overview
ms.date: 03/27/2025
ms.author: mattmcinnes
# Customer intent: "As a cloud architect, I want to evaluate the features and management options of Azure virtual machines, so that I can determine the best deployment strategy for my applications and optimize costs and performance."
---

# Virtual machines in Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

Azure virtual machines (VMs) are one of several types of [on-demand, scalable computing resources](/azure/architecture/guide/technology-choices/compute-decision-tree) that Azure offers. Typically, you choose a virtual machine when you need more control over the computing environment than the other choices offer. This article gives you information about what you should consider before you create a virtual machine, how you create it, and how you manage it.

An Azure virtual machine gives you the flexibility of virtualization without having to buy and maintain the physical hardware that runs it. However, you still need to maintain the virtual machine by performing tasks, such as configuring, patching, and installing the software that runs on it.

Azure virtual machines can be used in various ways. Some examples are:

* **Development and test** – Azure virtual machines offer a quick and easy way to create a computer with specific configurations required to code and test an application.
* **Applications in the cloud** – Because demand for your application can fluctuate, it might make economic sense to run it on a virtual machine in Azure. You pay for extra virtual machines when you need them and shut them down when you don’t.
* **Extended datacenter** – virtual machines in an Azure virtual network can easily be connected to your organization’s network.

## What do I need to think about before creating a virtual machine?
There's always a multitude of [design considerations](/azure/architecture/reference-architectures/n-tier/linux-vm) when you build out an application infrastructure in Azure. These aspects of a virtual machine are important to think about before you start:

* The names of your resources
* The location where the resources are stored
* The size of the virtual machine
* The maximum number of virtual machines that can be created
* The operating system that the virtual machine runs
* The configuration of the virtual machine after it starts
* The related resources that the virtual machine needs

> [!NOTE]
>
> Trusted Launch as default (TLaD) is available in preview for new [Generation 2](../virtual-machines/generation-2.md) Virtual machines (VMs). With TLaD, any new Generation 2 VMs created through any client tools defaults to [Trusted Launch](../virtual-machines/trusted-launch.md) with secure boot and vTPM enabled.
> [Register for the TLaD preview](../virtual-machines/trusted-launch.md#preview-trusted-launch-as-default) to validate the default changes in your respective environment and prepare for the upcoming change.

## Parts of a VM and how they're billed

When you create a virtual machine, you're also creating resources that support the virtual machine. These resources come with their own costs that should be considered.

The default resources supporting a virtual machine and how they're billed are detailed in the following table:

| Resource | Description | Cost | 
|-|-|-|
| Virtual network | For giving your virtual machine the ability to communicate with other resources | [Virtual Network pricing](https://azure.microsoft.com/pricing/details/virtual-network/) |
| A virtual Network Interface Card (NIC) | For connecting to the virtual network  | There's no separate cost for NICs. However, there's a limit to how many NICs you can use based on your [VM's size](sizes.md). Size your VM accordingly and reference [Virtual Machine pricing](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). | 
| A private IP address and sometimes a public IP address. | For communication and data exchange on your network and with external networks | [IP Addresses pricing](https://azure.microsoft.com/pricing/details/ip-addresses/) |
| Network security group (NSG) | For managing the network traffic to and from your VM. For example, you might need to open port 22 for SSH access, but you might want to block traffic to port 80. Blocking and allowing port access is done through the NSG.| There are no additional charges for network security groups in Azure. |
| OS Disk and possibly separate disks for data. | It's a best practice to keep your data on a separate disk from your operating system, in case you ever have a VM fail, you can detach the data disk, and attach it to a new VM. | All new virtual machines have an operating system disk and a local disk. <br> Azure doesn't charge for local disk storage. <br> The operating system disk, which is usually 127GiB but is smaller for some images, is charged at the [regular rate for disks](https://azure.microsoft.com/pricing/details/managed-disks/). <br> You can see the cost for attach Premium (SSD based) and Standard (HDD) based disks to your virtual machines on the [Managed Disks pricing page](https://azure.microsoft.com/pricing/details/managed-disks/). |
| In some cases, a license for the OS | For providing your virtual machine runs to run the OS | The cost varies based on the number of cores on your VM, so [size your VM accordingly](sizes.md). The cost can be reduced through the [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/#overview). |

You can also choose to have Azure can create and store public and private SSH keys - Azure uses the public key in your VM and you use the private key when you access the VM over SSH. Otherwise, you need a username and password.

By default, these resources are created in the same resource group as the VM. 

### Locations
There are multiple [geographical regions](https://azure.microsoft.com/regions/) around the world where you can create Azure resources. Usually, the region is called **location** when you create a virtual machine. For a virtual machine, the location specifies where the virtual hard disks are stored.

This table shows some of the ways you can get a list of available locations.

| Method | Description |
| --- | --- |
| Azure portal |Select a location from the list when you create a virtual machine. |
| Azure PowerShell |Use the [Get-AzLocation](/powershell/module/az.resources/get-azlocation) command. |
| REST API |Use the [List locations](/rest/api/resources/subscriptions) operation. |
| Azure CLI |Use the [az account list-locations](/cli/azure/account) operation. |

## Availability
There are multiple options to manage the availability of your virtual machines in Azure. 
- **[Availability Zones](/azure/reliability/availability-zones-overview)** are physically separated zones within an Azure region. Availability zones guarantee virtual machine connectivity to at least one instance at least 99.99% of the time when you have two or more instances deployed across two or more Availability Zones in the same Azure region. 
- **[Virtual Machine Scale Sets](../virtual-machine-scale-sets/overview.md)** let you create and manage a group of load balanced virtual machines. The number of virtual machine instances can automatically increase or decrease in response to demand or a defined schedule. Scale sets provide high availability to your applications, and allow you to centrally manage, configure, and update many virtual machines. Virtual machines in a scale set can also be deployed into multiple availability zones, a single availability zone, or regionally.

Fore more information see [Availability options for Azure virtual machines](availability.md) and [SLA for Azure virtual machines](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9/). 

## Sizes and pricing
The [size](sizes.md) of the virtual machine that you use is determined by the workload that you want to run. The size that you choose then determines factors such as processing power, memory, storage capacity, and network bandwidth. Azure offers a wide variety of sizes to support many types of uses.

Azure charges an [hourly price](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) based on the virtual machine’s size and operating system. For partial hours, Azure charges only for the minutes used. Storage is priced and charged separately.

## Scaling to multiple VMs
The number of virtual machines that your application uses can scale up and out to whatever is required to meet your needs. Azure offers several automatic scaling systems and instance types based on the size of your workload. For more information on scaling, [compare VM-based compute products.](./compare-compute-products.md)

## Virtual machine total core limits
Your subscription has default [quota limits](/azure/azure-resource-manager/management/azure-subscription-service-limits) in place that could impact the deployment of many virtual machines for your project. The current limit on a per subscription basis is 20 virtual machine total cores per region. Limits can be raised by [filing a support ticket requesting an increase](/azure/azure-portal/supportability/regional-quota-requests)

## Managed Disks

Managed Disks handles Azure Storage account creation and management in the background for you, and ensures that you don't have to worry about the scalability limits of the storage account. You specify the disk size and the performance tier (Standard or Premium), and Azure creates and manages the disk. As you add disks or scale the virtual machine up and down, you don't have to worry about the storage being used. If you're creating new virtual machines, [use the Azure CLI](linux/quick-create-cli.md) or the Azure portal to create virtual machines with Managed OS and data disks. If you have virtual machines with unmanaged disks, you can [convert your virtual machines to be backed with Managed Disks](linux/convert-unmanaged-to-managed-disks.md).

You can also manage your custom images in one storage account per Azure region, and use them to create hundreds of virtual machines in the same subscription. For more information about Managed Disks, see the [Managed Disks Overview](managed-disks-overview.md).

## Distributions 
Microsoft Azure supports various Linux and Windows distributions. You can find available distributions in the [marketplace](https://azuremarketplace.microsoft.com), Azure portal or by querying results using CLI, PowerShell, and REST APIs. 

This table shows some ways that you can find the information for an image.

| Method | Description |
| --- | --- |
| Azure portal |The values are automatically specified for you when you select an image to use. |
| Azure PowerShell |[Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) -Location *location*<BR>[Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) -Location *location* -Publisher *publisherName*<BR>[Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) -Location *location* -Publisher *publisherName* -Offer *offerName* |
| REST APIs |[List image publishers](/rest/api/compute/platformimages/platformimages-list-publishers)<BR>[List image offers](/rest/api/compute/platformimages/platformimages-list-publisher-offers)<BR>[List image skus](/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus) |
| Azure CLI |[az vm image list-publishers](/cli/azure/vm/image) --location *location*<BR>[az vm image list-offers](/cli/azure/vm/image) --location *location* --publisher *publisherName*<BR>[az vm image list-skus](/cli/azure/vm) --location *location* --publisher *publisherName* --offer *offerName*|

Microsoft works closely with partners to ensure the images available are updated and optimized for an Azure runtime. For more information on Azure partner offers, see the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?filters=partners%3Bvirtual-machine-images&page=1)


## Cloud-init 

Azure supports for [cloud-init](https://cloud-init.io/) across most Linux distributions that support it. We're actively working with our Linux partners to make cloud-init enabled images available in the Azure Marketplace. These images make your cloud-init deployments and configurations work seamlessly with virtual machines and virtual machine scale sets.

For more information, see [Using cloud-init on Azure Linux virtual machines](linux/using-cloud-init.md).

## Storage
* [Introduction to Microsoft Azure Storage](/azure/storage/common/storage-introduction)
* [Add a disk to a Linux virtual machine using the azure-cli](linux/add-disk.md)
* [How to attach a data disk to a Linux virtual machine in the Azure portal](linux/attach-disk-portal.yml)

## Networking
* [Virtual Network Overview](/azure/virtual-network/virtual-networks-overview)
* [IP addresses in Azure](/azure/virtual-network/ip-services/public-ip-addresses)
* [Opening ports to a Linux virtual machine in Azure](linux/nsg-quickstart.md)
* [Create a Fully Qualified Domain Name in the Azure portal](create-fqdn.md)

## Service disruptions

At Microsoft, we work hard to make sure that our services are always available to you when you need them. Forces beyond our control sometimes impact us in ways that cause unplanned service disruptions.

Microsoft provides a Service Level Agreement (SLA) for its services as a commitment for uptime and connectivity. The SLA for individual Azure services can be found at [Azure Service Level Agreements](https://azure.microsoft.com/support/legal/sla/).

Azure already has many built-in platform features that support highly available applications. For more about these services, read [Disaster recovery and high availability for Azure applications](/azure/architecture/framework/resiliency/backup-and-recovery).

This article covers a true disaster recovery scenario, when a whole region experiences an outage due to major natural disaster or widespread service interruption. These are rare occurrences, but you must prepare for the possibility that there's an outage of an entire region. If an entire region experiences a service disruption, the locally redundant copies of your data would temporarily be unavailable. If you enabled geo-replication, three additional copies of your Azure Storage blobs and tables are stored in a different region. In the event of a complete regional outage or a disaster in which the primary region isn't recoverable, Azure remaps all of the DNS entries to the geo-replicated region.

In the case of a service disruption of the entire region where your Azure virtual machine application is deployed, we provide the following guidance for Azure virtual machines.

### Option 1: Initiate a failover by using Azure Site Recovery
You can configure Azure Site Recovery for your VMs so that you can recover your application with a single click in matter of minutes. You can replicate to Azure region of your choice and not restricted to paired regions. You can get started by [replicating your virtual machines](/azure/site-recovery/azure-to-azure-quickstart). You can [create a recovery plan](/azure/site-recovery/site-recovery-create-recovery-plans) so that you can automate the entire failover process for your application. You can [test your failovers](/azure/site-recovery/site-recovery-test-failover-to-azure) beforehand without impacting production application or the ongoing replication. In the event of a primary region disruption, you just [initiate a failover](/azure/site-recovery/site-recovery-failover) and bring your application in target region.


### Option 2: Wait for recovery
In this case, no action on your part is required. Know that we're working diligently to restore service availability. You can see the current service status on our [Azure Service Health Dashboard](https://azure.microsoft.com/status/).

This option is the best if you don't set up Azure Site Recovery, read-access geo-redundant storage, or geo-redundant storage prior to the disruption. If you set up geo-redundant storage or read-access geo-redundant storage for the storage account where your VM virtual hard drives (VHDs) are stored, you can look to recover the base image VHD and try to provision a new VM from it. This option isn't preferred because there are no guarantees of synchronization of data, which means this option isn't guaranteed to work.


> [!NOTE]
> Be aware that you don't have any control over this process, and it will only occur for region-wide service disruptions. Because of this, you must also rely on other application-specific backup strategies to achieve the highest level of availability. For more information, see the section on [Data strategies for disaster recovery](/azure/architecture/reliability/disaster-recovery#disaster-recovery-plan).
>
>

### Resources for service disruptions

- Start [protecting your applications running on Azure virtual machines](/azure/site-recovery/azure-to-azure-quickstart) using Azure Site Recovery

- To learn more about how to implement a disaster recovery and high availability strategy, see [Disaster recovery and high availability for Azure applications](/azure/architecture/framework/resiliency/backup-and-recovery).

- To develop a detailed technical understanding of a cloud platform’s capabilities, see [Azure resiliency technical guidance](/azure/data-lake-store/data-lake-store-disaster-recovery-guidance).


- If the instructions aren't clear, or if you would like Microsoft to do the operations on your behalf, contact [Customer Support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## Data residency

In Azure, the feature to enable storing customer data in a single region is currently only available in the Southeast Asia Region (Singapore) of the Asia Pacific Geo and Brazil South (Sao Paulo State) Region of Brazil Geo. For all other regions, customer data is stored in Geo. For more information, see [Trust Center](https://azure.microsoft.com/global-infrastructure/data-residency/).


## Next steps

Create your first virtual machine!

- [Portal](linux/quick-create-portal.md)
- [Azure CLI](linux/quick-create-cli.md)
- [PowerShell](linux/quick-create-powershell.md)
