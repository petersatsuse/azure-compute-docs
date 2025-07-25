---
title: Deploy Stateless-only node types in a Service Fabric cluster
description: Learn how to create and deploy stateless node types in Azure Service Fabric cluster.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 11/21/2024
# Customer intent: As a cloud architect, I want to deploy stateless-only node types in a Service Fabric cluster, so that I can optimize scaling and management of resources using the latest capabilities in Azure.
---
# Deploy an Azure Service Fabric cluster with stateless-only node types
Service Fabric node types come with inherent assumption that at some point of time, stateful services might be placed on the nodes. Stateless node types change this assumption for a node type, thus allowing the node type to use other features such as faster scale out operations, support for Automatic OS Upgrades on Bronze durability and scaling out to more than 100 nodes in a single virtual machine scale set.

* Primary node types can't be configured to be stateless
* Stateless node types are only supported with Bronze Durability Levels
* Stateless node types are only supported on Service Fabric Runtime version 7.1.409 or above.


Sample templates are available: [Service Fabric Stateless Node Types template](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/10-VM-2-NodeTypes-Windows-Stateless-Secure)

## Enabling stateless node types in Service Fabric cluster
To set one or more node types as stateless in a cluster resource, set the **isStateless** property to **true**. When deploying a Service Fabric cluster with stateless node types, do remember to have atleast one primary node type in the cluster resource.

* The Service Fabric cluster resource apiVersion should be "2020-12-01-preview" or higher.

```json
{
    "nodeTypes": [
    {
        "name": "[parameters('vmNodeType0Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt0applicationEndPort')]",
            "startPort": "[parameters('nt0applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt0fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt0ephemeralEndPort')]",
            "startPort": "[parameters('nt0ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt0fabricHttpGatewayPort')]",
        "isPrimary": true,
        "isStateless": false, // Primary Node Types cannot be stateless
        "vmInstanceCount": "[parameters('nt0InstanceCount')]"
    },
    {
        "name": "[parameters('vmNodeType1Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt1applicationEndPort')]",
            "startPort": "[parameters('nt1applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt1fabricTcpGatewayPort')]",
        "durabilityLevel": "Bronze",
        "ephemeralPorts": {
            "endPort": "[parameters('nt1ephemeralEndPort')]",
            "startPort": "[parameters('nt1ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt1fabricHttpGatewayPort')]",
        "isPrimary": false,
        "isStateless": true,
        "vmInstanceCount": "[parameters('nt1InstanceCount')]"
    }    
    ],
}
```

## Configuring virtual machine scale set for stateless node types
To enable stateless node types, you should configure the underlying virtual machine scale set resource in the following way:

* The value  **singlePlacementGroup** property, which should be set to **false** if you require to scale to more than 100 VMs.
* The Scale set's **upgradePolicy** should be set to **Rolling**.
* Rolling Upgrade Mode requires Application Health Extension or Health probes configured. For more details on configuring the health probes or the application health extension refer to this [doc](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md#how-does-automatic-os-image-upgrade-work). Configure health probe with the default configuration for Stateless Node types as suggested below. Once the applications are deployed to the node type, Health Probe/Health extension ports can be changed to monitor the actual application health.

>[!NOTE]
> While using AutoScaling with Stateless node types, after scale down operation, node state is not automatically cleaned up. In order to cleanup the NodeState of Down Nodes during AutoScale, using [Service Fabric AutoScale Helper](https://github.com/Azure/service-fabric-autoscale-helper) is advised.

```json
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
          "mode": "Rolling",
          "automaticOSUpgradePolicy": {
            "enableAutomaticOSUpgrade": true
          }
        },
        "platformFaultDomainCount": 5
    },
    "virtualMachineProfile": {
    "extensionProfile": {
    "extensions": [
    {
    "name": "[concat(parameters('vmNodeType1Name'),'_ServiceFabricNode')]",
    "properties": {
        "type": "ServiceFabricNode",
        "autoUpgradeMinorVersion": false,
        "publisher": "Microsoft.Azure.ServiceFabric",
        "settings": {
            "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
            "nodeTypeRef": "[parameters('vmNodeType1Name')]",
            "dataPath": "D:\\\\SvcFab",
            "durabilityLevel": "Bronze",
            "certificate": {
                "thumbprint": "[parameters('certificateThumbprint')]",
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
            "systemLogUploadSettings": {
                "Enabled": true
            },
        },
        "typeHandlerVersion": "1.1"
    }
    },
    {
        "type": "extensions",
        "name": "HealthExtension",
        "properties": {
            "publisher": "Microsoft.ManagedServices",
            "type": "ApplicationHealthWindows",
            "autoUpgradeMinorVersion": true,
            "typeHandlerVersion": "1.0",
            "settings": {
            "protocol": "tcp",
            "port": "19000"
            }
            }
        },
    ]
}
```

## Configuring Stateless node types with multiple Availability Zones
To configure Stateless node type spanning across multiple availability zones follow the documentation [here](./service-fabric-cross-availability-zones.md#1-enable-multiple-availability-zones-in-single-virtual-machine-scale-set), along with the few changes as follows:

* Set **singlePlacementGroup** :  **false** if multiple placement groups are required to be enabled.
* Set  **upgradePolicy** : **Rolling**   and add Application Health Extension/Health Probes as mentioned above.
* Set **platformFaultDomainCount** : **5** for virtual machine scale set.

For reference, look at the [template](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/15-VM-2-NodeTypes-Windows-Stateless-CrossAZ-Secure) for configuring Stateless node types with multiple Availability Zones

## Networking requirements

### Public IP and Load Balancer Resource

To enable scaling to more than 100 VMs on a virtual machine scale set resource, the load balancer and IP resource referenced by that virtual machine scale set must both be using a *Standard* SKU. Creating an IP resource without the SKU property will create a Basic SKU, which doesn't support scaling to more than 100 VMs. A Standard SKU load balancer blocks all traffic from the outside by default; to allow outside traffic, an NSG must be deployed to the subnet.

```json
{
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat('LB','-', parameters('clusterName')]",
    "location": "[parameters('computeLocation')]",
    "sku": {
        "name": "Standard"
    }
}
{
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB','-', parameters('clusterName')]", 
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Network/networkSecurityGroups/', concat('nsg', parameters('subnet0Name')))]"
    ],
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[parameters('addressPrefix')]"
            ]
        },
        "subnets": [
        {
            "name": "[parameters('subnet0Name')]",
            "properties": {
                "addressPrefix": "[parameters('subnet0Prefix')]",
                "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat('nsg', parameters('subnet0Name')))]"
              }
            }
          }
        ]
    },
    "sku": {
        "name": "Standard"
    }
}
```

>[!NOTE]
> It is not possible to do an in-place change of SKU on the public IP and load balancer resources. 

### Virtual machine scale set NAT rules
The load balancer inbound NAT rules should match the NAT pools from the virtual machine scale set. Each virtual machine scale set must have a unique inbound NAT pool.

```json
{
"inboundNatPools": [
    {
        "name": "LoadBalancerBEAddressNatPool0",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "50999",
            "frontendPortRangeStart": "50000",
            "protocol": "tcp"
        }
    },
    {
        "name": "LoadBalancerBEAddressNatPool1",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "51999",
            "frontendPortRangeStart": "51000",
            "protocol": "tcp"
        }
    },
    {
        "name": "LoadBalancerBEAddressNatPool2",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "52999",
            "frontendPortRangeStart": "52000",
            "protocol": "tcp"
        }
    }
    ]
}
```

### Standard SKU Load Balancer outbound rules

Standard Public IP introduce new abilities and different behaviors to outbound connectivity when compared to using the Basic SKU. If you want outbound connectivity when working with Standard SKUs, you must explicitly define it either with Standard Public IP addresses or Standard public Load Balancer. For more information, see [Outbound connections](/azure/load-balancer/load-balancer-outbound-connections) and [Azure Standard Load Balancer](/azure/load-balancer/load-balancer-overview).

>[!NOTE]
> The standard template references an NSG which allows all outbound traffic by default. Inbound traffic is limited to the ports that are required for Service Fabric management operations. The NSG rules can be modified to meet your requirements.

>[!NOTE]
> Any Service Fabric cluster making use of a Standard SKU SLB needs to ensure that each node type has a rule allowing outbound traffic on port 443. This is necessary to complete cluster setup, and any deployment without such a rule will fail.



## Migrate to using Stateless node types in a cluster

For all migration scenarios, a new stateless-only node type needs to be added. Existing node type can't be migrated to be stateless-only.

To migrate a cluster, which was using an IP with a basic SKU, you must first create an entirely new IP resource using the standard SKU. It is not possible to update these resources in-place.

The new LB and IP should be referenced in the new Stateless node types that you would like to use. In the example above, a new virtual machine scale set resources is added to be used for Stateless node types. These virtual machine scale sets reference the newly created LB and IP and are marked as stateless node types in the Service Fabric Cluster Resource.

To begin, you will need to add the new resources to your existing Resource Manager template. These resources include:
* A Public IP Resource using Standard SKU.
* A Load Balancer Resource using Standard SKU.
* A NSG referenced by the subnet in which you deploy your virtual machine scale sets.

Once the resources have finished deploying, you can begin to disable the nodes in the node type that you want to remove from the original cluster.

## Next steps 
* [Reliable Services](service-fabric-reliable-services-introduction.md)
* [Node types and virtual machine scale sets](service-fabric-cluster-nodetypes.md)
