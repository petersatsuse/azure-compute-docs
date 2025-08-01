---
 title: include file
 description: include file
 services: virtual-machines
 author: mattmcinnes
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 05/14/2024
 ms.author: jainan
 ms.reviewer: mattmcinnes
 ms.custom: include file
# Customer intent: As a cloud infrastructure manager, I want to use VM hibernation to pause unused virtual machines, so that I can reduce compute costs and optimize resource allocation for temporary workloads or long boot time applications.
---


Hibernation allows you to pause VMs that aren't being used and save on compute costs. It's an effective cost management feature for scenarios such as:
- Virtual desktops, dev/test servers, and other scenarios where the VMs don't need to run 24/7.
- Systems with long boot times due to memory intensive applications. These applications can be initialized on VMs and hibernated. These “prewarmed” VMs can then be quickly started when needed, with the applications already up and running in the desired state.
