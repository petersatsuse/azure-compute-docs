---
title: Secure and use policies
description: Learn about security and policies for virtual machines in Azure.
author: ju-shim
ms.service: azure-virtual-machines
ms.subservice: security
ms.date: 02/26/2024
ms.author: jushiman
ms.topic: concept-article
# Customer intent: As a cloud administrator, I want to implement security policies for virtual machines, so that I can protect applications and data in the cloud while ensuring compliance with organizational security requirements.
---

# Secure and use policies on virtual machines in Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

It's important to keep your virtual machine (VM) secure for the applications that you run. Securing your VMs can include one or more Azure services and features that cover secure access to your VMs and secure storage of your data. This article provides information that enables you to keep your VM and applications secure.

## Antimalware

The modern threat landscape for cloud environments is dynamic, increasing the pressure to maintain effective protection in order to meet compliance and security requirements. [Microsoft Antimalware for Azure](/azure/security/fundamentals/antimalware) is a free real-time protection capability that helps identify and remove viruses, spyware, and other malicious software. Alerts can be configured to notify you when known malicious or unwanted software attempts to install itself or run on your VM. It is not supported on VMs running Linux or Windows Server 2008.

## Microsoft Defender for Cloud

[Microsoft Defender for Cloud](/azure/security-center/security-center-introduction) helps you prevent, detect, and respond to threats to your VMs. Defender for Cloud provides integrated security monitoring and policy management across your Azure subscriptions, helps detect threats that might otherwise go unnoticed, and works with a broad ecosystem of security solutions.

Defender for Cloud's just-in-time access can be applied across your VM deployment to lock down inbound traffic to your Azure VMs, reducing exposure to attacks while providing easy access to connect to VMs when needed. When just-in-time is enabled and a user requests access to a VM, Defender for Cloud checks what permissions the user has for the VM. If they have the correct permissions, the request is approved and Defender for Cloud automatically configures the Network Security Groups (NSGs) to allow inbound traffic to the selected ports for a limited amount of time. After the time has expired, Defender for Cloud restores the NSGs to their previous states. 

## Encryption

Two encryption methods are offered for managed disks. Encryption at the OS-level, which is Azure Disk Encryption, and encryption at the platform-level, which is server-side encryption.

### Server-side encryption

Azure managed disks automatically encrypt your data by default when persisting it to the cloud. Server-side encryption protects your data and helps you meet your organizational security and compliance commitments. Data in Azure managed disks is encrypted transparently using 256-bit [AES encryption](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), one of the strongest block ciphers available, and is FIPS 140-2 compliant.

Encryption does not impact the performance of managed disks. There is no additional cost for the encryption.

You can rely on platform-managed keys for the encryption of your managed disk, or you can manage encryption using your own keys. If you choose to manage encryption with your own keys, you can specify a *customer-managed key* to use for encrypting and decrypting all data in managed disks. 

To learn more about server-side encryption, refer to either the articles for [Windows](./disk-encryption.md) or [Linux](./disk-encryption.md).

### Azure Disk Encryption

For enhanced [Windows VM](windows/disk-encryption-overview.md) and [Linux VM](linux/disk-encryption-overview.md) security and compliance, virtual disks in Azure can be encrypted. Virtual disks on Windows VMs are encrypted at rest using BitLocker. Virtual disks on Linux VMs are encrypted at rest using dm-crypt. 

There is no charge for encrypting virtual disks in Azure. Cryptographic keys are stored in Azure Key Vault using software-protection, or you can import or generate your keys in Hardware Security Modules (HSMs) certified to [FIPS 140 validated](/azure/key-vault/keys/about-keys#compliance) standards. These cryptographic keys are used to encrypt and decrypt virtual disks attached to your VM. You retain control of these cryptographic keys and can audit their use. A Microsoft Entra service principal provides a secure mechanism for issuing these cryptographic keys as VMs are powered on and off.

## Key Vault and SSH Keys

Secrets and certificates can be modeled as resources and provided by [Key Vault](/azure/key-vault/general/basic-concepts). You can use Azure PowerShell to create key vaults for [Windows VMs](windows/key-vault-setup.md) and the Azure CLI for [Linux VMs](linux/key-vault-setup.md). You can also create keys for encryption.

Key vault access policies grant permissions to keys, secrets, and certificates separately. For example, you can give a user access to only keys, but no permissions for secrets. However, permissions to access keys or secrets or certificates are at the vault level. In other words, [key vault access policy](/azure/key-vault/general/security-features) does not support object level permissions.

When you connect to VMs, you should use public-key cryptography to provide a more secure way to sign in to them. This process involves a public and private key exchange using the secure shell (SSH) command to authenticate yourself rather than a username and password. Passwords are vulnerable to brute-force attacks, especially on Internet-facing VMs such as web servers. With a secure shell (SSH) key pair, you can create a [Linux VM](linux/mac-create-ssh-keys.md) that uses SSH keys for authentication, eliminating the need for passwords to sign-in. You can also use SSH keys to connect from a [Windows VM](linux/ssh-from-windows.md) to a Linux VM.

## Managed identities for Azure resources

A common challenge when building cloud applications is how to manage the credentials in your code for authenticating to cloud services. Keeping the credentials secure is an important task. Ideally, the credentials never appear on developer workstations and aren't checked into source control. Azure Key Vault provides a way to securely store credentials, secrets, and other keys, but your code has to authenticate to Key Vault to retrieve them. 

The managed identities for Azure resources feature in Microsoft Entra solves this problem. The feature provides Azure services with an automatically managed identity in Microsoft Entra ID. You can use the identity to authenticate to any service that supports Microsoft Entra authentication, including Key Vault, without any credentials in your code.  Your code that's running on a VM can request a token from two endpoints that are accessible only from within the VM. For more detailed information about this service, review the [managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) overview page.   

## Policies

[Azure policies](/azure/governance/policy/overview) can be used to define the desired behavior for your organization's VMs. By using policies, an organization can enforce various conventions and rules throughout the enterprise. Enforcement of the desired behavior can help mitigate risk while contributing to the success of the organization.

## Azure role-based access control

Using [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview), you can segregate duties within your team and grant only the amount of access to users on your VM that they need to perform their jobs. Instead of giving everybody unrestricted permissions on the VM, you can allow only certain actions. You can configure access control for the VM in the [Azure portal](/azure/role-based-access-control/role-assignments-portal), using the [Azure CLI](/cli/azure/role), or[Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell).


## Next steps
- Walk through the steps to monitor virtual machine security by using Microsoft Defender for Cloud for [Linux](/azure/security/fundamentals/overview) or [Windows](/previous-versions/azure/virtual-machines/tutorial-azure-security).
