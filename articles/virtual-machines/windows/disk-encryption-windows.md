---
title: Azure Disk Encryption scenarios on Windows VMs
description: This article provides instructions on enabling Microsoft Azure Disk Encryption for Windows VMs for various scenarios
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.collection: windows
ms.topic: how-to
ms.author: mbaldwin
ms.date: 03/03/2025
ms.custom: devx-track-azurepowershell, devx-track-azurecli
# Customer intent: As a cloud administrator, I want to enable Azure Disk Encryption on Windows VMs, so that I can secure my virtual machine data using BitLocker and manage encryption keys effectively with Azure Key Vault.
---

# Azure Disk Encryption scenarios on Windows VMs

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

Azure Disk Encryption for Windows virtual machines (VMs) uses the BitLocker feature of Windows to provide full disk encryption of the OS disk and data disk. Additionally, it provides encryption of the temporary disk when the VolumeType parameter is All.

Azure Disk Encryption is [integrated with Azure Key Vault](disk-encryption-key-vault.yml) to help you control and manage the disk encryption keys and secrets. For an overview of the service, see [Azure Disk Encryption for Windows VMs](disk-encryption-overview.md).

## Prerequisites

You can only apply disk encryption to virtual machines of [supported VM sizes and operating systems](disk-encryption-overview.md#supported-vms-and-operating-systems). You must also meet the following prerequisites:

- [Networking requirements](disk-encryption-overview.md#networking-requirements)
- [Group Policy requirements](disk-encryption-overview.md#group-policy-requirements)
- [Encryption key storage requirements](disk-encryption-overview.md#encryption-key-storage-requirements)

## Restrictions

If you have previously used Azure Disk Encryption with Microsoft Entra ID to encrypt a VM, you must continue use this option to encrypt your VM. See [Azure Disk Encryption with Microsoft Entra ID (previous release)](disk-encryption-overview-aad.md) for details.

You should [take a snapshot](snapshot-copy-managed-disk.md) and/or create a backup before disks are encrypted. Backups ensure that a recovery option is possible if an unexpected failure occurs during encryption. VMs with managed disks require a backup before encryption occurs. Once a backup is made, you can use the [Set-AzVMDiskEncryptionExtension cmdlet](/powershell/module/az.compute/set-azvmdiskencryptionextension) to encrypt managed disks by specifying the -skipVmBackup parameter. For more information about how to back up and restore encrypted VMs, see [Back up and restore encrypted Azure VM](/azure/backup/backup-azure-vms-encryption).

Encrypting or disabling encryption may cause a VM to reboot.

Azure Disk Encryption does not work for the following scenarios, features, and technology:

- Encrypting basic tier VM or VMs created through the classic VM creation method.
- Encrypting v6 series VMs. For more information, see the individual pages for each of these VM sizes listed on [Sizes for virtual machines in Azure](../sizes/overview.md)
- All requirements and restrictions of BitLocker, such as requiring NTFS. For more information, see [BitLocker overview](/windows/security/operating-system-security/data-protection/bitlocker/#system-requirements).
- Encrypting VMs configured with software-based RAID systems.
- Encrypting VMs configured with Storage Spaces Direct (S2D), or Windows Server versions before 2016 configured with Windows Storage Spaces.
- Integration with an on-premises key management system.
- Azure Files (shared file system).
- Network File System (NFS).
- Dynamic volumes.
- Windows Server containers, which create dynamic volumes for each container.
- Ephemeral OS disks.
- iSCSI disks.
- Encryption of shared/distributed file systems like (but not limited to) DFS, GFS, DRDB, and CephFS.
- Moving an encrypted VM to another subscription or region.
- Creating an image or snapshot of an encrypted VM and using it to deploy additional VMs.
- M-series VMs with Write Accelerator disks.
- Applying ADE to a VM that has disks encrypted with [Encryption at Host](../disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data) or [server-side encryption with customer-managed keys](../disk-encryption.md) (SSE + CMK). Applying SSE + CMK to a data disk or adding a data disk with SSE + CMK configured to a VM encrypted with ADE is an unsupported scenario as well.
- Migrating a VM that is encrypted with ADE, or has **ever** been encrypted with ADE, to [Encryption at Host](../disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data) or [server-side encryption with customer-managed keys](../disk-encryption.md).
- Encrypting VMs in failover clusters.
- Encryption of [Azure ultra disks](../disks-enable-ultra-ssd.md).
- Encryption of [Premium SSD v2 disks](../disks-types.md#premium-ssd-v2-limitations).
- Encryption of VMs in subscriptions that have the [`Secrets should have the specified maximum validity period`](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F342e8053-e12e-4c44-be01-c3c2f318400f) policy enabled with the [DENY effect](/azure/governance/policy/concepts/effects).
- Encryption of VMs in subscriptions that have the [`Key Vault secrets should have an expiration date`](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F98728c90-32c7-4049-8429-847dc0f4fe37) policy enabled with the [DENY effect](/azure/governance/policy/concepts/effects)

## Install tools and connect to Azure

[!INCLUDE [disk-encryption-install-cli-powershell](../includes/disk-encryption-install-cli-powershell.md)]

## Enable encryption on an existing or running Windows VM
In this scenario, you can enable encryption by using the Resource Manager template, PowerShell cmdlets, or CLI commands. If you need schema information for the virtual machine extension, see the [Azure Disk Encryption for Windows extension](../extensions/azure-disk-enc-windows.md) article.

### Enable encryption on existing or running VMs with Azure PowerShell
Use the [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) cmdlet to enable encryption on a running IaaS virtual machine in Azure.

-  **Encrypt a running VM:** The script below initializes your variables and runs the Set-AzVMDiskEncryptionExtension cmdlet. The resource group, VM, and key vault should have already been created as prerequisites. Replace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, and MySecureVault with your values.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Encrypt a running VM using KEK:**

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```

   >[!NOTE]
   > The syntax for the value of disk-encryption-keyvault parameter is the full identifier string:
/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br>
   > The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in:
https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]

- **Verify the disks are encrypted:** To check on the encryption status of an IaaS VM, use the [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) cmdlet.
     ```azurepowershell-interactive
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

To disable the encryption, see [Disable encryption and remove the encryption extension](#disable-encryption-and-remove-the-encryption-extension).

### Enable encryption on existing or running VMs with the Azure CLI

Use the [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) command to enable encryption on a running IaaS virtual machine in Azure.

- **Encrypt a running VM:**

    ```azurecli-interactive
    az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
    ```

- **Encrypt a running VM using KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > The syntax for the value of disk-encryption-keyvault parameter is the full identifier string:
  /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] </br>
     > The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in:
  https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]

- **Verify the disks are encrypted:** To check on the encryption status of an IaaS VM, use the [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) command.

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

To disable the encryption, see [Disable encryption and remove the encryption extension](#disable-encryption-and-remove-the-encryption-extension).

### Using the Resource Manager template

You can enable disk encryption on existing or running IaaS Windows VMs in Azure by using the [Resource Manager template to encrypt a running Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/encrypt-running-windows-vm-without-aad).


1. On the Azure quickstart template, click **Deploy to Azure**.

2. Select the subscription, resource group, location, settings, legal terms, and agreement. Click **Purchase** to enable encryption on the existing or running IaaS VM.

The following table lists the Resource Manager template parameters for existing or running VMs:

| Parameter | Description |
| --- | --- |
| vmName | Name of the VM to run the encryption operation. |
| keyVaultName | Name of the key vault that the BitLocker key should be uploaded to. You can get it by using the cmdlet `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` or the Azure CLI command `az keyvault list --resource-group "MyKeyVaultResourceGroup"`|
| keyVaultResourceGroup | Name of the resource group that contains the key vault|
|  keyEncryptionKeyURL | The URL of the key encryption key, in the format https://&lt;keyvault-name&gt;.vault.azure.net/key/&lt;key-name&gt;. If you do not wish to use a KEK, leave this field blank. |
| volumeType | Type of volume that the encryption operation is performed on. Valid values are _OS_, _Data_, and _All_.
| forceUpdateTag | Pass in a unique value like a GUID every time the operation needs to be force run. |
| resizeOSDisk | Should the OS partition be resized to occupy full OS VHD before splitting system volume. |
| location | Location for all resources. |

## Enable encryption on NVMe disks for Lsv2 VMs

This scenario describes enabling Azure Disk Encryption on NVMe disks for Lsv2 series VMs.  The Lsv2-series features local NVMe storage. Local NVMe Disks are temporary, and data will be lost on these disks if you stop/deallocate your VM (See: [Lsv2-series](../lsv2-series.md)).

To enable encryption on NVMe disks:

1. Initialize the NVMe disks and create NTFS volumes.
1. Enable encryption on the VM with the VolumeType parameter set to All. This will enable encryption for all OS and data disks, including volumes backed by NVMe disks. For information, see [Enable encryption on an existing or running Windows VM](#enable-encryption-on-an-existing-or-running-windows-vm).

Encryption will persist on the NVMe disks in the following scenarios:
- VM reboot
- Virtual machine scale set reimage
- Swap OS

NVMe disks will be uninitialized the following scenarios:

- Start VM after deallocation
- Service healing
- Backup

In these scenarios, the NVMe disks need to be initialized after the VM starts. To enable encryption on the NVMe disks, run command to enable Azure Disk Encryption again after the NVMe disks are initialized.

In addition to the scenarios listed in the [Restrictions](#restrictions) section, encryption of NVMe disks is not supported for:

- VMs encrypted with Azure Disk Encryption with Microsoft Entra ID (previous release)
- NVMe disks with storage spaces
- Azure Site Recovery of SKUs with NVMe disks (see [Support matrix for Azure VM disaster recovery between Azure regions: Replicated machines - storage](/azure/site-recovery/azure-to-azure-support-matrix#replicated-machines---storage)).

## New IaaS VMs created from customer-encrypted VHD and encryption keys

In this scenario, you can create a new VM from a pre-encrypted VHD and the associated encryption keys using PowerShell cmdlets or CLI commands.

Use the instructions in [Prepare a pre-encrypted Windows VHD](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-windows-vhd). After the image is created, you can use the steps in the next section to create an encrypted Azure VM.


### Encrypt VMs with pre-encrypted VHDs with Azure PowerShell
You can enable disk encryption on your encrypted VHD by using the PowerShell cmdlet [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples). The example below gives you some common parameters.

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myKVresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## Enable encryption on a newly added data disk
You can [add a new disk to a Windows VM using PowerShell](attach-disk-ps.md), or [through the Azure portal](attach-managed-disk-portal.yml).

 >[!NOTE]
 > Newly added data disk encryption must be enabled via Powershell, or CLI only. Currently, the Azure portal does not support enabling encryption on new disks.

### Enable encryption on a newly added disk with Azure PowerShell
 When using PowerShell to encrypt a new disk for Windows VMs, a new sequence version should be specified. The sequence version has to be unique. The script below generates a GUID for the sequence version. In some cases, a newly added data disk might be encrypted automatically by the Azure Disk Encryption extension. Auto encryption usually occurs when the VM reboots after the new disk comes online. This is typically caused because "All" was specified for the volume type when disk encryption previously ran on the VM. If auto encryption occurs on a newly added data disk, we recommend running the Set-AzVmDiskEncryptionExtension cmdlet again with new sequence version. If your new data disk is auto encrypted and you do not wish to be encrypted, decrypt all drives first then re-encrypt with a new sequence version specifying OS for the volume type.



-  **Encrypt a running VM:** The script below initializes your variables and runs the Set-AzVMDiskEncryptionExtension cmdlet. The resource group, VM, and key vault should have already been created as prerequisites. Replace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, and MySecureVault with your values. This example uses "All" for the -VolumeType parameter, which includes both OS and Data volumes. If you only want to encrypt the OS volume, use "OS" for the -VolumeType parameter.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;
    ```
- **Encrypt a running VM using KEK:** This example uses "All" for the -VolumeType parameter, which includes both OS and Data volumes. If you only want to encrypt the OS volume, use "OS" for the -VolumeType parameter.

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
     $sequenceVersion = [Guid]::NewGuid();

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > The syntax for the value of disk-encryption-keyvault parameter is the full identifier string:
/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br>
   > The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in:
https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]

### Enable encryption on a newly added disk with Azure CLI

 The Azure CLI command will automatically provide a new sequence version for you when you run the command to enable encryption. The example uses "All" for the volume-type parameter. You may need to change the volume-type parameter to OS if you're only encrypting the OS disk. In contrast to PowerShell syntax, the CLI does not require the user to provide a unique sequence version when enabling encryption. The CLI automatically generates and uses its own unique sequence version value.

-  **Encrypt a running VM:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Encrypt a running VM using KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "All"
     ```

## Disable encryption and remove the encryption extension

You can disable the Azure disk encryption extension, and you can remove the Azure disk encryption extension. These are two distinct operations. 

To remove ADE, it is recommended that you first disable encryption and then remove the extension. If you remove the encryption extension without disabling it, the disks will still be encrypted. If you disable encryption **after** removing the extension, the extension will be reinstalled (to perform the decrypt operation) and will need to be removed a second time.

### Disable encryption

You can disable encryption using Azure PowerShell, the Azure CLI, or with a Resource Manager template. Disabling encryption does **not** remove the extension (see [Remove the encryption extension](#remove-the-encryption-extension)).

> [!WARNING]
> Disabling data disk encryption when both the OS and data disks have been encrypted can have unexpected results. Disable encryption on all disks instead.
>
> Disabling encryption will start a background process of BitLocker to decrypt the disks. This process should be given sufficient time to complete before attempting to any re-enable encryption.  

- **Disable disk encryption with Azure PowerShell:** To disable the encryption, use the [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption) cmdlet.

     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName "MyVirtualMachineResourceGroup" -VMName "MySecureVM" -VolumeType "all"
     ```

- **Disable encryption with the Azure CLI:** To disable encryption, use the [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) command. 

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type "all"
     ```

- **Disable encryption with a Resource Manager template:** 

    1. Click **Deploy to Azure** from the [Disable disk encryption on running Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/decrypt-running-windows-vm-without-aad) template.
    2. Select the subscription, resource group, location, VM, volume type, legal terms, and agreement.
    3.  Click **Purchase** to disable disk encryption on a running Windows VM.

### Remove the encryption extension

If you want to decrypt your disks and remove the encryption extension, you must disable encryption **before** removing the extension; see [disable encryption](#disable-encryption).

You can remove the encryption extension using Azure PowerShell or the Azure CLI. 

- **Disable disk encryption with Azure PowerShell:** To remove the encryption, use the [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension) cmdlet.

     ```azurepowershell-interactive
     Remove-AzVMDiskEncryptionExtension -ResourceGroupName "MyVirtualMachineResourceGroup" -VMName "MySecureVM"
     ```

- **Disable encryption with the Azure CLI:** To remove encryption, use the [az vm extension delete](/cli/azure/vm/extension#az-vm-extension-delete) command.

     ```azurecli-interactive
     az vm extension delete -g "MyVirtualMachineResourceGroup" --vm-name "MySecureVM" -n "AzureDiskEncryption"
     ```


## Next steps

- [Azure Disk Encryption overview](disk-encryption-overview.md)
- [Azure Disk Encryption sample scripts](disk-encryption-sample-scripts.md)
- [Azure Disk Encryption troubleshooting](disk-encryption-troubleshooting.md)
