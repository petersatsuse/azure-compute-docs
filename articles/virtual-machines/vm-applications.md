---
title: Overview of VM Applications in the Azure Compute Gallery
description: Learn about Azure VM Applications in an Azure Compute Gallery used to create and deploy applications on Azure Virtual Machines & Scale Sets.
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: concept-article
ms.date: 03/14/2025
author: ju-shim
ms.author: gabsta
ms.reviewer: jushiman
ms.custom: linux-related-content
# Customer intent: As a cloud engineer, I want to leverage VM Applications in the Azure Compute Gallery, so that I can efficiently deploy and manage applications across virtual machines without the overhead of rebuilding VM images for every change.
---

# VM Applications Overview

VM Applications are a resource type in Azure Compute Gallery that provides a modern, flexible approach to managing and deploying applications across your virtual machines and scale sets. By decoupling application installation from your base VM images, you can streamline updates, reduce image maintenance overhead, and accelerate deployment cycles. This approach eliminates the need to rebuild and republish VM images for every application change, enabling faster iteration and greater operational agility. VM Applications are best suited for deploying AI, high scale, low latency, micro-services, secure and compliant workloads on Azure VMs.

| Resource | Description|
|----------|------------|
| **Azure Compute Gallery** | A gallery is a repository for managing and sharing application packages. Users can share the gallery resource and all the child resources are shared automatically. The gallery name must be unique per subscription. For example, you may have one gallery to store all your OS images and another gallery to store all your VM applications.|
| **VM Application** | The definition of your VM application. It's a *logical* resource that stores the common metadata for all the versions under it. For example, you may have an application definition for Apache Tomcat and have multiple versions within it. |
| **VM Application version** | The deployable resource which holds your application package and version specific configurations. You can globally replicate your VM application versions to target regions closer to your VM infrastructure. The VM Application version must be replicated to a region before it may be deployed on a VM in that region. |
| **Storage Account**| Application packages are first uploaded to your storage account. Azure Compute Gallery then downloads the application package from this storage account using SAS URLs and stores it within the VM Application version. Azure Compute Gallery also replicates this package across regions & regional replicas per the VM Application version definition. The application package in the storage account can be deleted after VM application version is created in Azure Compute Gallery. |   

:::image type="content" source="media/vmapps/vm-application-overview.png" alt-text="Diagram showing steps to create VM application and deploying it to Azure":::

## Key Benefits: 
- **Centralized and Flexible Application Management**: 
  - Package Once, Deploy Anywhere: Package applications in ZIP, MSI (Microsoft Package Installed), or EXE formats and manage them centrally in Azure Compute Gallery.
  - Version Control: Deploy either the latest or a specific version by maintaining multiple versions of each application. 
- **Seamless Sharing and Access Control**
  - Tenant-Wide Sharing: Share applications within teams or across your entire organization (tenant).
  - Integrated RBAC: Control publishing and deployment access using Azure Role-Based Access Control (RBAC).
- **Reliable and Customizable Deployments**
  - Individual Application Control: Install, update, or delete applications independently—no need to rebuild VM images.
  - Customizable Operations: Customize install, update, and delete operations for applications, including reboot handling.
  - Built-In Failure Handling: Ensure resilient deployments by connecting VM application failure to VM failure.
- **Scalable and Low-Latency Distribution**
  - Global and Intra-Region Replication: Automatically replicate applications across and within regions to reduce latency and improve resiliency—no need for AzCopy or manual transfers.
  - Optimized for High-Scale Scenarios: Achieve low create latency even during large-scale deployments.
- **Secure and Compliant by Design**
  - Policy-Driven Enforcement: Use Azure Policy to enforce application presence and configuration across your fleet.
  - Secure Deployments: Avoid internet-based downloads and complex private link setups which aren't ideal for locked-down or secure environments.
- **Broad Platform Support**
  - VMs and Scale Sets: Deploy to individual VMs, flexible scale sets, or uniform scale sets with full support.
  - Block Blob Support: Efficiently handle large application packages (upto 2 GB) using Azure Block Blobs for chunked uploads and background streaming.


## VM Applications & VM Applications version resource

The VM application resource defines the following about your VM application:
- Azure Compute Gallery where the VM application is stored
- Name of the application
- Supported OS type like Linux or Windows
- A description of the VM application

VM application versions are the deployable resource. Versions are defined with the following properties:
- Version number
- Link to the application package file in a storage account
- Install string to properly install the application
- Remove string to properly remove the application
- Update string to properly update the VM application to a newer version
- Package file name to use when the package is downloaded to the VM.
- Configuration file name to be used to configure the application on the VM
- A link to the configuration file for the VM application, which you can include license files
- End-of-life date. End-of-life dates are informational; you're still able to deploy VM Application versions past the end-of-life date.
- Exclude from latest. You can keep a version from being used as the latest version of the application.
- Target regions for replication
- Replica count per region

#### [Template](#tab/template)
```json

{
  "$schema": "https://schema.management.azure.com/schemas/2020-06-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "galleryName": {
      "type": "string"
    },
    "applicationName": {
      "type": "string"
    },
    "versionName": {
      "type": "string",
      "metadata": {
        "description": "Must follow the format: major.minor.patch (e.g., 1.0.0)"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "West US"
    },
    "supportedOSType": {
      "type": "string",
      "allowedValues": ["Windows", "Linux"]
    },
    "endOfLifeDate": {
      "type": "string"
    },
    "description": {
      "type": "string",
      "defaultValue": "Description of the application"
    },
    "eula": {
      "type": "string",
      "defaultValue": ""
    },
    "privacyStatementUri": {
      "type": "string",
      "defaultValue": ""
    },
    "releaseNoteUri": {
      "type": "string",
      "defaultValue": ""
    },
    "mediaLink": {
      "type": "string"
    },
    "configLink": {
      "type": "string"
    },
    "appConfigFileName": {
      "type": "string"
    },
    "appPackageFileName": {
      "type": "string"
    },
    "replicaRegion1": {
      "type": "string",
      "defaultValue": "East US"
    },
    "replicaRegion2": {
      "type": "string",
      "defaultValue": "South Central US"
    },
    "installScript": {
      "type": "string",
      "metadata": {
        "description": "Optional. Script to run to install the application. E.g. echo 'Installing application...'"
      }
    },
    "updateScript": {
      "type": "string",
      "metadata": {
        "description": "Optional. Script to run to update the application. E.g. echo 'Updating application...'"
      }
    },
    "removeScript": {
      "type": "string",
      "metadata": {
        "description": "Optional. Script to run to delete the application. E.g. echo 'Deleting application...'"
      }
    },
    "storageAccountType": {
      "type": "string",
      "allowedValues": ["PremiumV2_LRS", "Premium_LRS", "Standard_LRS", "Standard_ZRS"],
      "defaultValue": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/galleries",
      "apiVersion": "2024-03-03",
      "name": "[parameters('galleryName')]",
      "location": "[parameters('location')]",
      "properties": {
        "identifier": {}
      }
    },
    {
      "type": "Microsoft.Compute/galleries/applications",
      "apiVersion": "2024-03-03",
      "name": "[format('{0}/{1}', parameters('galleryName'), parameters('applicationName'))]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/galleries', parameters('galleryName'))]"
      ],
      "properties": {
        "supportedOSType": "[parameters('supportedOSType')]",
        "endOfLifeDate": "[parameters('endOfLifeDate')]",
        "description": "[parameters('description')]",
        "eula": "[if(equals(parameters('eula'), ''), json('null'), parameters('eula'))]",
        "privacyStatementUri": "[if(equals(parameters('privacyStatementUri'), ''), json('null'), parameters('privacyStatementUri'))]",
        "releaseNoteUri": "[if(equals(parameters('releaseNoteUri'), ''), json('null'), parameters('releaseNoteUri'))]"
      }
    },
    {
      "type": "Microsoft.Compute/galleries/applications/versions",
      "apiVersion": "2024-03-03",
      "name": "[format('{0}/{1}/{2}', parameters('galleryName'), parameters('applicationName'), parameters('versionName'))]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/galleries/applications', parameters('galleryName'), parameters('applicationName'))]"
      ],
      "properties": {
        "publishingProfile": {
          "source": {
            "mediaLink": "[parameters('mediaLink')]",
            "defaultConfigurationLink": "[parameters('configLink')]"
          },
          "manageActions": {
            "install": "[parameters('installScript')]",
            "remove": "[parameters('removeScript')]",
            "update": "[parameters('updateScript')]"
          },
          "settings": {
            "scriptBehaviorAfterReboot": "Rerun",
            "configFileName": "[parameters('appConfigFileName')]",
            "packageFileName": "[parameters('appPackageFileName')]"
          },
          "targetRegions": [
            {
              "name": "[parameters('location')]",
              "regionalReplicaCount": 3,
              "storageAccountType": "[parameters('storageAccountType')]"
            },
            {
              "name": "[parameters('replicaRegion1')]",
              "regionalReplicaCount": 1,
              "storageAccountType": "[parameters('storageAccountType')]"
            },
            {
              "name": "[parameters('replicaRegion2')]"
            },
          ],
          "excludeFromLatest": false,
          "replicaCount": 2,
          "storageAccountType": "[parameters('storageAccountType')]"
        },
        "safetyProfile": {
          "allowDeletionOfReplicatedLocations": true
        },
        "endOfLifeDate": "[parameters('endOfLifeDate')]"
      }
    }
  ]
}

```
---

## Cost

There's no extra charge for using VM Application Packages, but you're charged for the following resources:

- Storage costs of storing each package and any replicas.
- Network egress charges for replication of the first image version from the source region to the replicated regions. Subsequent replicas are handled within the region, so there are no extra charges.

For more information on network egress, see [Bandwidth pricing](https://azure.microsoft.com/pricing/details/bandwidth/).

## Technical Details

### Considerations and Current Limits

- **Up to 10 replicas per region**: When you're creating a VM Application version, the maximum number of replicas per region is 10 for both page blob and block blob.

- **Storage with public access and SAS URI with read privilege:** The storage account needs to have public level access and use a SAS URI with read privilege, as other restriction levels fail deployments. SAS Tokens can be bypassed by publishing the artifact on the storage account by allowing anonymous access.

- **Manual retry for failed installations**: Currently, the only way to retry a failed installation is to remove the application from the profile, then add it back. 

- **Up to 25 applications per VM**: A maximum of 25 applications can be deployed to a single virtual machine.

- **2 GB application size**: The maximum file size of an application version `mediaLink` is 2 GB. The maximum file size for `defaultConfigurationLink` is 1 GB. 

- **Requires a VM Agent**: The VM agent must exist on the VM and be able to receive goal states.

- **Single version of the application per VM**: Only one version of a given application can be deployed to a VM.
  
- **Move operations currently not supported**: Moving VMs with associated VM Applications across resource groups is currently not supported.

> [!NOTE]
> For Azure Compute Gallery and VM Applications, Storage SAS can be deleted after replication. However, any subsequent update operation requires a valid SAS.

### Download directory within the VM

The download location of the application package and the configuration files are:

- Linux: `/var/lib/waagent/Microsoft.CPlat.Core.VMApplicationManagerLinux/<application name>/<application version> `
- Windows: `C:\Packages\Plugins\Microsoft.CPlat.Core.VMApplicationManagerWindows\1.0.9\Downloads\<application name>\<application version> `

The install/update/remove commands should be written assuming the application package and the configuration file are in the current directory.

### File naming

When the application file gets downloaded to the VM, the file is renamed as "MyVmApp" and has no file extension (E.g. .exe, .msi). The VM is unaware of the file's original name and extension . 

Here are a few alternatives to navigate this issue:

You can modify your script to include a command for renaming the file before execution:
```azurepowershell
move .\\MyVmApp .\\MyApp.exe & MyApp.exe /S
```
You can also use the `packageFileName` (and the corresponding `configFileName`) property to instruct us what to rename your file. For example, setting it to "MyApp.exe" makes your install script as follows.
```powershell
MyAppe.exe /S
```
> [!TIP]
> If your blob is originally named as "myApp.exe" instead of "myapp", then the script works without setting the `packageFileName` property.

### Command interpreter

The default command interpreters are:

- Linux: `/bin/bash`
- Windows: `cmd.exe`

It's possible to use a different interpreter like Chocolatey or PowerShell, as long as it's installed on the machine, by calling the executable and passing the command to it. For example, to have your command run in PowerShell on Windows instead of cmd, you can pass `powershell.exe -Command '<powershell commmand>'`

### How updates are handled

When you update an application version on a VM or Virtual Machine Scale Sets, the update command you provided during deployment is used. If the updated version doesn't have an update command, then the current version is removed and the new version is installed.

Update commands should be written with the expectation that it could be updating from any older version of the VM Application.

### Treat failure as deployment failure

The VM Application extension always returns a *success* regardless of whether any VM app failed while being installed/updated/removed. The VM Application extension only reports the extension status as failure when there's a problem with the extension or the underlying infrastructure. This behavior is triggered by the "treat failure as deployment failure" flag, which is set to `$false` by default and can be changed to `$true`. The failure flag can be configured in [PowerShell](/powershell/module/az.compute/add-azvmgalleryapplication#parameters) or [CLI](/cli/azure/vm/application#az-vm-application-set).

## Creating VM Applications on Linux
To create a VM Application, you need application package and scripts to properly install, update, and delete the application.  
Third party applications for Linux can be packaged in a few ways. Let's explore how to handle creating the install commands for some of the most common.

### .tar and .gz files

These files are compressed archives and can be extracted to a desired location. Check the installation instructions for the original package to in case they need to be extracted to a specific location. If .tar.gz file contains source code, see the instructions for the package for how to install from source.

Example to install command to install `golang` on a Linux machine:

```bash
sudo tar -C /usr/local -xzf go_linux
```

Example remove command:

```bash
sudo rm -rf /usr/local/go
```

### Creating application packages using `.deb`, `.rpm`, and other platform specific packages for VMs with restricted internet access

You can download individual packages for platform specific package managers, but they usually don't contain all the dependencies. For these files, you must also include all dependencies in the application package, or have the system package manager download the dependencies through the repositories that are available to the VM. If you're working with a VM with restricted internet access, you must package all the dependencies yourself.

Figuring out the dependencies can be a bit tricky. There are third party tools that can show you the entire dependency tree.

#### [Ubuntu](#tab/ubuntu)

In Ubuntu, you can run `sudo apt show <package_name> | grep Depends` to show all the packages that are installed when executing the `sudo apt-get install <packge_name>` command. Then you can use that output to download all `.deb` files to create an archive that can be used as the application package.

To create a VM Application package for installing PowerShell on Ubuntu, perform the following steps: 
1. Run the following commands to enable the repository for downloading PowerShell and to identify package dependencies on a new Ubuntu VM
   
```bash
# Download the Microsoft repository GPG keys
wget -q "https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb"
# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb
sudo rm -rf packages-microsoft-prod.deb
sudo apt update
sudo apt show powershell | grep Depends
```

2. Check the output of the line **Depends** which lists the following packages:

```output
Depends: libc6, lib32gcc-s1, libgssapi-krb5-2, libstdc++6, zlib1g, libicu72|libicu71|libicu70|libicu69|libicu68|libicu67|libicu66|libicu65|libicu63|libicu60|libicu57|libicu55|libicu52, libssl3|libssl1.1|libssl1.0.2|libssl1.
```

3. Download each of these files using `sudo apt-get download <package_name>` and create a tar compressed archive with all files.

- Ubuntu 18.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc-s1
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl1.1
sudo apt-get download libicu60
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

- Ubuntu 20.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc-s1
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl1.1
sudo apt-get download libicu66
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

- Ubuntu 22.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc-s1
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl3
sudo apt-get download libicu70
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

4. This tar archive is the application package file.

- The install command in this case is:

```bash
sudo tar -xvzf powershell.tar.gz && sudo dpkg -i *.deb
```

- And the remove command is:

```bash
sudo apt remove powershell
```

Use `sudo apt autoremove` instead of explicitly trying to remove all the dependencies. You may have installed other applications with overlapping dependencies, and in that case, an explicit remove command would fail.

In case you don't want to resolve the dependencies yourself, and `apt` is able to connect to the repositories, you can install an application with just one `.deb` file and let `apt` handle the dependencies.

Example install command:

```bash
dpkg -i <package_name> || apt --fix-broken install -y
```

#### [Red Hat](#tab/rhel)

In Red Hat, you can run `sudo yum deplist <package_name>` to show all the packages that are installed when executing the `sudo yum install <package_name>` command. Then you can use that output to download all `.rpm` files to create an archive that can be used as the application package.

1. Example, to create a VM Application package to install PowerShell for Red Hat, first run the following commands to enable the repository where PowerShell can be downloaded from and also to identify the package dependencies on a new Red Hat Enterprise Linux (RHEL) VM.

- RHEL 7:

```bash
# Register the Microsoft RedHat repository
curl https://packages.microsoft.com/config/rhel/7/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo

sudo yum deplist powershell
```

- RHEL 8:

```bash
# Register the Microsoft RedHat repository
curl https://packages.microsoft.com/config/rhel/8/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo

sudo dnf deplist powershell
```

2. Check the output of each of the dependency entries, the dependencies are named after `provider:`:

```output
  dependency: /bin/sh
   provider: bash.x86_64 4.2.46-35.el7_9
  dependency: libicu
   provider: libicu.x86_64 50.2-4.el7_7
   provider: libicu.i686 50.2-4.el7_7
  dependency: openssl-libs
   provider: openssl-libs.x86_64 1:1.0.2k-26.el7_9
   provider: openssl-libs.i686 1:1.0.2k-26.el7_9
```

3. Download each of these files using `sudo yum install --downloadonly <package_name>`, to download a package when isn't yet installed in the system, or `sudo yum reinstall --downloadonly <package_name>`, to download a package that's already installed in the system, and create a tar compressed archive with all files.

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo yum reinstall --downloadonly --downloaddir=/tmp/powershell bash
sudo yum reinstall --downloadonly --downloaddir=/tmp/powershell libicu
sudo yum reinstall --downloadonly --downloaddir=/tmp/powershell openssl-libs
sudo yum install --downloadonly --downloaddir=/tmp/powershell powershell
sudo tar -cvzf powershell.tar.gz *.rpm
```

4. This tar archive is the application package file.

- The install command in this case is:

```bash
sudo tar -xvzf powershell.tar.gz && sudo yum install *.rpm -y
```

- And the remove command is:

```bash
sudo yum remove powershell
```

In case you don't want to resolve the dependencies yourself and yum/dnf is able to connect to the repositories, you can install an application with just one `.rpm` file and let yum/dnf handle the dependencies.

Example install command:

```bash
yum install <package.rpm> -y
```

#### [SUSE](#tab/sles)

In SUSE, you can run `sudo zypper info --requires <package_name>` to show all the packages that are installed when executing the `sudo zypper install <package_name>` command. Then you can use that output to download all `.rpm` files to create an archive that can be used as the application package.

1. Example, to create a VM Application package to install `azure-cli` for SUSE, first run the following commands to enable the repository where Azure CLI can be downloaded from and also to identify the package dependencies on a new SUSE VM.

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo zypper addrepo --name 'Azure CLI' --check https://packages.microsoft.com/yumrepos/azure-cli azure-cli
sudo zypper info --requires azure-cli
```

2. Check the output after **Requires** which lists the following packages:

```output
Requires       : [98]
    /usr/bin/python3
    python(abi) = 3.6
    azure-cli-command-modules-nspkg >= 2.0
    azure-cli-nspkg >= 3.0.3
    python3-azure-loganalytics >= 0.1.0
    python3-azure-mgmt-apimanagement >= 0.2.0
    python3-azure-mgmt-authorization >= 0.61.0
    python3-azure-mgmt-batch >= 9.0.0
    python3-azure-mgmt-cognitiveservices >= 6.3.0
    python3-azure-mgmt-containerservice >= 9.4.0
    python3-azure-mgmt-cosmosdb >= 1.0.0
    python3-azure-mgmt-datalake-store >= 0.5.0
    python3-azure-mgmt-deploymentmanager >= 0.2.0
    python3-azure-mgmt-imagebuilder >= 0.4.0
    python3-azure-mgmt-iothubprovisioningservices >= 0.2.0
    python3-azure-mgmt-maps >= 0.1.0
    python3-azure-mgmt-media >= 2.1.0
<truncated>
...
<truncated>
    python3-vsts-cd-manager >= 1.0.2
    python3-websocket-client >= 0.56.0
    python3-xmltodict >= 0.12
    python3-azure-mgmt-keyvault >= 8.0.0
    python3-azure-mgmt-storage >= 16.0.0
    python3-azure-mgmt-billing >= 1.0.0
    python3-azure-mgmt-cdn >= 5.2.0
    python3-azure-mgmt-hdinsight >= 2.0.0
    python3-azure-mgmt-netapp >= 0.14.0
    python3-azure-mgmt-synapse >= 0.5.0
    azure-cli-core = 2.17.1
    python3-azure-batch >= 10.0
    python3-azure-mgmt-compute >= 18.0
    python3-azure-mgmt-containerregistry >= 3.0.0rc16
    python3-azure-mgmt-databoxedge >= 0.2.0
    python3-azure-mgmt-network >= 17.0.0
    python3-azure-mgmt-security >= 0.6.0
```

3. Download each of these files using `sudo zypper install -f --download-only <package_name>` and create a tar compressed archive with all files.

```bash
mkdir /tmp/azurecli
cd /tmp/azurecli
for i in $(sudo zypper info --requires azure-cli | sed -n -e '/Requires*/,$p' | grep -v "Requires" | awk -F '[>=]' '{print $1}') ; do sudo zypper --non-interactive --pkg-cache-dir /tmp/azurecli install -f --download-only $i; done
for i in $(sudo find /tmp/azurecli -name "*.rpm") ; do sudo cp $i /tmp/azurecli; done
sudo tar -cvzf azurecli.tar.gz *.rpm
```

4. This tar archive is the application package file.

- The install command in this case is:

```bash
sudo tar -xvzf azurecli.tar.gz && sudo zypper --no-refresh --no-remote --non-interactive install *.rpm
```

- And the remove command is:

```bash
sudo zypper remove azure-cli
```

---

## Creating VM Applications on Windows

Most third party applications in Windows are available as .exe or .msi installers. Some are also available as extract and run zip files. Let us look at the best practices for each of them.

### .exe installer

Installer executables typically launch a user interface (UI) and require someone to select through the UI. If the installer supports a silent mode parameter, it should be included in your installation string.

Cmd.exe also expects executable files to have the extension `.exe`, so you need to rename the file to have the `.exe` extension.

If I want to create a VM Application package for `myApp.exe`, which ships as an executable, my VM Application is called 'myApp', so I write the command assuming the application package is in the current directory:

```terminal
"move .\\myApp .\\myApp.exe & myApp.exe /S -config myApp_config"
```

If the installer executable file doesn't support an uninstall parameter, you can sometimes look up the registry on a test machine to know where the uninstaller is located.

In the registry, the uninstall string is stored in `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\<installed application name>\UninstallString` so I would use the contents as my remove command:

```terminal
'\"C:\\Program Files\\myApp\\uninstall\\helper.exe\" /S'
```

### .msi installer

For command line execution of `.msi` installers, the commands to install or remove an application should use `msiexec`. Typically, `msiexec` runs as its own separate process and `cmd` doesn't wait for it to complete, which can lead to problems when installing more than one VM application.  The `start` command can be used with `msiexec` to ensure that the installation completes before the command returns. For example:

```terminal
start /wait %windir%\\system32\\msiexec.exe /i myapp /quiet /forcerestart /log myapp_install.log
```

Example remove command:

```terminal
start /wait %windir%\\system32\\msiexec.exe /x myapp /quiet /forcerestart /log myapp_uninstall.log
```

Typically, the `start` command would be called within a batch script. If used with the `/wait` parameter, the calling script is paused until the called process terminates. Once complete, the batch script would check for the `errorlevel` variable set by the `start` command and exit as follows:
 
```batch
start /wait %windir%\\system32\\msiexec.exe /i myapp /quiet /forcerestart /log myapp_install.log
if %errorlevel% neq 0 exit /b %errorlevel%
...
```

### Zipped files

For .zip or other zipped files, rename and unzip the contents of the application package to the desired destination.

Example install command:

```terminal
rename myapp myapp.zip && mkdir C:\myapp && powershell.exe -Command "Expand-Archive -path myapp.zip -destinationpath C:\myapp"
```

Example remove command:

```terminal
rmdir /S /Q C:\\myapp
```

## Troubleshooting VM Applications

To know whether a particular VM Application was successfully added to the VM instance, check the message of the VM Application extension.

To learn more about getting the status of VM extensions, see [Virtual machine extensions and features for Linux](extensions/features-linux.md#view-extension-status) and [Virtual machine extensions and features for Windows](extensions/features-windows.md#view-extension-status).

To get status of VM extensions, use [Get-AzVM](/powershell/module/az.compute/get-azvm):

```azurepowershell-interactive
Get-AzVM -name <VM name> -ResourceGroupName <resource group name> -Status | convertto-json -Depth 10
```

To get status of scale set extensions, use [Get-AzVMSS](/powershell/module/az.compute/get-azvmss):

```azurepowershell-interactive
$result = Get-AzVmssVM -ResourceGroupName $rgName -VMScaleSetName $vmssName -InstanceView
$resultSummary  = New-Object System.Collections.ArrayList
$result | ForEach-Object {
    $res = @{ instanceId = $_.InstanceId; vmappStatus = $_.InstanceView.Extensions | Where-Object {$_.Name -eq "VMAppExtension"}}
    $resultSummary.Add($res) | Out-Null
}
$resultSummary | convertto-json -depth 5
```

### Error messages

| Message | Description |
|--|--|
| Current VM Application Version {name} was deprecated at {date}. | You tried to deploy a VM Application version that has already been deprecated. Try using `latest` instead of specifying a specific version. |
| Current VM Application Version {name} supports OS {OS}, while current OSDisk's OS is {OS}. | You tried to deploy a Linux application to Windows instance or vice versa. |
| The maximum number of VM applications (max=5, current={count}) has been exceeded. Use fewer applications and retry the request. | We currently only support five VM applications per VM or scale set. |
| More than one VM Application was specified with the same packageReferenceId. | The same application was specified more than once. |
| Subscription not authorized to access this image. | The subscription doesn't have access to this application version. |
| Storage account in the arguments doesn't exist. | There are no applications for this subscription. |
| The platform image {image} isn't available. Verify that all fields in the storage profile are correct. For more details about storage profile information, see https://aka.ms/storageprofile. | The application doesn't exist. |
| The gallery image {image} isn't available in {region} region. Contact image owner to replicate to this region, or change your requested region. | The gallery application version exists, but it wasn't replicated to this region. |
| The SAS isn't valid for source uri {uri}. | A `Forbidden` error was received from storage when attempting to retrieve information about the url (either mediaLink or defaultConfigurationLink). |
| The blob referenced by source uri {uri} doesn't exist. | The blob provided for the mediaLink or defaultConfigurationLink properties doesn't exist. |
| The gallery application version url {url} can't be accessed due to the following error: remote name not found. Ensure that the blob exists and that it's either publicly accessible or is a SAS url with read privileges. | The most likely case is that a SAS uri with read privileges wasn't provided. |
| The gallery application version url {url} can't be accessed due to the following error: {error description}. Ensure that the blob exists and that it's either publicly accessible or is a SAS url with read privileges. | There was an issue with the storage blob provided. The error description provides more information. |
| Operation {operationName} isn't allowed on {application} since it's marked for deletion. You can only retry the delete operation (or wait for an ongoing one to complete). | Attempt to update an application that's currently being deleted. |
| The value {value} of parameter 'galleryApplicationVersion.properties.publishingProfile.replicaCount' is out of range. The value must be between one and three, inclusive. | Only between one and three replicas are allowed for VM Application versions. |
| Changing property 'galleryApplicationVersion.properties.publishingProfile.manageActions.install' isn't allowed. (Or update, delete) | It isn't possible to change any of the manage actions on an existing VmApplication. A new VmApplication version must be created. |
| Changing property ' galleryApplicationVersion.properties.publishingProfile.settings.packageFileName ' isn't allowed. (Or configFileName) | It isn't possible to change any of the settings, such as the package file name or config file name. A new VmApplication version must be created. |
| The blob referenced by source uri {uri} is too large: size = {size}. The maximum blob size allowed is '1 GB'. | The maximum size for a blob referred to by mediaLink or defaultConfigurationLink is currently 1 GB. |
| The blob referenced by source uri {uri} is empty. | An empty blob was referenced. |
| {type} blob type isn't supported for {operation} operation. Only page blobs and block blobs are supported. | VmApplications only supports page blobs and block blobs. |
| The SAS isn't valid for source uri {uri}. | The SAS uri supplied for mediaLink or defaultConfigurationLink isn't a valid SAS uri. |
| Can't specify {region} in target regions because the subscription is missing required feature {featureName}. Either register your subscription with the required feature or remove the region from the target region list. | To use VmApplications in certain restricted regions, one must have the feature flag registered for that subscription. |
| Gallery image version publishing profile regions {regions} must contain the location of image version {location}. | The list of regions for replication must contain the location where the application version is. |
| Duplicate regions aren't allowed in target publishing regions. | The publishing regions may not have duplicates. |
| Gallery application version resources currently don't support encryption. | The encryption property for target regions isn't supported for VM Applications |
| Entity name doesn't match the name in the request URL. | The gallery application version specified in the request url doesn't match the one specified in the request body. |
| The gallery application version name is invalid. The application version name should follow Major(int32). Minor(int32). Patch(int32) format, where `int` is between 0 and 2,147,483,647 (inclusive). For example, 1.0.0, 2018.12.1 etc. | The gallery application version must follow the format specified. |

## Next steps

- Learn how to [create and deploy VM application packages](vm-applications-how-to.md).
