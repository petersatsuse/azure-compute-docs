---
title: Azure Disk Encryption on an isolated network
description: In this article, learn about troubleshooting tips for Microsoft Azure Disk Encryption on Linux VMs.
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.custom: linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.author: mbaldwin
ms.date: 05/14/2025
# Customer intent: As a Linux system administrator managing isolated virtual machines, I want to troubleshoot Azure Disk Encryption installation issues, so that I can ensure proper package management and network configurations for successful encryption in a secure environment.
---
# Azure Disk Encryption on an isolated network

> [!CAUTION]
> This article references CentOS, a Linux distribution that is End Of Life (EOL) status. Consider your use and plan accordingly. For more information, see the [CentOS End Of Life guidance](~/articles/virtual-machines/workloads/centos/centos-end-of-life.md).

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets.

When connectivity is restricted by a firewall, proxy requirement, or network security group (NSG) settings, the ability of the extension to perform needed tasks might be disrupted. This disruption can result in status messages such as "Extension status not available on the VM."

## Package management

Azure Disk Encryption depends on many components, which are typically installed as part of ADE enablement if not already present. When behind a firewall or otherwise isolated from the Internet, these packages must be preinstalled or available locally.

Here are the packages necessary for each distribution. For a full list of supported distros and volume types, see [supported VMs and operating systems](disk-encryption-overview.md#supported-vms-and-operating-systems).

- **Ubuntu 14.04, 16.04, 18.04**: lsscsi, psmisc, at, cryptsetup-bin, python-parted, python-six, procps, grub-pc-bin
- **CentOS 7.2 - 7.9, 8.1, 8.2**: lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup, cryptsetup-reencrypt, pyparted, procps-ng, util-linux
- **CentOS 6.8**: lsscsi, psmisc, lvm2, uuid, at, cryptsetup-reencrypt, parted, python-six
- **RedHat 7.2 - 7.9, 8.1, 8.2**: lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup, cryptsetup-reencrypt, procps-ng, util-linux
- **RedHat 6.8**: lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup-reencrypt
- **openSUSE 42.3, SLES 12-SP4, 12-SP3**: lsscsi, cryptsetup

On Red Hat, when a proxy is required, you must make sure that the subscription-manager and yum are set up properly. For more information, see [How to troubleshoot subscription-manager and yum problems](https://access.redhat.com/solutions/189533).  

When packages are installed manually, they must also be manually upgraded as new versions are released.

## Network security groups
Any network security group settings that are applied must still allow the endpoint to meet the documented network configuration prerequisites for disk encryption. See [Azure Disk Encryption: Networking requirements](disk-encryption-overview.md#networking-requirements)

<a name='azure-disk-encryption-with-azure-ad-previous-version'></a>

## Azure Disk Encryption with Microsoft Entra ID (previous version)

If using [Azure Disk Encryption with Microsoft Entra ID (previous version)](disk-encryption-overview-aad.md), the [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) must be installed manually for all distros (in addition to the [packages appropriate for the distro](#package-management)).

When encryption is being enabled with [Microsoft Entra credentials](disk-encryption-linux-aad.md), the target VM must allow connectivity to both Microsoft Entra endpoints and Key Vault endpoints. Current Microsoft Entra authentication endpoints are maintained in sections 56 and 59 of the [Microsoft 365 URLs and IP address ranges](/microsoft-365/enterprise/urls-and-ip-address-ranges) documentation. Key Vault instructions are provided in the documentation on how to [Access Azure Key Vault behind a firewall](/azure/key-vault/general/access-behind-firewall).

### Azure Instance Metadata Service 

The virtual machine must be able to access the [Azure Instance Metadata service](instance-metadata-service.md) endpoint, which uses a well-known nonroutable IP address (`169.254.169.254`) that can be accessed only from within the VM. Proxy configurations that alter local HTTP traffic to this address (for example, adding an X-Forwarded-For header) aren't supported.

## Next steps

- See more steps for [Azure disk encryption troubleshooting](disk-encryption-troubleshooting.md)
- [Azure data encryption at rest](/azure/security/fundamentals/encryption-atrest)
