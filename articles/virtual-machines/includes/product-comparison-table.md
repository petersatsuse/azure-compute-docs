---
ms.service: azure-virtual-machines
ms.topic: include
ms.date: 03/27/2025
author: mattmcinnes
ms.author: mattmcinnes
---

| | Virtual Machines | Virtual Machine Scale Sets | Compute Fleet (preview) |
| --- | --- | --- | --- |
| **Product Description** | Create and manually configure individual VMs.| Load-balanced, automatically scaling VM groups with workload optimization across multiple availability zones. | Provision thousands of mixed-size VMs for performance and high availability. |
| **Product Differences**<br><br>See top product capabilities and limitations side by side| - Deploy and manage each VM individually <br>- Fine-tune custom applications<br> - Assigned to one availability zone at a time<br>| - Up to 2,000 mixed-size VMs in a group <br> - Scale automatically <br> - Fault domains for high availability <br>- Integrate Azure Spot VMs to cut costs <br>- Optimized for stateless or stateful workloads | - Up to 10,000 mixed-size VMs in a group<br>- Fault domains for high availability<br>- Hyper-scale with demand <br>- Maintain capacity with Spot VMs to cut costs<br>- Fleet allocation for optimizing price, capacity, or both
| **Use Case**<br><br>Find the right product for your project | - Small, simple jobs <br>- Specialized single-instance apps <br>- Test configurations and experiment with Azure | - Scaling beyond a single VM <br>- Platform engineering or infrastructure as code <br>- Moving on-premises apps to the cloud <br>- Parallelized [high-performance computing](/azure/architecture/topics/high-performance-computing) workloads <br>- Databases <br>- Batch processing <br> - Maintain performance while controlling costs <br>- Mixed VM sizes for more flexibility | - Large scale highly parallelized workloads or batch jobs <br> - Flexibility with VM sizes - Large scale cost optimization with Azure Spot |
