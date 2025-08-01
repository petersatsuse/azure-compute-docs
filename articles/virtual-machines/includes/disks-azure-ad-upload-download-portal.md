---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 01/30/2024
 ms.author: rogarana
 ms.custom: include file
# Customer intent: As a system administrator, I want to restrict uploads and downloads for Azure managed disks using identity verification, so that I can enforce security policies and ensure only authorized users can access sensitive data.
---
If you're using [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to control resource access, you can now use it to restrict uploads and downloads of Azure managed disks. This feature is available as a GA offering in all regions. When a user attempts to upload or download a disk, Azure validates the identity of the requesting user in Microsoft Entra ID, and confirms that user has the required permissions. At a higher level, a system administrator could set a policy at the Azure account or subscription level, to ensure that all disks and snapshots must use Microsoft Entra ID for uploads or downloads. If you have any questions on securing uploads or downloads with Microsoft Entra ID, reach out to this email: azuredisks@microsoft .com

### Restrictions
[!INCLUDE [disks-azure-ad-upload-download-restrictions](disks-azure-ad-upload-download-restrictions.md)]

### Prerequisites
[!INCLUDE [disks-azure-ad-upload-download-prereqs](disks-azure-ad-upload-download-prereqs.md)]

### Assign RBAC role

To access managed disks secured with Microsoft Entra ID, the requesting user must have either the [Data Operator for Managed Disks](/azure/role-based-access-control/built-in-roles#data-operator-for-managed-disks) role, or a [custom role](/azure/role-based-access-control/custom-roles-portal) with the following permissions: 

- **Microsoft.Compute/disks/download/action**
- **Microsoft.Compute/disks/upload/action**
- **Microsoft.Compute/snapshots/download/action**
- **Microsoft.Compute/snapshots/upload/action**

For detailed steps on assigning a role, see the following articles for [portal](/azure/role-based-access-control/role-assignments-portal), [PowerShell](/azure/role-based-access-control/role-assignments-powershell), or [CLI](/azure/role-based-access-control/role-assignments-cli). To create or update a custom role, see the following articles for [portal](/azure/role-based-access-control/custom-roles-portal), [PowerShell](/azure/role-based-access-control/role-assignments-powershell), or [CLI](/azure/role-based-access-control/role-assignments-cli).

### Enable data access authentication mode

# [Portal](#tab/azure-portal)

Enable **data access authentication mode** to restrict access to the disk. You can either enable it when creating the disk, or you can enable it on the **Disk Export** page under **Settings** for existing disks.

:::image type="content" source="./media/disks-upload-download-portal/disks-data-access-auth-mode.png" alt-text="Screenshot of a disk's data access authentication mode checkbox, tick the checkbox to restrict access to the disk, and save your changes." lightbox="./media/disks-upload-download-portal/disks-data-access-auth-mode.png":::

# [PowerShell](#tab/azure-powershell)

Set `dataAccessAuthMode` to `"AzureActiveDirectory"` on your disk, in order to download it when it's been secured. Use the following script to update an existing disk, replace the values for `-ResourceGroupName` and `-DiskName` before running the script:

```azurepowershell
New-AzDiskUpdateConfig -DataAccessAuthMode "AzureActiveDirectory" | Update-AzDisk -ResourceGroupName 'yourResourceGroupName' -DiskName 'yourDiskName"
```

# [Azure CLI](#tab/azure-cli)

Set `dataAccessAuthMode` to `"AzureActiveDirectory"` on your disk, in order to download it when it's been secured. Use the following script to update an existing disk, replace the values for `--resource-group` and `--Name` before running the script:

```azurecli
az disk update --name yourDiskName --resource-group yourResourceGroup --data-access-auth-mode AzureActiveDirectory
```

---
