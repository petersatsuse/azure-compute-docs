---
title: Set environment variables in container instance
description: Learn how to set environment variables in the containers you run in Azure Container Instances
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.date: 08/29/2024
ms.custom: devx-track-azurepowershell, devx-track-azurecli 
ms.devlang: azurecli
# Customer intent: As a container developer, I want to set environment variables when creating container instances, so that I can dynamically configure the application's behavior according to my specific requirements.
---
# Set environment variables in container instances

Setting environment variables in your container instances allows you to provide dynamic configuration of the application or script run by the container. This feature is similar to the `--env` command-line argument to `docker run`. 

To set environment variables in a container, specify them when you create a container instance. This article shows examples of setting environment variables when you start a container with the [Azure CLI](#azure-cli-example), [Azure PowerShell](#azure-powershell-example), and the [Azure portal](#azure-portal-example). 

For example, if you run the Microsoft aci-wordcount container image, you can modify its behavior by specifying the following environment variables:

*NumWords*: The number of words sent to STDOUT.

*MinLength*: The minimum number of characters in a word for it to be counted. A higher number ignores common words like "of" and "the."

>[!IMPORTANT]
>If you need to pass secrets as environment variables, Azure Container Instances supports [secure values](#secure-values) for both Windows and Linux containers.

[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

## Azure CLI example

To see the default output of the aci-wordcount container, run it first with this [az container create][az-container-create] command (no environment variables specified):

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer1 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure
```

To modify the output, start a second container with the `--environment-variables` argument added, specifying values for the *NumWords* and *MinLength* variables. (This example assumes you run the CLI in a Bash shell or Azure Cloud Shell. If you use the Windows Command Prompt, specify the variables with double-quotes, such as `--environment-variables "NumWords"="5" "MinLength"="8"`.)

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer2 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8'
```

Once both containers' state shows as *Terminated* (use [az container show][az-container-show] to check state), display their logs with [az container logs][az-container-logs] to see the output.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer1
az container logs --resource-group myResourceGroup --name mycontainer2
```

The outputs of the containers show how you modified the second container's script behavior by setting environment variables.

**mycontainer1**
```output
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]
```

**mycontainer2**
```output
[('CLAUDIUS', 120),
 ('POLONIUS', 113),
 ('GERTRUDE', 82),
 ('ROSENCRANTZ', 69),
 ('GUILDENSTERN', 54)]
```

## Azure PowerShell example

Setting environment variables in PowerShell is similar to the CLI, but uses the `-EnvironmentVariable` command-line argument.

First, launch the aci-wordcount container in its default configuration with this [New-AzContainerGroup][new-Azcontainergroup] command:

```azurepowershell-interactive
New-AzContainerGroup `
    -ResourceGroupName myResourceGroup `
    -Name mycontainer1 `
    -Image mcr.microsoft.com/azuredocs/aci-wordcount:latest
```

Now run the following [New-AzContainerGroup][new-Azcontainergroup] command. This one specifies the *NumWords* and *MinLength* environment variables after populating an array variable, `envVars`:

```azurepowershell-interactive
$envVars = @(
    New-AzContainerInstanceEnvironmentVariableObject -Name "NumWords" -Value "5"
    New-AzContainerInstanceEnvironmentVariableObject -Name "MinLength" -Value "8"
)

$containerGroup = New-AzContainerGroup -ResourceGroupName "myResourceGroup" `
    -Name "mycontainer2" `
    -Image "mcr.microsoft.com/azuredocs/aci-wordcount:latest" `
    -RestartPolicy "OnFailure" `
    -Container @(
        New-AzContainerInstanceContainer -Name "mycontainer2" `
            -EnvironmentVariable $envVars
    )
```

Once both containers' state is *Terminated* (use [Get-AzContainerInstanceLog][azure-instance-log] to check state), pull their logs with the [Get-AzContainerInstanceLog][azure-instance-log] command.

```azurepowershell-interactive
Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer1
Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer2
```

The output for each container shows how you've modified the script run by the container by setting environment variables.

```console
PS Azure:\> Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer1
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]

Azure:\
PS Azure:\> Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer2
[('CLAUDIUS', 120),
 ('POLONIUS', 113),
 ('GERTRUDE', 82),
 ('ROSENCRANTZ', 69),
 ('GUILDENSTERN', 54)]

Azure:\
```

## Azure portal example

To set environment variables when you start a container in the Azure portal, specify them in the **Advanced** page when you create the container.

1. On the **Advanced** page, set the **Restart policy** to *On failure*
2. Under **Environment variables**, enter `NumWords` with a value of `5` for the first variable, and enter `MinLength` with a value of `8` for the second variable. 
1. Select **Review + create** to verify and then deploy the container.

![Portal page showing environment variable Enable button and text boxes][portal-env-vars-01]

To view the container's logs, under **Settings** select **Containers**, then **Logs**. Similar to the output shown in the previous CLI and PowerShell sections, you can see how the environment variables change the script's behavior. Only five words are displayed, each with a minimum length of eight characters.

![Portal showing container log output][portal-env-vars-02]

## Secure values

Objects with secure values are intended to hold sensitive information like passwords or keys for your application. Using secure values for environment variables is both safer and more flexible than including it in your container's image. Another option is to use secret volumes, described in [Mount a secret volume in Azure Container Instances](container-instances-volume-secret.md).

Environment variables with secure values aren't visible in your container's properties--their values can be accessed only from within the container. For example, container properties viewed in the Azure portal or Azure CLI display only a secure variable's name, not its value.

Set a secure environment variable by specifying the `secureValue` property instead of the regular `value` for the variable's type. The two variables defined in the following YAML demonstrate the two variable types.

### YAML deployment

Create a `secure-env.yaml` file with the following snippet.

```yaml
apiVersion: 2019-12-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Run the following command to deploy the container group with YAML (adjust the resource group name as necessary):

```azurecli-interactive
az container create --resource-group myResourceGroup --file secure-env.yaml
```

### Verify environment variables

Run the [az container show][az-container-show] command to query your container's environment variables:

```azurecli-interactive
az container show --resource-group myResourceGroup --name securetest --query 'containers[].environmentVariables'
```

The JSON response shows both the insecure environment variable's key and value, but only the name of the secure environment variable:

```json
[
  [
    {
      "name": "NOTSECRET",
      "secureValue": null,
      "value": "my-exposed-value"
    },
    {
      "name": "SECRET",
      "secureValue": null,
      "value": null
    }
  ]
]
```

With the [az container exec][az-container-exec] command, which enables executing a command in a running container, you can verify that the secure environment variable is set. Run the following command to start an interactive bash session in the container:

```azurecli-interactive
az container exec --resource-group myResourceGroup --name securetest --exec-command "/bin/sh"
```

Once you open an interactive shell within the container, you can access the `SECRET` variable's value:

```console
root@caas-ef3ee231482549629ac8a40c0d3807fd-3881559887-5374l:/# echo $SECRET
my-secret-value
```

## Next steps

Task-based scenarios, such as batch processing a large dataset with several containers, can benefit from custom environment variables at runtime. For more information about running task-based containers, see [Run containerized tasks with restart policies](container-instances-restart-policy.md).

<!-- IMAGES -->
[portal-env-vars-01]: ./media/container-instances-environment-variables/portal-env-vars-01.png
[portal-env-vars-02]: ./media/container-instances-environment-variables/portal-env-vars-02.png

<!-- LINKS - External -->
[aci-wordcount]: https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount

<!-- LINKS Internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-container-exec]: /cli/azure/container#az_container_exec
[az-container-logs]: /cli/azure/container#az_container_logs
[az-container-show]: /cli/azure/container#az_container_show
[azure-cli-install]: /cli/azure/
[azure-instance-log]: /powershell/module/az.containerinstance/get-azcontainerinstancelog
[azure-powershell-install]: /powershell/azure/install-Az-ps
[new-Azcontainergroup]: /powershell/module/az.containerinstance/new-azcontainergroup
