---
title: Common PowerShell commands for Azure Virtual Networks
description: Common PowerShell commands to get you started creating a virtual network and its associated resources for VMs.
author: ju-shim
ms.service: azure-virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.date: 07/17/2017
ms.author: jushiman
# Customer intent: As a cloud administrator, I want to utilize PowerShell commands to create and manage virtual networks and their associated resources, so that I can efficiently deploy and configure virtual machines within a network environment.
---
# Common PowerShell commands for Azure Virtual Networks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs 

If you want to create a virtual machine, you need to create a [virtual network](/azure/virtual-network/virtual-networks-overview) or know about an existing virtual network in which the VM can be added. Typically, when you create a VM, you also need to consider creating the resources described in this article.

See [How to install and configure Azure PowerShell](/powershell/azure/) for information about installing the latest version of Azure PowerShell, selecting your subscription, and signing in to your account.

Some variables might be useful for you if running more than one of the commands in this article:

- $location - The location of the network resources. You can use [Get-AzLocation](/powershell/module/az.resources/get-azlocation) to find a [geographical region](https://azure.microsoft.com/regions/) that works for you.
- $myResourceGroup - The name of the resource group where the network resources are located.

## Create network resources

| Task | Command |
| ---- | ------- |
| Create subnet configurations |$subnet1 = [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) -Name "mySubnet1" -AddressPrefix XX.X.X.X/XX<BR>$subnet2 = New-AzVirtualNetworkSubnetConfig -Name "mySubnet2" -AddressPrefix XX.X.X.X/XX<BR><BR>A typical network might have a subnet for an [internet facing load balancer](/azure/load-balancer/load-balancer-overview) and a separate subnet for an [internal load balancer](/azure/load-balancer/load-balancer-overview). |
| Create a virtual network |$vnet = [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup -Location $location -AddressPrefix XX.X.X.X/XX -Subnet $subnet1, $subnet2 |
| Test for a unique domain name |[Test-AzDnsAvailability](/powershell/module/az.network/test-azdnsavailability) -DomainNameLabel "myDNS" -Location $location<BR><BR>You can specify a DNS domain name for a [public IP resource](/azure/virtual-network/ip-services/public-ip-addresses), which creates a mapping for domainname.location.cloudapp.azure.com to the public IP address in the Azure-managed DNS servers. The name can contain only letters, numbers, and hyphens. The first and last character must be a letter or number and the domain name must be unique within its Azure location. If **True** is returned, your proposed name is globally unique. |
| Create a public IP address |$pip = [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) -Name "myPublicIp" -ResourceGroupName $myResourceGroup -DomainNameLabel "myDNS" -Location $location -AllocationMethod Dynamic<BR><BR>The public IP address uses the domain name that you previously tested and is used by the frontend configuration of the load balancer. |
| Create a frontend IP configuration |$frontendIP = [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) -Name "myFrontendIP" -PublicIpAddress $pip<BR><BR>The frontend configuration includes the public IP address that you previously created for incoming network traffic. |
| Create a backend address pool |$beAddressPool = [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) -Name "myBackendAddressPool"<BR><BR>Provides internal addresses for the backend of the load balancer that are accessed through a network interface. |
| Create a probe |$healthProbe = [New-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) -Name "myProbe" -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>Contains health probes used to check availability of virtual machines instances in the backend address pool. |
| Create a load balancing rule |$lbRule = [New-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>Contains rules that assign a public port on the load balancer to a port in the backend address pool. |
| Create an inbound NAT rule |$inboundNATRule = [New-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) -Name "myInboundRule1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>Contains rules mapping a public port on the load balancer to a port for a specific virtual machine in the backend address pool. |
| Create a load balancer |$loadBalancer = [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) -ResourceGroupName $myResourceGroup -Name "myLoadBalancer" -Location $location -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe |
| Create a network interface |$nic1= [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) -ResourceGroupName $myResourceGroup -Name "myNIC" -Location $location -PrivateIpAddress XX.X.X.X -Subnet $subnet2 -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>Create a network interface using the public IP address and virtual network subnet that you previously created. |

## Get information about network resources

| Task | Command |
| ---- | ------- |
| List virtual networks |[Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) -ResourceGroupName $myResourceGroup<BR><BR>Lists all the virtual networks in the resource group. |
| Get information about a virtual network |Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup |
| List subnets in a virtual network |Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup &#124; Select Subnets |
| Get information about a subnet |[Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>Gets information about the subnet in the specified virtual network. The $vnet value represents the object returned by Get-AzVirtualNetwork. |
| List IP addresses |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) -ResourceGroupName $myResourceGroup<BR><BR>Lists the public IP addresses in the resource group. |
| List load balancers |[Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) -ResourceGroupName $myResourceGroup<BR><BR>Lists all the load balancers in the resource group. |
| List network interfaces |[Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) -ResourceGroupName $myResourceGroup<BR><BR>Lists all the network interfaces in the resource group. |
| Get information about a network interface |Get-AzNetworkInterface -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>Gets information about a specific network interface. |
| Get the IP configuration of a network interface |[Get-AzNetworkInterfaceIPConfig](/powershell/module/az.network/get-aznetworkinterfaceipconfig) -Name "myNICIP" -NetworkInterface $nic<BR><BR>Gets information about the IP configuration of the specified network interface. The $nic value represents the object returned by Get-AzNetworkInterface. |

## Manage network resources

| Task | Command |
| ---- | ------- |
| Add a subnet to a virtual network |[Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) -AddressPrefix XX.X.X.X/XX -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>Adds a subnet to an existing virtual network. The $vnet value represents the object returned by Get-AzVirtualNetwork. |
| Delete a virtual network |[Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup<BR><BR>Removes the specified virtual network from the resource group. |
| Delete a network interface |[Remove-AzNetworkInterface](/powershell/module/az.network/remove-aznetworkinterface) -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>Removes the specified network interface from the resource group. |
| Delete a load balancer |[Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer) -Name "myLoadBalancer" -ResourceGroupName $myResourceGroup<BR><BR>Removes the specified load balancer from the resource group. |
| Delete a public IP address |[Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress)-Name "myIPAddress" -ResourceGroupName $myResourceGroup<BR><BR>Removes the specified public IP address from the resource group. |

## Next Steps
Use the network interface that you just created when you [create a VM](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json).
