---
title: Set up your development environment on Linux 
description: Install the runtime and SDK and create a local development cluster on Linux. After completing this setup, you'll be ready to build applications.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
ms.custom: devx-track-dotnet, devx-track-extended-java, linux-related-content
services: service-fabric
ms.date: 07/14/2022

# Maintainer notes: Keep these documents in sync:
# service-fabric-local-linux-cluster-windows-wsl2.md
# Customer intent: As a developer using Linux, I want to set up my development environment for Azure Service Fabric, so that I can deploy and run Service Fabric applications locally.
---

# Prepare your development environment on Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [Mac OS X](service-fabric-get-started-mac.md)

To deploy and run [Azure Service Fabric applications](service-fabric-application-model.md) on your Linux development machine, install the runtime and common SDK. You can also install optional SDKs for Java and .NET Core development. 

The steps in this article assume that you install natively on Linux or use the [Service Fabric OneBox container image](https://hub.docker.com/_/microsoft-service-fabric-onebox), i.e. `mcr.microsoft.com/service-fabric/onebox:u18`.

You can manage Service Fabric entities hosted in the cloud or on-premises with the Azure Service Fabric command-line interface (CLI). For information on how to install the CLI, see [Set up the Service Fabric CLI](./service-fabric-cli.md).


## Prerequisites

- Ensure you are using a supported [Linux version](service-fabric-versions.md#supported-linux-versions-and-support-end-date).

- Install the `apt-transport-https` package:
         
    ```bash
    sudo apt-get install apt-transport-https
    ```

## Installation methods

<!-- markdownlint-disable MD025 -->
<!-- markdownlint-disable MD024 -->

# [Ubuntu](#tab/sdksetupubuntu)

## Update your APT sources
To install the SDK and associated runtime package via the apt-get command-line tool, you must first update your Advanced Packaging Tool (APT) sources.

## Script installation

For convenience, a script is provided to install the Service Fabric runtime and the Service Fabric common SDK along with the [**sfctl** CLI](service-fabric-cli.md). Running the script assumes you agree to the licenses for all the software that is being installed. Alternatively you may run the [Manual installation](#manual-installation) steps in the next section which will present associated licenses as well as the components being installed.

After the script runs successfully, you can skip to [Set up a local cluster](#set-up-a-local-cluster).

```bash
sudo curl -s https://raw.githubusercontent.com/Azure/service-fabric-scripts-and-templates/master/scripts/SetupServiceFabric/SetupServiceFabric.sh | sudo bash
```

## Manual installation
For manual installation of the Service Fabric runtime and common SDK, follow the rest of this guide.

1. Open a terminal.

2. Add the `dotnet` repo to your sources list corresponding to your distribution.

    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    ```

3. Add the new MS Open Tech Gnu Privacy Guard (GnuPG or GPG) key to your APT keyring.

    ```bash
    sudo curl -fsSL https://packages.microsoft.com/keys/msopentech.asc | sudo apt-key add -
    ```

4. Add the official Docker GPG key to your APT keyring.

    ```bash
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

5. Set up the Docker repository.

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

6. Add Azul JDK Key to your APT keyring and set up its repository.

    ```bash
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9
    sudo apt-add-repository "deb http://repos.azul.com/azure-only/zulu/apt stable main"
    ```

7. Refresh your package lists based on the newly added repositories.

    ```bash
    sudo apt-get update
    ```

## Install and set up the Service Fabric SDK for a local cluster

After you update your sources, you can install the SDK. Install the Service Fabric SDK package, confirm the installation, and accept the license agreement.

### Ubuntu

```bash
sudo apt-get install servicefabricsdkcommon
```

> [!TIP]
>   The following commands automate accepting the license for Service Fabric packages:
>   ```bash
>   echo "servicefabric servicefabric/accepted-eula-ga select true" | sudo debconf-set-selections
>   echo "servicefabricsdkcommon servicefabricsdkcommon/accepted-eula-ga select true" | sudo debconf-set-selections
>   ```

# [Red Hat Enterprise Linux 7.4](#tab/sdksetuprhel74)

## Update your Yum repositories
To install the SDK and associated runtime package via the yum command-line tool, you must first update your package sources.

## Manual installation (RHEL)
For manual installation of the Service Fabric runtime and common SDK, follow the rest of this guide.

1. Open a terminal.
2. Download and install Extra Packages for Enterprise Linux (EPEL).

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum install epel-release-latest-7.noarch.rpm
    ```
3. Add the EfficiOS RHEL7 package repository to your system.

    ```bash
    sudo wget -P /etc/yum.repos.d/ https://packages.efficios.com/repo.files/EfficiOS-RHEL7-x86-64.repo
    ```

4. Import the EfficiOS package signing key to the local GPG keyring.

    ```bash
    sudo rpmkeys --import https://packages.efficios.com/rhel/repo.key
    ```

5. Add the Microsoft RHEL repository to your system.

    ```bash
    curl https://packages.microsoft.com/config/rhel/7.4/prod.repo > ./microsoft-prod.repo
    sudo cp ./microsoft-prod.repo /etc/yum.repos.d/
    ```

## Install and set up the Service Fabric SDK for a local cluster (RHEL)

After you update your sources, you can install the SDK. Install the Service Fabric SDK package, confirm the installation, and accept the license agreement.

```bash
sudo yum install servicefabricsdkcommon
```

---

## Set up a local cluster
1. Start a local Service Fabric cluster for development.

# [Container-based local cluster](#tab/localclusteroneboxcontainer)

Start a container-based [Service Fabric Onebox](https://hub.docker.com/_/microsoft-service-fabric-onebox) cluster.

1. Install Moby to be able to deploy Docker containers.
    ```bash
    sudo apt-get install moby-engine moby-cli -y
    ```
2. Update the Docker daemon configuration on your host with the following settings and restart the Docker daemon. Details: [Enable IPv6 support](https://docs.docker.com/config/daemon/ipv6/)

    ```json
    {
        "ipv6": true,
        "fixed-cidr-v6": "fd00::/64"
    }
    ```

3. Start the cluster.<br/>
    <b>Ubuntu 20.04 LTS:</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u20
    ```

    <b>Ubuntu 18.04 LTS:</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u18
    ```

    >[!TIP]
    > By default, this will pull the image with the latest version of Service Fabric. For particular revisions, please visit the [Docker Hub](https://hub.docker.com/_/microsoft-service-fabric-onebox) page.

# [Local cluster](#tab/localcluster)

After installing the SDK using steps above, start a local cluster.

1. Run the cluster setup script.

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

---

2. Open a web browser and go to **Service Fabric Explorer** (`http://localhost:19080/Explorer`). When the cluster starts, you see the Service Fabric Explorer dashboard. It might take several minutes for the cluster to be completely set up. If your browser fails to open the URL or if Service Fabric Explorer doesn't show that the system is ready, wait a few minutes and try again.

    ![Service Fabric Explorer on Linux][sfx-linux]

    Now you can deploy prebuilt Service Fabric application packages or new ones based on guest containers or guest executables. To build new services by using the Java or .NET Core SDKs, follow the optional setup steps that are provided in the next sections.


> [!NOTE]
> Standalone clusters aren't supported in Linux.


> [!TIP]
> If you have an SSD disk available, we recommend to pass an SSD folder path by using `--clusterdataroot` with devclustersetup.sh for superior performance.

## Set up the Service Fabric CLI

The [Service Fabric CLI](service-fabric-cli.md) has commands for interacting with Service Fabric entities,
including clusters and applications. To install the CLI, follow the instructions at [Service Fabric CLI](service-fabric-cli.md).


## Set up Yeoman generators for containers and guest executables
Service Fabric provides scaffolding tools that help you create Service Fabric applications from a terminal by using Yeoman template generators. Follow these steps to set up the Service Fabric Yeoman template generators:

1. Install Node.js and npm on your machine.

    ```bash
    sudo add-apt-repository "deb https://deb.nodesource.com/node_8.x $(lsb_release -s -c) main"
    sudo apt-get update
    sudo apt-get install nodejs
    ```
2. Install the [Yeoman](https://yeoman.io/) template generator from npm on your machine.

    ```bash
    sudo npm install -g yo
    ```
3. Install the Service Fabric Yeo container generator and guest executable generator from npm.

    ```bash
    sudo npm install -g generator-azuresfcontainer  # for Service Fabric container application
    sudo npm install -g generator-azuresfguest      # for Service Fabric guest executable application
    ```

After you install the generators, create guest executable or container services by running `yo azuresfguest` or `yo azuresfcontainer`, respectively.

## Set up .NET Core 3.1 development

Install the [.NET Core 3.1 SDK for Ubuntu](/dotnet/core/install/linux-ubuntu) to start [creating C# Service Fabric applications](service-fabric-create-your-first-linux-application-with-csharp.md). Packages for .NET Core Service Fabric applications are hosted on NuGet.org.

## Set up Java development

To build Service Fabric services using Java, install Gradle to run build tasks. Run the below command to install Gradle. The Service Fabric Java libraries are pulled from Maven.


* Ubuntu

    ```bash
    curl -s https://get.sdkman.io | bash
    sdk install gradle 5.1
    ```

* Red Hat Enterprise Linux 7.4 (Service Fabric preview support)

  ```bash
  sudo yum install java-1.8.0-openjdk-devel
  curl -s https://get.sdkman.io | bash
  sdk install gradle
  ```

You also need to install the Service Fabric Yeo generator for Java executables. Make sure you have [Yeoman installed](#set-up-yeoman-generators-for-containers-and-guest-executables), and then run the following command:

  ```bash
  npm install -g generator-azuresfjava
  ```
 
## Install the Eclipse plug-in (optional)

You can install the Eclipse plug-in for Service Fabric from within the Eclipse IDE for Java Developers or Java EE Developers. You can use Eclipse to create Service Fabric guest executable applications and container applications in addition to Service Fabric Java applications.

> [!IMPORTANT]
> The  Service Fabric plug-in requires Eclipse Neon or a later version. See the instructions that follow this note for how to check your version of Eclipse. If you have an earlier version of Eclipse installed, you can download more recent versions from the [Eclipse site](https://www.eclipse.org). We recommend that you do not install on top of (overwrite) an existing installation of Eclipse. Either remove it before running the installer, or install the newer version in a different directory.
> 
> On Ubuntu, we recommend installing directly from the Eclipse site rather than using a package installer (`apt` or `apt-get`). Doing so ensures that you get the most current version of Eclipse. You can install the Eclipse IDE for Java Developers or for Java EE Developers.

1. In Eclipse, make sure that you have installed Eclipse Neon or later and Buildship version 2.2.1 or later. Check the versions of installed components by selecting **Help** > **About Eclipse** > **Installation Details**. You can update Buildship by using the instructions at [Eclipse Buildship: Eclipse Plug-ins for Gradle][buildship-update].

2. To install the Service Fabric plug-in, select **Help** > **Install New Software**.

3. In the **Work with** box, enter **https:\//dl.microsoft.com/eclipse**.

4. Select **Add**.

    ![Available Software page][sf-eclipse-plugin]

5. Select the **ServiceFabric** plug-in, and then select **Next**.

6. Perform the installation steps. Then accept the end-user license agreement.

If you already have the Service Fabric Eclipse plug-in installed, make sure that you have the latest version. Check by selecting **Help** > **About Eclipse** > **Installation Details**. Then search for Service Fabric in the list of installed plug-ins. Select **Update** if a newer version is available.

For more information, see [Service Fabric plug-in for Eclipse Java application development](service-fabric-get-started-eclipse.md).

## Update the SDK and runtime

To update to the latest version of the SDK and runtime, run the following commands.

```bash
sudo apt-get update
sudo apt-get install servicefabric servicefabricsdkcommon
```
To update the Java SDK binaries from Maven, you need to update the version details of the corresponding binary in the ``build.gradle`` file to point to the latest version. To know exactly where you need to update the version, refer to any ``build.gradle`` file in the [Service Fabric getting-started samples](https://github.com/Azure-Samples/service-fabric-java-getting-started).

> [!NOTE]
> Updating the packages might cause your local development cluster to stop running. Restart your local cluster after an upgrade by following the instructions in this article.

## Remove the SDK
To remove the Service Fabric SDKs, run the following commands.

* Ubuntu

    ```bash
    sudo apt-get remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    sudo apt-get install -f
    ```


* Red Hat Enterprise Linux 7.4 (Service Fabric preview support)

    ```bash
    sudo yum remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    ```

## Next steps

* [Create and deploy your first Service Fabric Java application on Linux by using Yeoman](service-fabric-create-your-first-linux-application-with-java.md)
* [Create and deploy your first Service Fabric Java application on Linux by using Service Fabric Plugin for Eclipse](service-fabric-get-started-eclipse.md)
* [Create your first C# application on Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
* [Prepare your development environment on OSX](service-fabric-get-started-mac.md)
* [Prepare a Linux development environment on Windows](service-fabric-local-linux-cluster-windows.md)
* [Manage your applications by using the Service Fabric CLI](service-fabric-application-lifecycle-sfctl.md)
* [Service Fabric Windows and Linux differences](service-fabric-linux-windows-differences.md)
* [Get started with Service Fabric CLI](service-fabric-cli.md)

<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png
