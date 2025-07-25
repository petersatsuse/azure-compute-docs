---
title: Monitor Linux cluster events in Azure Service Fabric 
description: Learn how to monitor Service Fabric Linux cluster events by writing Service Fabric platform events to Syslog.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
ms.custom: linux-related-content
services: service-fabric
ms.date: 07/14/2022
# Customer intent: "As a system administrator managing a Service Fabric Linux cluster, I want to configure platform events to be written to Syslog, so that I can monitor and analyze cluster activity effectively using my preferred logging tools."
---

# Service Fabric Linux cluster events in Syslog

Service Fabric exposes a set of platform events to inform you of important activity in your cluster. The full list of events that are exposed is available [here](service-fabric-diagnostics-event-generation-operational.md). There are various ways through which these events can be consumed. In this article, we discuss how to configure Service Fabric to write these events to Syslog.

## Introduction

In the 6.4 release, the SyslogConsumer was introduced to send the Service Fabric platform events to Syslog for Linux clusters. Once turned on, events automatically flow to Syslog which can be collected and sent by the Log Analytics Agent.

Each Syslog event has four components
* Facility
* Identity
* Message
* Severity

The SyslogConsumer writes all platform events using Facility `Local0`. You can update to any valid facility by changing the config. The Identity used is `ServiceFabric`. The Message field contains the whole event serialized in JSON so that it can be queried and consumed by various tools. 

## Enable SyslogConsumer

To enable the SyslogConsumer, you need to perform an upgrade of your cluster. The `fabricSettings` section needs to be updated with the following code. Note this code just includes sections related to SyslogConsumer

```json
    "fabricSettings": [
        {
            "name": "Diagnostics",
            "parameters": [
            {
                "name": "ConsumerInstances",
                "value": "AzureWinFabCsv, AzureWinFabCrashDump, AzureTableWinFabEtwQueryable, SyslogConsumer"
            }
            ]
        },
        {
            "name": "SyslogConsumer",
            "parameters": [
            {
                "name": "ProducerInstance",
                "value": "WinFabLttProducer"
            },
            {
            "name": "ConsumerType",
            "value": "SyslogConsumer"
            },
            {
                "name": "IsEnabled",
                "value": "true"
            }
            ]
        },
        {
            "name": "Common",
            "parameters": [
            {
                "name": "LinuxStructuredTracesEnabled",
                "value": "true"
            }
            ]
        }
    ],
```

Here are the changes to call out
1. In the Common section, there's a new parameter called `LinuxStructuredTracesEnabled`. **This is required to have Linux events structured and serialized when sent to Syslog.**
2. In the Diagnostics section, a new ConsumerInstance: SyslogConsumer was added. This tells the platform that there's another consumer of the events. 
3. The new section SyslogConsumer needs to have `IsEnabled` as `true`. It's configured to use the Local0 facility automatically. You can override this by adding another parameter.

```json
    {
        "name": "New LogFacility",
        "value": "<Valid Syslog Facility>"
    }
```

## Azure Monitor logs integration
You can read these Syslog events in a monitoring tool such as Azure Monitor logs. You can create a Log Analytics workspace by using the Azure Marketplace using these [instructions](/azure/azure-monitor/logs/quick-create-workspace).

You also need to add the Log Analytics agent to your cluster to collect and send this data to the workspace. This is the same agent used to collect performance counters. 

1. Navigate to the `Advanced Settings` section

    ![Workspace Settings](media/service-fabric-diagnostics-oms-syslog/workspace-settings.png)

2. Select `Data`
3. Select `Syslog`
4. Configure Local0 as the Facility to track. You can add another Facility if you changed it in fabricSettings

    ![Configure Syslog](media/service-fabric-diagnostics-oms-syslog/syslog-configure.png)
5. Head over to the query explorer by clicking `Logs` in the workspace resource's menu to start querying

    ![Workspace logs](media/service-fabric-diagnostics-oms-syslog/workspace-logs.png)
6. You can query against the `Syslog` table looking for `ServiceFabric` as the ProcessName. The following query is an example of how to parse the JSON in the event and display its contents

```kusto
    Syslog | where ProcessName == "ServiceFabric" | extend $payload = parse_json(SyslogMessage) | project $payload
```

![Syslog query](media/service-fabric-diagnostics-oms-syslog/syslog-query.png)

The example above is of a NodeDown event. You can view the full list of events [here](service-fabric-diagnostics-event-generation-operational.md).

## Next steps
* [Deploy the Log Analytics agent](service-fabric-diagnostics-oms-agent.md) onto your nodes to gather performance counters and collect docker stats and logs for your containers
* Get familiarized with the [log search and querying](/azure/azure-monitor/logs/log-query-overview) features offered as part of Azure Monitor logs
* [Use View Designer to create custom views in Azure Monitor logs](/previous-versions/azure/azure-monitor/visualize/view-designer)
* Reference for how to [Azure Monitor logs integration with Syslog](/azure/azure-monitor/agents/data-sources-syslog).
