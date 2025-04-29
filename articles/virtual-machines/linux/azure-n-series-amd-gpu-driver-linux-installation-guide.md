---
title: Azure N-series AMD GPU Driver Setup for Linux
description: How to set up AMD Radeon PRO V710 NVv5 Linux Installation Guide.
services: virtual-machines
author: nmagatala-MSFT
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.collection: linux
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 01/27/2025
ms.author: v-nmagatala
ms.reviewer: vikancha
---

# Install AMD GPU drivers on Azure NVv5_V710 Linux VMs

**Applies to:** :heavy_check_mark: Linux VMs

## 1. Installation Guide

### 1.1 Introduction

This document outlines the steps for installing the AMD Linux Driver to harness the capabilities of the AMD Radeon&trade; PRO V710 GPU on an NVv5-V710 GPU Linux instance provided by Microsoft Azure. Subsequent sections provide detailed Linux driver installation instructions for users who wish to perform inference using ROCm on the NVv5-V710 GPU Linux instance.

## 2. Linux Driver Installation

### 2.1 Supported Linux Distros

Confirm the system has a supported Linux version.
To obtain the Linux distribution information, use the following command:
``` bash
$ cat /etc/*release
```
Output is similar to the following example
```bash
DISTRIB_ID=Ubuntu 
DISTRIB_RELEASE=XX 
DISTRIB_CODENAME=jammy 
DISTRIB_DESCRIPTION="Ubuntu" 
PRETTY_NAME="Ubuntu LTS"
```
Confirm that your Linux distribution matches a [supported distribution](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/reference/system-requirements.html#supported-distributions).

### 2.2 Supported Linux Kernel

To check the kernel version of your Linux system, use the below command:
```bash
$ uname -srmv
```
Output is similar to the following example

```bash
Linux 5.XX.0-XX-generic #86-Ubuntu SMP Mon Jul 10 16:07:21 UTC 2023 x86_64
```
Confirm that your kernel version matches the [supported operating systems](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/reference/system-requirements.html#supported-distributions).

## 3. Prerequisites

### 3.1 Setting Permissions for groups

Add yourself to the render and video group using the following command:
```bash
$ sudo usermod -a -G render,video $LOGNAME
```

### 3.2 Kernel headers and development packages

The driver package uses Dynamic Kernel Module Support (DKMS) to build the amdgpu-dkms module for installed kernels. This requires the installation of Linux kernel headers and modules for each kernel. These are installed automatically with the kernel. However, if you use multiple kernel versions or download kernel images without the meta-packages, you might need to install them manually.

```bash
$ sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
```

### 3.3 Verifying GPU Card in Linux&reg;

The output should the GPU card.

```Bash
$ sudo lspci -d 1002:7461
c3:00.0 Display controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 7461
```

>[!NOTE]
> 7461 is the Virtual Function Device ID. This confirms that the Virtual Machine is configured with the AMD Radeon&trade; PRO V710 GPU.

### 3.4 Virtual Machine Update

On an NVv5-V710 GPU Linux instance running Ubuntu 22.04 OS, run the update: 
```bash
$ sudo apt update
```

### 3.5 Exclude AMD GPU Driver

To install the latest AMD Linux driver, it's crucial to exclude the default AMD GPU driver found in Linux OS distributions such as Ubuntu or RHEL. The default AMD GPU driver in Linux OS distributions isn't certified for use with the AMD Radeon&trade; PRO V710 GPU on an NVv5-V710 GPU Linux instance. This Linux driver, on the other hand, is optimized explicitly for Azure NVv5-V710 GPU Linux workloads. Follow the steps below to exclude the driver: 

Open/etc/modprobe.d/exclude.conf file and append the following line:

```bash
exclude amdgpu
```
After updating the exclude.conf file as above, run the following command for the change to take effect after reboot:

```Bash
$ sudo update-initramfs -uk all
```

### 3.6 Reboot

After restarting the Virtual Machine, the default AMD GPU driver in Ubuntu Linux distributions does not load because we excluded it. To confirm that the driver isn't loaded, run the command "lsmod | grep amdgpu" to check if the amdgpu driver is loaded. If there's no output, the driver isn't loaded, and you can proceed. However, if the driver has remained loaded, return to the previous step to double-check that the amdgpu driver has been excluded correctly.

## 4. AMD Driver Installation

### 4.1 Installation

The following steps demonstrate the use of the amdgpu-install script for a single-version driver installation. To install the latest rocm driver, run the following commands on your terminal:

```bash
# URL to the directory listing 
BASE_URL="https://repo.radeon.com/amdgpu-install/latest/ubuntu/jammy/" 

# Fetch the directory listing and find the latest .deb file 
LATEST_DEB=$(wget -qO- $BASE_URL | grep -oP 'amdgpu-install_\d+\.\d+\.\d+-\d+_all\.deb' | head -n 1) 

# Download the latest .deb file 
wget "${BASE_URL}${LATEST_DEB}"

# install the driver installer 
sudo apt install ./${LATEST_DEB}

#install the driver 
sudo amdgpu-install --usecase=rocm
```

>[!NOTE]
> If needed, this can be used to create a script to automate the installation process.

### 4.2 Load AMD GPU driver

```bash
$ sudo modprobe amdgpu
```

>[!NOTE]
> The AMD GPU driver needs to be loaded on every boot.

Review the output of **" dmesg | grep amdgpu "** to confirm that the GPU driver is loaded and initialized successfully.

```bash
$ sudo dmesg | grep amdgpu 
[ 66.177373] [drm] amdgpu kernel modesetting enabled. 
[ 66.177379] [drm] amdgpu version: 6.7.0 
[ 66.177623] amdgpu: Virtual CRAT table created for CPU 
[ 66.177653] amdgpu: Topology: Add CPU node 
[ 66.184259] amdgpu 045b:00:00.0: enabling device (0000 -> 0002) 
[ 66.670226] [drm] add ip block number 5 <amdgpu_vkms> 
[ 66.685726] amdgpu 045b:00:00.0: amdgpu: Fetched VBIOS from VRAM BAR 
[ 66.685733] amdgpu: ATOM BIOS: 113-D7190300-104 
[ 66.689542] amdgpu 045b:00:00.0: amdgpu: CP RS64 enable
```

### 4.2.1 Run AMD-SMI to confirm the driver is loaded successfully 

```bash
$ amd-smi monitor
```
```bash
GPU  POWER  GPU_TEMP  MEM_TEMP  GFX_UTIL  GFX_CLOCK  MEM_UTIL  MEM_CLOCK  ENC_UTIL  ENC_CLOCK  DEC_UTIL  DEC_CLOCK     THROTTLE  SINGLE_ECC  DOUBLE_ECC  PCIE_REPLAY  VRAM_USED  VRAM_TOTAL   PCIE_BW 

  0   11 W     43 °C     58 °C      84 %   1814 MHz       1 %     96 MHz       N/A    812 MHz       N/A    512 MHz  UNTHROTTLED           0           0            0     227 MB    25476 MB  N/A Mb/s
```