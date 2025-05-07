---
title: Convert Audit Logs to an Allowlist
description: Learn more about audit logs.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Convert audit logs to an allowlist

With Metadata Security Protocol (MSP), you can define custom role-based access control (RBAC) allowlists for metadata service endpoints. A new resource type in Azure Compute Gallery, `InVMAccessControlProfile`, enables this allowlist.

To learn more about RBAC and the `InVMAccessControlProfile` resource type, see [Advanced configuration](../advanced-configuration.md).

## Collect audit logs

If MSP is enabled in `Audit` or `Enforce` mode, the Guest Proxy Agent (GPA) creates audit logs in the virtual machine (VM).

| OS family | Audit log location |
|--|--|
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |

## Convert logs to rules

To create an allowlist, you can use an automated method or a manual method.

### Generate an allowlist automatically

1. Download and run the Allowlist Tool from either option:
   - Select `allowListTool.exe` from the [latest release page](https://github.com/Azure/GuestProxyAgent/releases/latest)
   - Direct download [link](https://github.com/Azure/GuestProxyAgent/releases/latest/download/allowListTool.exe)
1. Follow the steps in the Allowlist Tool to create your Allowlist and download the generated Allowlist.

### Manually create an allowlist

After a VM is enabled with MSP in Audit/Enforce mode, the proxy agent captures all the requests being made to the host endpoints. The logs are captured in the folder inside the VM:

| Operating system | Log file path |
|--|--|
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |

From the connection logs, you can analyze the applications that are making the requests to the Instance Metadata Service or WireServer endpoints:

[![Screenshot of first audit logs.](../images/create-shared-image-gallery/status-log.png)](../images/create-shared-image-gallery/status-log.png#lightbox)

The JSON captured here would be of the format:

[![Screenshot of second audit logs.](../images/create-shared-image-gallery/parse-json-from-logs.png)](../images/create-shared-image-gallery/parse-json-from-logs.png#lightbox)

From the log file, you can identify the endpoints that you want to secure (which would be the `privileges` in the final `InVMAccessControlProfile` instance), and the `identities` that should have access.

A simple rules schema might look like the following example.

[![Screenshot of third audit logs.](../images/create-shared-image-gallery/example-access-control-rules.png)](../images/create-shared-image-gallery/example-access-control-rules.png#lightbox)

> [!NOTE]
> We built an allowlist generator tool to make it easier to generate the Access Control rules. The allowlist tool helps parse the audit logs and provide a UI to generate the access control roles.

## Create an `InVMAccessControlProfile` instance by using ARM template

1. [Create a new private gallery](https://learn.microsoft.com/azure/virtual-machines/create-gallery) in Azure compute gallery.
1. Create an `InVMAccessControlProfile` definition with parameters for:
    - The gallery name to store in (from step 1)
    - Profile name
    - OS type
    - Host Endpoint type (Wireserver or IMDS)
1. Create a specific version.

Here's a sample `InVMAccessControlProfile` instance:

```
"properties": {
    "mode": "Enforce",
    "defaultAccess": "Allow",
    "rules": {
      "privileges": [
        {
          "name": "GoalState",
          "path": "/machine",
          "queryParameters": {
            "comp": "goalstate"
          }
        }
      ],
      "roles": [
        {
          "name": "Provisioning",
          "privileges": [
            "GoalState"
          ]
        },
        {
          "name": "ManageGuestExtensions",
          "privileges": [
            "GoalState"
          ]
        },
        {
          "name": "MonitoringAndSecret",
          "privileges": [
            "GoalState"
          ]
        }
      ],
      "identities": [
        {
          "name": "WinPA",
          "userName": "SYSTEM",
          "exePath": "C:\\Windows\\System32\\cscript.exe"
        },
        {
          "name": "GuestAgent",
          "userName": "SYSTEM",
          "processName": "WindowsAzureGuestAgent.exe"
        },
        {
          "name": "WaAppAgent",
          "userName": "SYSTEM",
          "processName": "WaAppAgent.exe"
        },
        {
          "name": "CollectGuestLogs",
          "userName": "SYSTEM",
          "processName": "CollectGuestLogs.exe"
        },
        {
          "name": "AzureProfileExtension",
          "userName": "SYSTEM",
          "processName": "AzureProfileExtension.exe"
        },
        {
          "name": "AzurePerfCollectorExtension",
          "userName": "SYSTEM",
          "processName": "AzurePerfCollectorExtension.exe"
        },
        {
          "name": "WaSecAgentProv",
          "userName": "SYSTEM",
          "processName": "WaSecAgentProv.exe"
        }
      ],
      "roleAssignments": [
        {
          "role": "Provisioning",
          "identities": [
            "WinPA"
          ]
        },
        {
          "role": "ManageGuestExtensions",
          "identities": [
            "GuestAgent",
            "WaAppAgent",
            "CollectGuestLogs"
          ]
        },
        {
          "role": "MonitoringAndSecret",
          "identities": [
            "AzureProfileExtension",
            "AzurePerfCollectorExtension",
            "WaSecAgentProv"
          ]
        }
      ]
    },
```
