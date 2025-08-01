---
title: Azure Service Fabric Reliable Services lifecycle 
description: Learn about the lifecycle events in an Azure Service Fabric Reliable Services application using Java for stateful and stateless services.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
ms.custom: devx-track-extended-java
services: service-fabric
ms.date: 07/11/2022
# Customer intent: As a Java developer working with Azure Service Fabric, I want to understand the lifecycle events of Reliable Services, so that I can effectively manage stateful and stateless service behaviors during startup, shutdown, and primary swaps.
---

# Reliable Services lifecycle
> [!div class="op_single_selector"]
> * [C# on Windows](service-fabric-reliable-services-lifecycle.md)
> * [Java on Linux](service-fabric-reliable-services-lifecycle-java.md)
>
>

Reliable Services is one of the programming models available in Azure Service Fabric. When learning about the lifecycle of Reliable Services, it's most important to understand the basic lifecycle events. The exact ordering of events depends on configuration details. 

In general, the Reliable Services lifecycle includes the following events:

* During startup:
  * Services are constructed.
  * Services have an opportunity to construct and return zero or more listeners.
  * Any returned listeners are opened, for communication with the service.
  * The service's `runAsync` method is called, so the service can do long-running or background work.
* During shutdown:
  * The cancellation token that was passed to `runAsync` is canceled, and the listeners are closed.
  * The service object itself is destructed.

The order of events in Reliable Services might change slightly depending on whether the reliable service is stateless or stateful. 

Also, for stateful services, you must address the primary swap scenario. During this sequence, the role of primary is transferred to another replica (or comes back) without the service shutting down. 

Finally, you have to think about error or failure conditions.

## Stateless service startup
The lifecycle of a stateless service is fairly straightforward. Here's the order of events:

1. The service is constructed.
2. `StatelessService.createServiceInstanceListeners()` is invoked, and any returned listeners are opened. `CommunicationListener.openAsync()` is called on each listener.
3. Then in parallel:
    - The service's `runAsync` method (`StatelessService.runAsync()`) is called.
    - If present, the service's own `onOpenAsync` method is called. Specifically, `StatelessService.onOpenAsync()` is called. This is an uncommon override, but it is available.

## Stateless service shutdown
When shutting down a stateless service, the same pattern is followed, but in reverse:

1. Any open listeners are closed. `CommunicationListener.closeAsync()` is called on each listener.
2. The cancellation token that was passed to `runAsync()` is canceled. Checking the cancellation token's `isCancelled` property returns `true`, and if called, the token's `throwIfCancellationRequested` method throws a `CancellationException`.
3. When `runAsync()` finishes, the service's `StatelessService.onCloseAsync()` method is called, if it's present. Again, this is not a common override, but it can be used to safely close resources, stop background processing, finish saving external state, or close down existing connections.
4. After `StatelessService.onCloseAsync()` finishes, the service object is destructed.

## Stateful service startup
Stateful services have a pattern that is similar to stateless services, with a few changes.  Here's the order of events for starting a stateful service:

1. The service is constructed.
2. `StatefulServiceBase.onOpenAsync()` is called. This call is not commonly overridden in the service.
3. `StatefulServiceBase.createServiceReplicaListeners()` is invoked.
      - If the service is a primary service, all returned listeners are opened. `CommunicationListener.openAsync()` is called on each listener.
      - If the service is a secondary service, only listeners marked as `listenOnSecondary = true` are opened. Having listeners that are open on secondaries is less common.
4. Then in parallel:
    - If the service is currently a primary, the service's `StatefulServiceBase.runAsync()` method is called.
    - `StatefulServiceBase.onChangeRoleAsync()` is called. This call is not commonly overridden in the service.


  > [!NOTE]
  > For a new secondary replica, `StatefulServiceBase.onChangeRoleAsync()` is called twice. Once after step 2, when it becomes an Idle Secondary and again during step 4, when it becomes an Active Secondary. For more information on replica and instance lifecycle, read [Replica and Instance Lifecycle](service-fabric-concepts-replica-lifecycle.md).

## Stateful service shutdown
Like stateless services, the lifecycle events during shutdown are the same as during startup, but reversed. When a stateful service is being shut down, the following events occur:

1. Any open listeners are closed. `CommunicationListener.closeAsync()` is called on each listener.
2. The cancellation token that was passed to `runAsync()` is canceled. A call to the cancellation token's `isCancelled()` method returns `true`, and if called, the token's `throwIfCancellationRequested()` method throws an `OperationCanceledException`. Service Fabric waits for `runAsync()` to complete.
> [!NOTE]
> Waiting for `runAsync` to finish is necessary only if this replica is a primary replica.

3. After `runAsync()` finishes, the service's `StatefulServiceBase.onCloseAsync()` method is called. This call is an uncommon override, but it is available.
4. After `StatefulServiceBase.onCloseAsync()` finishes, the service object is destructed.

## Stateful service primary swaps
While a stateful service is running, communication listeners are opened and the `runAsync` method is called only for the primary replicas of that stateful services. Secondary replicas are constructed, but see no further calls. While a stateful service is running, the replica that's currently the primary can change. The lifecycle events that a stateful replica can see depends on whether it is the replica being demoted or promoted during the swap.

### For the demoted primary
Service Fabric needs the primary replica that's demoted to stop processing messages and stop any background work. This step is similar to when the service is shut down. One difference is that the service isn't destructed or closed, because it remains as a secondary. The following events occur:

1. Any open listeners are closed. `CommunicationListener.closeAsync()` is called on each listener.
2. The cancellation token that was passed to `runAsync()` is canceled. A check of the cancellation token's `isCancelled()` method returns `true`. If called, the token's `throwIfCancellationRequested()` method throws an `OperationCanceledException`. Service Fabric waits for `runAsync()` to complete.
3. Listeners marked as listenOnSecondary = true are opened.
4. The service's `StatefulServiceBase.onChangeRoleAsync()` is called. This call is not commonly overridden in the service.

### For the promoted secondary
Similarly, Service Fabric needs the secondary replica that's promoted to start listening for messages on the wire, and to start any background tasks that it needs to complete. This process is similar to when the service is created. The difference is that the replica itself already exists. The following events occur:

1. `CommunicationListener.closeAsync()` is called for all the opened listeners (marked with listenOnSecondary = true)
2. All the communication listeners are opened. `CommunicationListener.openAsync()` is called on each listener.
3. Then in parallel:
    - The service's `StatefulServiceBase.runAsync()` method is called.
    - `StatefulServiceBase.onChangeRoleAsync()` is called. This call is not commonly overridden in the service.

  > [!NOTE]
  > `createServiceReplicaListeners` is called only once and is not called again during the replica promotion or demotion process; the same `ServiceReplicaListener` instances are used but new `CommunicationListener` instances are created (by calling the `ServiceReplicaListener.createCommunicationListener` method) after the previous instances are closed.

### Common issues during stateful service shutdown and primary demotion
Service Fabric changes the primary of a stateful service for multiple reasons. The most common reasons are [cluster rebalancing](service-fabric-cluster-resource-manager-balancing.md) and [application upgrade](service-fabric-application-upgrade.md). During these operations, it's important that the service respects the `cancellationToken`. This also applies during normal service shutdown, such as if the service was deleted.

Services that don't handle cancellation cleanly can experience several issues. These operations are slow because Service Fabric waits for the services to stop gracefully. This can ultimately lead to failed upgrades that time out and rollback. Failure to honor the cancellation token also can cause imbalanced clusters. Clusters become unbalanced because nodes get hot. However, the services can't be rebalanced because it takes too long to move them elsewhere. 

Because the services are stateful, it's also likely that the services use [Reliable Collections](service-fabric-reliable-services-reliable-collections.md). In Service Fabric, when a primary is demoted, one of the first things that happens is that write access to the underlying state is revoked. This leads to a second set of issues that might affect the service lifecycle. The collections return exceptions based on the timing and whether the replica is being moved or shut down. It's important to handle these exceptions correctly. 

Exceptions thrown by Service Fabric are either permanent [(`FabricException`)](/java/api/system.fabric.exception) or transient [(`FabricTransientException`)](/java/api/system.fabric.exception.fabrictransientexception). Permanent exceptions should be logged and thrown. Transient exceptions can be retried based on retry logic.

An important part of testing and validating Reliable Services is handling the exceptions that come from using the `ReliableCollections` in conjunction with service lifecycle events. We recommend that you always run your service under load. You should also perform upgrades and [chaos testing](service-fabric-controlled-chaos.md) before deploying to production. These basic steps help ensure that your service is implemented correctly, and that it handles lifecycle events correctly.

## Notes on service lifecycle
* Both the `runAsync()` method and the `createServiceInstanceListeners/createServiceReplicaListeners` calls are optional. A service might have one, both, or neither. For example, if the service does all its work in response to user calls, there's no need for it to implement `runAsync()`. Only the communication listeners and their associated code are necessary.  Similarly, creating and returning communication listeners is optional. The service might have only background work to do, so it only needs to implement `runAsync()`.
* It's valid for a service to complete `runAsync()` successfully and return from it. This isn't considered a failure condition. It represents the background work of the service finishing. For stateful Reliable Services, `runAsync()` would be called again if the service is demoted from primary, and then promoted back to primary.
* If a service exits from `runAsync()` by throwing some unexpected exception, this is a failure. The service object is shut down, and a health error is reported.
* Although there's no time limit on returning from these methods, you immediately lose the ability to write. Therefore, you can't complete any real work. We recommend that you return as quickly as possible upon receiving the cancellation request. If your service doesn't respond to these API calls in a reasonable amount of time, Service Fabric might forcibly terminate your service. Usually, this happens only during application upgrades or when a service is being deleted. This timeout is 15 minutes by default.
* Failures in the `onCloseAsync()` path result in `onAbort()` being called. This call is a last-chance, best-effort opportunity for the service to clean up and release any resources that they have claimed. This is generally called when a permanent fault is detected on the node, or when Service Fabric cannot reliably manage the service instance's lifecycle due to internal failures.
* `OnChangeRoleAsync()` is called when the stateful service replica is changing role, for example to primary or secondary. Primary replicas are given write status (are allowed to create and write to Reliable Collections). Secondary replicas are given read status (can only read from existing Reliable Collections). Most work in a stateful service is performed at the primary replica. Secondary replicas can perform read-only validation, report generation, data mining, or other read-only jobs.

## Next steps
* [Introduction to Reliable Services](service-fabric-reliable-services-introduction.md)
* [Reliable Services quickstart](service-fabric-reliable-services-quick-start-java.md)
