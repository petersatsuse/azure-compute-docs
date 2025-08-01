---
title: Troubleshooting Windows VM extension failures
description: Learn about troubleshooting Azure Windows VM extension failures
ms.topic: troubleshooting
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.custom: devx-track-arm-template
ms.author: gabsta
author: GabstaMSFT
ms.reviewer: jushiman
ms.collection: windows
ms.date: 04/21/2023
# Customer intent: As a system administrator, I want to troubleshoot Windows VM extension failures so that I can ensure all deployed extensions are functioning properly and maintain the stability of my Azure environment.
---
# Troubleshooting Azure Windows VM extension failures
[!INCLUDE [virtual-machines-common-extensions-troubleshoot](../includes/virtual-machines-common-extensions-troubleshoot.md)]

## Viewing extension status
Azure Resource Manager templates can be executed from Azure PowerShell. Once the template is executed, the extension status can be viewed from Azure Resource Explorer or the command-line tools.

Here's an example:

Azure PowerShell:

```azurepowershell
Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status
```

Here's the sample output:

```output
Extensions:  {
  "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
  "Name": "myCustomScriptExtension",
  "SubStatuses": [
    {
      "Code": "ComponentStatus/StdOut/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
          \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
          test.txt                          \\n\\n",
                  "Time": null
      },
    {
      "Code": "ComponentStatus/StdErr/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "",
      "Time": null
    }
  ]
}
```

## Troubleshooting extension failures

### Verify that the VM Agent is running and Ready
The VM Agent is required to manage, install and execute extensions. If the VM Agent isn't running or is failing to report a Ready status to the Azure platform, then the extensions won't work correctly.

Please refer to the following pages to troubleshoot the VM Agent:
- [Troubleshooting Windows Azure Guest Agent](/troubleshoot/azure/virtual-machines/windows-azure-guest-agent) for a Windows VM
- [Troubleshoot the Azure Linux Agent](/troubleshoot/azure/virtual-machines/linux-azure-guest-agent) for a Linux VM

### Check for your specific extension troubleshooting guide
Some extensions have a specific page describing how to troubleshoot them. You can find the list of these extensions and pages on [Troubleshoot extensions
](./overview.md#troubleshoot-extensions).

### View the extension's status
As explained above, the extension's status can be found by running the PowerShell cmdlet:
```azurepowershell
Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status
```

or the CLI command:
```azurecli
az vm extension show -g <RG Name> --vm-name <VM Name>  --name <Extension Name>
```

or in the Azure portal, by browsing to the VM Blade / Settings / Extensions. You can then click on the extension and check its status and message.


### Rerun the extension on the VM
If you're running scripts on the VM using Custom Script Extension, you could sometimes run into an error where VM was created successfully but the script has failed. Under these conditions, the recommended way to recover from this error is to remove the extension and rerun the template again.
Note: In future, this functionality would be enhanced to remove the need for uninstalling the extension.

#### Remove the extension from Azure PowerShell
```azurepowershell
Remove-AzVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"
```

Once the extension has been removed, the template can be re-executed to run the scripts on the VM.

### Trigger a new GoalState to the VM
You might notice that an extension hasn't been executed, or is failing to execute because of a missing "Windows Azure CRP Certificate Generator" (that certificate is used to secure the transport of the extension's protected settings).
That certificate will be automatically regenerated by restarting the Windows Guest Agent from inside the Virtual Machine:
- Open the Task Manager
- Go to the Details tab
- Locate the WindowsAzureGuestAgent.exe process
- Right-click, and select "End Task". The process will be automatically restarted


You can also trigger a new GoalState to the VM, by executing a "VM Reapply". VM [Reapply](/rest/api/compute/virtualmachines/reapply) is an API introduced in 2020 to reapply a VM's state. We recommend doing this at a time when you can tolerate a short VM downtime. While Reapply itself doesn't cause a VM reboot, and the vast majority of times calling Reapply won't reboot the VM, there's a very small risk that some other pending update to the VM model gets applied when Reapply triggers a new goal state, and that other change could require a restart. 

Azure portal:

In the portal, select the VM and in the left pane under the **Support + troubleshooting**, select **Redeploy + reapply**, then select **Reapply**.


Azure PowerShell *(replace the RG Name and VM Name with your values)*:

```azurepowershell
Set-AzVM -ResourceGroupName <RG Name> -Name <VM Name> -Reapply
```

Azure CLI *(replace the RG Name and VM Name with your values)*:

```azurecli
az vm reapply -g <RG Name> -n <VM Name>
```

If a "VM Reapply" didn't work, you can add a new empty Data Disk to the VM from the Azure Management Portal, and then remove it later once the certificate has been added back.


### Look at the extension logs inside the VM

If the previous steps didn't work and if your extension is still in a failed state, the next step is to look at its logs inside the Virtual Machine.

On a **Windows** VM, the extension logs will typically reside in 
```
C:\WindowsAzure\Logs\Plugins
```
And the Extension settings and status files will be in 
```
C:\Packages\Plugins
```
<br/>

On a **Linux** VM,  the extension logs will typically reside in 
```
/var/log/azure/
```
And the Extension settings and status files will be in 
```
/var/lib/waagent/
```


Each extension is different, but they usually follow similar principles:

Extension packages and binaries are downloaded on the VM (eg. _"/var/lib/waagent/custom-script/download/1"_ for Linux or _"C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.10.12\Downloads\0"_ for Windows). 

Their configuration and settings are passed from Azure Platform to the extension handler through the VM Agent (eg. _"/var/lib/waagent/Microsoft.Azure.Extensions.CustomScript-2.1.3/config"_ for Linux or  _"C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.10.12\RuntimeSettings"_ for Windows)

Extension handlers inside the VM are writing to a status file (eg. _"/var/lib/waagent/Microsoft.Azure.Extensions.CustomScript-2.1.3/status/1.status"_ for Linux or _"C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.10.12\Status"_ for Windows) which will then be reported to the Azure Platform. That status is the one reported through PowerShell, CLI or in the VM's extension blade in the Azure portal.

They also write detailed logs of their execution (eg. _"/var/log/azure/custom-script/handler.log"_ for Linux or _"C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension\1.10.12\CustomScriptHandler.log"_ for Windows).


### If the VM is recreated from an existing VM

It could happen that you're creating an Azure VM based on a specialized Disk coming from another Azure VM. In that case, it's possible that the old VM contained  extensions, and so will have binaries, logs and status files left over. The new VM model won't be aware of the previous VM's extensions states, and it might report an incorrect status for these extensions. We strongly recommend you to remove the extensions from the old VM before creating the new one, and then reinstall these extensions once the new VM is created.
The same can happen when you create a generalized image from an existing Azure VM. We invite you to remove extensions to avoid inconsistent state from the extensions.


## Known issues


### PowerShell isn't recognized as an internal or external command

You notice the following error entries in the RunCommand extension's output:

```Log sample
RunCommandExtension failed with "'powershell' isn't recognized as an internal or external command,"
```

**Analysis**

Extensions run under Local System account, so it's very possible that powershell.exe works fine when you RDP into the VM, but fails when run with RunCommand.

**Solution**

- Check that PowerShell is properly listed in the PATH environment variable:
    - Open Control Panel
    - System and Security
    - System
    - Advanced tab -> Environmental Variables
- Under 'System variables' click edit and ensure that PowerShell is in the PATH environment variable (usually: "C:\Windows\System32\WindowsPowerShell\v1.0")
- Reboot the VM or restart the WindowsAzureGuestAgent service then try the Run Command again.


### Command isn't recognized as an internal or external command

You see the following in the C:\WindowsAzure\Logs\Plugins\<ExtensionName>\<Version>\CommandExecution.log file:

```Log sample
Execution Error: '<command>' isn't recognized as an internal or external command, operable program or batch file.
```

**Analysis**

Extensions run under Local System account, so it's very possible that powershell.exe works fine when you RDP into the VM, but fails when run with RunCommand.

**Solution**

- Open a Command Prompt in the VM and execute a command to reproduce the error. The VM Agent uses the Administrator cmd.exe and you may have some preconfigured command to execute every time cmd is started.
- It's also likely that your PATH variable is misconfigured, but this will depend on the command that is having the problem.





### VMAccessAgent is failing with Cannot update Remote Desktop Connection settings for Administrator account. Error: System.Runtime.InteropServices.COMException (0x800706D9): There are no more endpoints available from the endpoint mapper.

You see the following in the extension's status:

```Log sample
Type Microsoft.Compute.VMAccessAgent
Version 2.4.8
Status Provisioning failed
Status level Error
Status message Cannot update Remote Desktop Connection settings for Administrator account. Error: System.Runtime.InteropServices.COMException (0x800706D9): There are no more endpoints available from the endpoint mapper. (Exception from HRESULT: 0x800706D9) at NetFwTypeLib.INetFwRules.GetEnumerator() at 
Microsoft.WindowsAzure.GuestAgent.Plugins.JsonExtensions.VMAccess.RemoteDesktopManager.EnableRemoteDesktopFirewallRules() 
at Microsoft.WindowsAzure.GuestAgent.Plugins.JsonExtensions.VMAccess.RemoteDesktopManager.EnableRemoteDesktop() at
```

**Analysis**

This error can happen when the Windows Firewall service isn't running.

**Solution**

Check if the Windows Firewall service is enabled and running. If it's not, please enable and start it - then try again to run the VMAccessAgent.





### The remote certificate is invalid according to the validation procedure.

You see the following in the WaAppAgent.log

```Log sample
System.Net.WebException: The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel. ---> System.Security.
Authentication.AuthenticationException: The remote certificate is invalid according to the validation procedure.
```

**Analysis**

Your VM is probably missing the Baltimore CyberTrust Root certificate in "Trusted Root Certification Authorities".

**Solution**

Open the certificates console with certmgr.msc, and check if the certificate is there.

Another possible issue is that the certificate chain is broken by a third party SSL Inspection tool, like ZScaler. That kind of tool should be configured to bypass SSL inspection.
