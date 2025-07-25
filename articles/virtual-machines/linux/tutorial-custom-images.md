---
title: Tutorial - Create custom VM images with the Azure CLI
description: In this tutorial, you learn how to use the Azure CLI to create a custom virtual machine image in Azure
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: tutorial
ms.date: 01/25/2023
ms.author: mattmcinnes
ms.custom: mvc, devx-track-azurecli, linux-related-content
ms.reviewer: cynthn
#Customer intent: As an IT administrator, I want to learn about how to create custom VM images to minimize the number of post-deployment configuration tasks.
# Customer intent: "As an IT administrator, I want to create custom VM images using the command line so that I can streamline deployments by preloading necessary configurations and applications."
---

# Tutorial: Create a custom image of an Azure VM with the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

Custom images are like marketplace images, but you create them yourself. Custom images can be used to bootstrap configurations such as preloading applications, application configurations, and other OS configurations. In this tutorial, you create your own custom image of an Azure virtual machine. You learn how to:

> [!div class="checklist"]
> * Create an Azure Compute Gallery (formerly known as Shared Image Gallery)
> * Create an image definition
> * Create an image version
> * Create a VM from an image 
> * Share a gallery


This tutorial uses the CLI within the [Azure Cloud Shell](/azure/cloud-shell/overview), which is constantly updated to the latest version. To open the Cloud Shell, select **Try it** from the top of any code block.

If you choose to install and use the CLI locally, this tutorial requires that you're running the Azure CLI version 2.35.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI]( /cli/azure/install-azure-cli).

## Overview

An [Azure Compute Gallery](../shared-image-galleries.md) simplifies custom image sharing across your organization. Custom images are like marketplace images, but you create them yourself. Custom images can be used to bootstrap configurations such as preloading applications, application configurations, and other OS configurations. 

The Azure Compute Gallery lets you share your custom VM images with others. Choose which images you want to share, which regions you want them to be available in, and who you want to share them with. 

The Azure Compute Gallery feature has multiple resource types:

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../includes/virtual-machines-shared-image-gallery-resources.md)]

## Before you begin

The following steps show how to take an existing VM and turn it into a reusable custom image that you can use to create new VM instances.

To complete the example in this tutorial, you must have an existing virtual machine. If needed, you can see the [CLI quickstart](quick-create-cli.md) to create a VM to use for this tutorial. When working through the tutorial, replace the resource names where needed.

## Launch Azure Cloud Shell

The Azure Cloud Shell is a free interactive shell that you can use to run the steps in this article. It has common Azure tools preinstalled and configured to use with your account. 

To open the Cloud Shell, just select **Try it** from the upper right corner of a code block. You can also launch Cloud Shell in a separate browser tab by going to [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Select **Copy** to copy the blocks of code, paste it into the Cloud Shell, and press enter to run it.

## Create a gallery 

A gallery is the primary resource used for enabling image sharing. 

Allowed characters for gallery name are uppercase or lowercase letters, digits, dots, and periods. The gallery name can't contain dashes. Gallery names must be unique within your subscription. 

Create a gallery using [az sig create](/cli/azure/sig#az-sig-create). The following example creates a resource group named gallery named *myGalleryRG* in *East US*, and a gallery named *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## Get information about the VM

You can see a list of VMs that are available using [az vm list](/cli/azure/vm#az-vm-list). 

```azurecli-interactive
az vm list --output table
```

Once you know the VM name and what resource group it is in, get the ID of the VM using [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view). 

```azurecli-interactive
az vm get-instance-view -g MyResourceGroup -n MyVm --query id
```

Copy the ID of your VM to use later.

## Create an image definition

Image definitions create a logical grouping for images. They're used to manage information about the image versions that are created within them. 

Image definition names can be made up of uppercase or lowercase letters, digits, dots, dashes, and periods. 

For more information about the values you can specify for an image definition, see [Image definitions](../shared-image-galleries.md#image-definitions).

Create an image definition in the gallery using [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create). 

In this example, the image definition is named *myImageDefinition*, and is for a [specialized](../shared-image-galleries.md#generalized-and-specialized-images) Linux OS image. 

```azurecli-interactive 
az sig image-definition create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```

Copy the ID of the image definition from the output to use later.

## Create the image version

Create an image version from the VM using [az sig image-version create](/cli/azure/sig/image-version#az-sig-image-version-create).  

Allowed characters for image version are numbers and periods. Numbers must be within the range of a 32-bit integer. Format: *MajorVersion*.*MinorVersion*.*Patch*.

In this example, the version of our image is *1.0.0* and we're going to create two replicas in the *West Central US* region, one replica in the *South Central US* region and one replica in the *East US 2* region using zone-redundant storage. The replication regions must include the region the source VM is located.

Replace the value of `--virtual-machine` in this example with the ID of your VM from the previous step.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "westcentralus" "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --virtual-machine "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> You need to wait for the image version to completely finish being built and replicated before you can use the same managed image to create another image version.
>
> You can also store your image in Premium storage by a adding `--storage-account-type  premium_lrs`, or [Zone Redundant Storage](/azure/storage/common/storage-redundancy) by adding `--storage-account-type  standard_zrs` when you create the image version.
>

 
## Create the VM

Create the VM using [az vm create](/cli/azure/vm#az-vm-create) using the `--specialized` parameter to indicate the image is a specialized image.

Use the image definition ID for `--image` to create the VM from the latest version of the image that is available. You can also create the VM from a specific version by supplying the image version ID for `--image`. 

In this example, we're creating a VM from the latest version of the *myImageDefinition* image.

```azurecli
az group create --name myResourceGroup --location eastus
az vm create --resource-group myResourceGroup \
    --name myVM2 \
    --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
    --specialized
```

## Share the gallery

You can share images across subscriptions using Azure role-based access control (Azure RBAC). You can share images at the gallery, image definition or image version level. Any user that has read permissions to an image version, even across subscriptions, will be able to deploy a VM using the image version.

We recommend that you share with other users at the gallery level. To get the object ID of your gallery, use [az sig show](/cli/azure/sig#az-sig-show).

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Use the object ID as a scope, along with an email address and [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) to give a user access to the Azure Compute Gallery. Replace `<email-address>` and `<gallery iD>` with your own information.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

For more information about how to share resources using Azure RBAC, see [Add or remove Azure role assignments using Azure CLI](/azure/role-based-access-control/role-assignments-cli).

## Azure Image Builder

Azure also offers a service, built on Packer, [Azure VM Image Builder](../image-builder-overview.md). Describe your customizations in a template, and it will handle the image creation. 

## Next steps

In this tutorial, you created a custom VM image. You learned how to:

> [!div class="checklist"]
> * Create an Azure Compute Gallery
> * Create an image definition
> * Create an image version
> * Create a VM from an image 
> * Share a gallery

Advance to the next tutorial to learn about Virtual Machine Scale Sets.

> [!div class="nextstepaction"]
> [Create a virtual machine scale set](tutorial-create-vmss.md)
