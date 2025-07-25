---
title: Stackify Retrace Azure Linux Agent Extension
description: Deploy the Stackify Retrace Linux agent on a Linux virtual machine.
ms.topic: concept-article
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.author: gabsta
author: GabstaMSFT
ms.collection: linux
ms.date: 04/12/2018
ms.custom: devx-track-azurecli, linux-related-content
ms.devlang: azurecli
# Customer intent: As a developer managing applications on Linux virtual machines, I want to deploy the Stackify Retrace agent extension, so that I can monitor application performance and troubleshoot issues effectively across different environments.
---
# Stackify Retrace Linux Agent Extension

> [!CAUTION]
> This article references CentOS, a Linux distribution that is End Of Life (EOL) status. Please consider your use and plan accordingly. For more information, see the [CentOS End Of Life guidance](~/articles/virtual-machines/workloads/centos/centos-end-of-life.md).

## Overview

Stackify provides products that track details about your application to help find and fix problems quickly. For developer teams, Retrace is a fully integrated, multi-environment, app performance super-power. It combines several tools every development team needs.

Retrace is the ONLY tool that delivers all of the following capabilities across all environments in a single platform.

* Application performance management (APM)
* Application and server logging
* Error tracking and monitoring
* Server, application, and custom metrics

**About Stackify Linux Agent Extension**

This extension provides an install path for the Linux Agent for Retrace.

## Prerequisites

### Operating system

The Retrace agent can be run against these Linux distributions

| Distribution | Version |
|---|---|
| Ubuntu | 16.04 LTS |
| Debian |  9 |
| Red Hat | 6.10, 7.1+ |
| CentOS | 6.10, 7.0+ |

> [!IMPORTANT]

> Keep in consideration Red Hat Enterprise Linux 6.X is already EOL.
> RHEL 6.10 has available [ELS support](https://www.redhat.com/en/resources/els-datasheet), which [will end on 06/2024]( https://access.redhat.com/product-life-cycles/?product=Red%20Hat%20Enterprise%20Linux,OpenShift%20Container%20Platform%204).
### Internet connectivity

The Stackify Agent extension for Linux requires that the target virtual machine is connected to the internet.

You may need to adjust your network configuration to allow connections to Stackify, see https://support.stackify.com/hc/en-us/articles/207891903-Adding-Exceptions-to-a-Firewall.


## Extension schema

---

The following JSON shows the schema for the Stackify Retrace Agent extension. The extension requires the `environment` and `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }
```

## Template deployment

Azure VM extensions can be deployed with Azure Resource Manager templates. The JSON schema detailed in the previous section can be used in an Azure Resource Manager template to run the Stackify Retrace Linux Agent extension during an Azure Resource Manager template deployment.

The JSON for a virtual machine extension can be nested inside the virtual machine resource, or placed at the root or top level of a Resource Manager JSON template. The placement of the JSON affects the value of the resource name and type. For more information, see Set name and type for child resources.

The following example assumes the Stackify Retrace Linux extension is nested inside the virtual machine resource. When nesting the extension resource, the JSON is placed in the "resources": [] object of the virtual machine.

The extension requires the `environment` and `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }
```

When placing the extension JSON at the root of the template, the resource name includes a reference to the parent virtual machine, and the type reflects the nested configuration.

```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "<parentVmResource>/StackifyExtension",
        "apiVersion": "[variables('apiVersion')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Stackify.LinuxAgent.Extension",
            "type": "StackifyLinuxAgentExtension",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "environment": "myEnvironment"
            },
            "protectedSettings": {
              "activationKey": "myActivationKey"
            }
        }
    }
```


## PowerShell deployment

The `Set-AzVMExtension` command can be used to deploy the Stackify Retrace Linux Agent virtual machine extension to an existing virtual machine. Before running the command, the public and private configurations need to be stored in a PowerShell hash table.

The extension requires the `environment` and `activationKey`.

```powershell
$PublicSettings = @{"environment" = "myEnvironment"}
$ProtectedSettings = @{"activationKey" = "myActivationKey"}

Set-AzVMExtension -ExtensionName "Stackify.LinuxAgent.Extension" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Stackify.LinuxAgent.Extension" `
    -ExtensionType "StackifyLinuxAgentExtension" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS `
```

## Azure CLI deployment

The Azure CLI tool can be used to deploy the Stackify Retrace Linux Agent virtual machine extension to an existing virtual machine.

The extension requires the `environment` and `activationKey`.

```azurecli
az vm extension set --publisher 'Stackify.LinuxAgent.Extension' --version 1.0 --name 'StackifyLinuxAgentExtension' --protected-settings '{"activationKey":"myActivationKey"}' --settings '{"environment":"myEnvironment"}'  --resource-group 'myResourceGroup' --vm-name 'myVmName'
```

## Troubleshoot and support

### Error codes

| Error code | Meaning | Possible action |
| :---: | --- | --- |
| 10 | Install Error | wget is required |
| 20 | Install Error | Python is required |
| 30 | Install Error | sudo is required |
| 40 | Install Error | activationKey is required |
| 51 | Install Error | OS distro not supported |
| 60 | Install Error | environment is required |
| 70 | Install Error | Unknown |
| 80 | Enable Error | Service setup failed |
| 90 | Enable Error | Service startup failed |
| 100 | Disable Error | Service Stop Failed |
| 110 | Disable Error | Service Removal Failed |
| 120 | Uninstall Error | Service Stop Failed |

If you need more help you can contact Stackify support at https://support.stackify.com.
