---
title: Create SSH keys with the Azure CLI
description: Learn how to generate and store SSH keys, before creating a VM, with the Azure CLI for connecting to Linux VMs.
author: ju-shim
ms.collection: linux
ms.service: azure-virtual-machines
ms.custom: devx-track-azurecli, linux-related-content
ms.topic: how-to
ms.date: 04/13/2023
ms.author: jushiman
# Customer intent: "As a cloud administrator, I want to generate and manage SSH keys using a command-line interface, so that I can securely connect to and manage Linux virtual machines in Azure."
---

# Generate and store SSH keys with the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

You can create SSH keys before creating a VM and store them in Azure. Each newly created SSH key is also stored locally.

If you have existing SSH keys, you can upload and store them in Azure for reuse.

For more information, see [Detailed steps: Create and manage SSH keys for authentication to a Linux VM in Azure](./linux/create-ssh-keys-detailed.md).

For more information on how to create and use SSH keys with Linux VMs, see [Use SSH keys to connect to Linux VMs](./linux/ssh-from-windows.md).

## Generate new keys

1. After you sign in, use the [az sshkey create](/cli/azure/sshkey#az-sshkey-create) command to create the new SSH key:

    ```azurecli
    az sshkey create --name "mySSHKey" --resource-group "myResourceGroup"
   ```

    > [!NOTE]
    > This command would default to key type of RSA, in order to generate ED25519 keys you can pass in the optional flag `--encryption-type Ed25519`.


1. The resulting output lists the new key files' paths:

    ```azurecli
    Private key is saved to "/home/user/.ssh/7777777777_9999999".
    Public key is saved to "/home/user/.ssh/7777777777_9999999.pub".
   ```

1. Change the permissions for the private key file for privacy:

    ```azurecli
    chmod 600 /home/user/.ssh/7777777777_9999999
    ```

## Connect to the VM

On your local computer, open a Bash prompt:

```azurecli
ssh -i <path to the private key file> username@<ipaddress of the VM>
```

For example, enter: `ssh -i /home/user/.ssh/mySSHKey azureuser@123.45.67.890`

## Upload an SSH key

You can upload a public SSH key to store in Azure.

Use the [az sshkey create](/cli/azure/sshkey#az-sshkey-create) command to upload an SSH public key by specifying its file:

```azurecli
az sshkey create --name "mySSHKey" --public-key "@/home/user/.ssh/7777777777_9999999.pub" --resource-group "myResourceGroup"
```

## List keys

Use the [az sshkey list](/cli/azure/sshkey#az-sshkey-list) command to list all public SSH keys, optionally specifying a resource group:

```azurecli
az sshkey list --resource-group "myResourceGroup"
```

## Get the public key

Use the [az sshkey show](/cli/azure/sshkey#az-sshkey-show) command to show the values of a public SSH key:

```azurecli
az sshkey show --name "mySSHKey" --resource-group "myResourceGroup"
```

## Next steps

To learn more about how to use SSH keys with Azure VMs, see [Use SSH keys to connect to Linux VMs](./linux/ssh-from-windows.md).
