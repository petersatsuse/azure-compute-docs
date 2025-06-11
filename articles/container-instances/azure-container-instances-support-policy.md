---
title: Support Policy for Azure Container Instances
description: Outline of the support policies on Azure Container Instances
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.topic: conceptual
ms.date: 04/16/2025
---
# Support policy for Azure Container Instances

This article describes technical support policies and limitations for **Azure Container Instances (ACI)**.  
It also outlines how Microsoft manages the container runtime and infrastructure, clarifies customer responsibilities, and defines the support boundaries for networking, third-party software, and security or patch management in a serverless container environment.

## Azure Container Instances (ACI) Overview

The following is a summary of the ACI service.

- Platform-as-a-Service (PaaS): ACI abstracts away infrastructure management. You only provide the container image and resource definitions—ACI handles provisioning, scaling, and securing the runtime environment.

- No Host-Level Responsibility: There’s no shared responsibility model for the host. You don’t need to patch operating systems, manage VMs, or configure load balancers.

- Ideal Use Cases: ACI is best suited for lightweight, event-driven, or short-lived workloads that benefit from fast startup and minimal operational overhead.

- Limited Customization by Design: While you can configure container group properties (e.g., CPU, memory, networking), you cannot modify the host OS or runtime. This ensures consistency, security, and high availability.

Not a Full Orchestrator: ACI doesn’t offer the orchestration features of AKS (Azure Kubernetes Service), but it excels at rapid, scalable container deployment without the complexity of managing clusters.

## Managed features in ACI

Azure Container Instances (ACI) provide a simplified, serverless environment for running containers without the need to manage any underlying virtual machines or orchestrators. Unlike traditional IaaS-based solutions where users configure compute, networking, and storage resources manually, ACI offers a streamlined experience with minimal infrastructure management.

With ACI, you can deploy containers in seconds using a fully managed platform. Microsoft handles the container runtime and compute environment, abstracting away the complexities of VM provisioning, scaling infrastructure, or configuring Kubernetes.

Microsoft manages and monitors the following components as part of the ACI platform:

- Container runtime (such as Docker-compatible runtimes)
- Compute and networking infrastructure necessary to run your containers
- Storage integration (when attached) such as Azure File shares
- Container scheduling and isolation to ensure resource fairness and security
- DNS resolution and automatic public IP assignment for container groups
- Security patches and updates for the underlying OS and runtime

## Shared responsibility
When you deploy containers using Azure Container Instances (ACI), Microsoft handles the underlying infrastructure—including compute, networking, and the operating system. Your responsibility is limited to managing the container image, the application inside it, and the data it processes.

Because ACI abstracts away the virtual machine layer, Microsoft Support cannot access your container’s file system, logs, or runtime data unless you explicitly configure diagnostics (e.g., sending logs to Azure Monitor or a storage account). You retain full control over the container image and its behavior.

Since you don’t manage the host environment, simulating host-level changes (as you might in a VM or Kubernetes cluster) is not supported. Instead, all configuration should be done within the container image or through supported environment variables and ACI resource definitions.

While you can apply custom tags and metadata to the ACI resource, modifying system-managed settings or trying to bypass platform constraints may lead to unsupported or unstable states.
## ACI support coverage

### Supported scenarios

Microsoft provides technical support for the following Azure Container Instances scenarios:

- **Container Deployment and Lifecycle Management**  
  Microsoft supports issues related to the deployment, startup, and shutdown of containers using ACI. This includes troubleshooting container creation failures, runtime crashes, or container restart behavior based on your configuration (e.g., restart policies and resource limits).

- **Connectivity to Azure Services**  
  ACI supports VNET integration, allowing containers to securely communicate with other Azure services such as Azure Storage, Key Vault, Azure SQL, and custom applications. Microsoft provides support for connectivity issues within these VNET-enabled deployments, including DNS resolution and access to private endpoints.

- **Resource Configuration and Quotas**  
  Support is available for resource allocation issues such as CPU, memory, GPU usage, and container group quotas. Microsoft ensures the underlying infrastructure is provisioned according to defined container group limits and that scaling behaviors align with service capabilities. Microsoft has discretion over whether or not they provide quota for subscriptions.

- **Networking and IP Management**  
  When containers are deployed into a VNET, Microsoft supports common networking concerns such as IP address assignment, NAT, outbound internet access, DNS issues, and network isolation. Both public and private IP scenarios are supported, as well as basic port exposure configurations.

- **Platform Reliability and Uptime**  
  Microsoft manages the underlying infrastructure that runs ACI containers, including host OS, security patches, and uptime of the container runtime platform. Any outages or infrastructure issues impacting container availability are covered under Microsoft’s platform SLAs.

- **Logging and Monitoring Integration**  
  Microsoft supports integration with Azure Monitor, Log Analytics, and Application Insights to help track container performance, resource usage, and failures. Support includes troubleshooting for diagnostics configuration and ingestion of metrics/logs.

### Unsupported scenarios

> [!NOTE]
> Microsoft may offer best-effort guidance for issues related to integration points with Azure-native services (e.g., connecting a Helm-deployed app to Azure Files, Azure Firewall, Application Gateway, or Private Endpoints).

Microsoft doesn't provide technical support for the following scenarios:

- **Custom application logic or container internals**  
  Microsoft Support does not provide guidance on debugging, optimizing, or modifying the internal code, runtime behavior, or file system structure of your container images or application workloads.

- **Third-party open-source or commercial tools inside containers**  
  Troubleshooting the behavior of tools like Helm, Istio, Envoy, or any other software not managed by Microsoft and running inside ACI containers is outside of support scope. This includes frameworks installed as part of your container image or via runtime scripts.

- **Non-Microsoft networking solutions or custom network topologies**  
  While ACI supports VNET integration, configuring or debugging third-party VPNs, SD-WAN appliances, custom DNS servers, firewalls, or other advanced network solutions is not supported.

- **Ingress and proxy configurations within containers**  
  Microsoft does not support troubleshooting self-managed ingress controllers or reverse proxies (e.g., NGINX, Traefik) deployed inside ACI containers.

- **Certificate issuance and management for workloads**  
  Microsoft does not manage TLS/SSL certificates for your applications running in ACI.

- **Proactive or stand-by operational support**  
  ACI support is reactive only. For proactive help, explore Azure Event Management services.

- **Custom startup scripts or automation logic in containers**  
  Debugging startup scripts, CI/CD commands, or entrypoints is not within support scope.

- **Security scanning or vulnerability management of your container images**  
  You are responsible for scanning and patching container images. Microsoft provides vulnerability support only for Microsoft-maintained base images.

- **Custom code samples or development requests**  
  Microsoft does not offer application-specific code, but may provide basic usage examples.

## Customer responsibilities in ACI

As a user of ACI, your responsibilities include:

- Providing secure and well-maintained container images
- Monitoring and logging your application workloads
- Defining correct container settings (CPU/memory, ports, env variables, volumes)
- Managing the container lifecycle and keeping images updated

## Customization limitations

You **cannot**:

- SSH into the infrastructure
- Use DaemonSets, privileged containers, or init containers
- Modify sysctl settings or host OS
- Apply VM extensions or access host-level networking

> [!IMPORTANT]
> All customization must happen inside the container image or with environment variables and startup commands. Host-level changes are not supported.

## Security and patch management
Microsoft handles:

- Host OS and container runtime patching
- Security updates for Microsoft-managed images

You handle:

- Updating your container images
- Scanning and remediating vulnerabilities in your code and dependencies

## Platform access and maintenance

ACI is fully managed. You:

- Cannot access or modify the host infrastructure
- Must define container configuration at deployment via CLI, ARM, Bicep, or Portal
- Cannot simulate IaaS-level access

## Network ports, access, and NSGs

### Custom network configurations

ACI supports both public and VNET-integrated
