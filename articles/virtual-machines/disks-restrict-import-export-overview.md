---
title: Restrict managed disks from being imported or exported
description: Restrict managed disks from being imported or exported
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/25/2024
ms.author: rogarana
# Customer intent: As an IT security administrator, I want to restrict the import and export of Azure managed disks, so that I can safeguard sensitive data and ensure compliance with organizational policies.
---

# Restrict managed disks from being imported or exported

This article provides an overview of your options for preventing your Azure managed disks from being imported or exported.

## Custom role

To limit the number of people who can import or export managed disks or snapshots using Azure RBAC, create a [custom RBAC role](/azure/role-based-access-control/custom-roles-powershell) that doesn't have the following permissions:

- Microsoft.Compute/disks/beginGetAccess/action
- Microsoft.Compute/disks/endGetAccess/action
- Microsoft.Compute/snapshots/beginGetAccess/action
- Microsoft.Compute/snapshots/endGetAccess/action

Any custom role without those permissions can't upload or download managed disks.

<a name='azure-ad-authentication'></a>

## Microsoft Entra authentication

If you're using Microsoft Entra ID to control resource access, you can also use it to restrict uploading of Azure managed disks. When a user attempts to upload a disk, Azure validates the identity of the requesting user in Microsoft Entra ID, and confirms that user has the required permissions. To learn more, see either the [PowerShell](windows/disks-upload-vhd-to-managed-disk-powershell.md#secure-uploads-with-azure-ad) or [CLI](linux/disks-upload-vhd-to-managed-disk-cli.md#secure-uploads-with-azure-ad) articles.

## Private links

You can use private endpoints to restrict the upload and download of managed disks and more securely access data over a private link from clients on your Azure virtual network. The private endpoint uses an IP address from the virtual network address space for your managed disks. Network traffic between clients on their virtual network and managed disks only traverses over the virtual network and a private link on the Microsoft backbone network, eliminating exposure from the public internet. To learn more, see either the [portal](disks-enable-private-links-for-import-export-portal.yml) or [CLI](linux/disks-export-import-private-links-cli.md) articles.

## Azure policy

[Configure an Azure Policy](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8426280e-b5be-43d9-979e-653d12a08638) to disable public network access to your managed disks.

## Configure the Network access policy

Each managed disk and snapshot has its own NetworkAccessPolicy parameter that can prevent the resource from being exported. You can use the [Azure CLI](/cli/azure/disk#az-disk-update) or [Azure PowerShell module](/powershell/module/az.compute/new-azdiskconfig) to set the parameter to **DenyAll**, which prevents the resource from being exported.
