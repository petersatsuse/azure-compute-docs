---
title: Guidelines for Reliable Collections
description: Guidelines and Recommendations for using Service Fabric Reliable Collections in an Azure Service Fabric application.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 05/17/2023
# Customer intent: "As a developer using Azure Service Fabric, I want to implement Reliable Collections best practices, so that I can ensure data consistency and avoid common pitfalls in my application."
---

# Guidelines and recommendations for Reliable Collections in Azure Service Fabric
This section provides guidelines for using Reliable State Manager and Reliable Collections. The goal is to help users avoid common pitfalls.

## Reliable Collection guidelines

The guidelines are organized as simple recommendations prefixed with the terms *Do*, *Consider*, *Avoid* and *Do not*.

* Do not modify an object of custom type returned by read operations (for example, `TryPeekAsync` or `TryGetValueAsync`). Reliable Collections, just like Concurrent Collections, return a reference to the objects and not a copy.
* Do deep copy the returned object of a custom type before modifying it. Since structs and built-in types are pass-by-value, you do not need to do a deep copy on them unless they contain reference-typed fields or properties that you intend to modify.
* Do not use `TimeSpan.MaxValue` for timeouts. Timeouts should be used to detect deadlocks.
* Do not use a transaction after it has been committed, aborted, or disposed.
* Do not use an enumeration outside of the transaction scope it was created in.
* Do not create a transaction within another transaction's `using` statement because it can cause deadlocks.
* Do not create reliable state with `IReliableStateManager.GetOrAddAsync` and use the reliable state in the same transaction. This results in an InvalidOperationException.
* Do ensure that your `IComparable<TKey>` implementation is correct. The system takes dependency on `IComparable<TKey>` for merging checkpoints and rows.
* Do use Update lock when reading an item with an intention to update it to prevent a certain class of deadlocks.
* Consider keeping number of Reliable Collections per partition to be less than 1000. Prefer Reliable Collections with more items over more Reliable Collections with fewer items.
* Consider keeping your items (for example, TKey + TValue for Reliable Dictionary) below 80 KBytes: smaller the better. This reduces the amount of Large Object Heap usage as well as disk and network IO requirements. Often, it reduces replicating duplicate data when only one small part of the value is being updated. Common way to achieve this in Reliable Dictionary, is to break your rows in to multiple rows.
* Consider using backup and restore functionality to have disaster recovery.
* Avoid mixing single entity operations and multi-entity operations (e.g `GetCountAsync`, `CreateEnumerableAsync`) in the same transaction due to the different isolation levels.
* Do handle InvalidOperationException. User transactions can be aborted by the system for variety of reasons. For example, when the Reliable State Manager is changing its role out of Primary or when a long-running transaction is blocking truncation of the transactional log. In such cases, user may receive InvalidOperationException indicating that their transaction has already been terminated. Assuming, the termination of the transaction was not requested by the user, best way to handle this exception is to dispose the transaction, check if the cancellation token has been signaled (or the role of the replica has been changed), and if not create a new transaction and retry.  
* Do not apply [parallel](/dotnet/standard/parallel-programming/how-to-use-parallel-invoke-to-execute-parallel-operations) 
  or [concurrent](/dotnet/api/system.threading.tasks.task.whenall) operations within a transaction. 
  Only one user thread operation is supported within a transaction. Otherwise, it will cause memory leak and lock issues.
* Consider dispose transaction as soon as possible after commit completes (especially if using ConcurrentQueue).
* Do not perform any blocking code inside a transaction.
* When [string](/dotnet/api/system.string) is used as the key for a reliable dictionary, the sorting order uses [default string comparer CurrentCulture](/dotnet/api/system.string.compare#system-string-compare(system-string-system-string)). Note that the CurrentCulture sorting order is different from [Ordinal string comparer](/dotnet/api/system.stringcomparer.ordinal). 
* Do not dispose or cancel a committing transaction. This is not supported and could crash the host process.
* Do ensure the operation order of different dictionaries stays the same for all concurrent transactions when reading or writing multiple dictionaries to avoid deadlock.

Here are some things to keep in mind:

* The default timeout is 4 seconds for all the Reliable Collection APIs. Most users should use the default timeout.
* The default cancellation token is `CancellationToken.None` in all Reliable Collections APIs.
* The key type parameter (*TKey*) for a Reliable Dictionary must correctly implement `GetHashCode()` and `Equals()`. Keys must be immutable.
* To achieve high availability for the Reliable Collections, each service should have at least a target and minimum replica set size of 3.
* Read operations on the secondary may read versions that are not quorum committed.
  This means that a version of data that is read from a single secondary might be false progressed.
  Reads from Primary are always stable: can never be false progressed.
* Security/Privacy of the data persisted by your application in a reliable collection is your decision and subject to the protections provided by your storage management; I.E. Operating System disk encryption could be used to protect your data at rest.
* `ReliableDictionary` enumeration uses a sorted data structure ordered by key. To make enumeration efficient, commits are added to a temporary hashtable and later moved into the main sorted data structure post checkpoint. Adds/Updates/Deletes have best case runtime of O(1) and worst case runtime of O(log n), in the case of validation checks on the presence of the key. Gets might be O(1) or O(log n) depending on whether you are reading from a recent commit or from an older commit.

## Additional guidelines for volatile Reliable Collections

When deciding to use volatile reliable collections, consider the following:

* ```ReliableDictionary``` does have volatile support
* ```ReliableQueue``` does have volatile support
* ```ReliableConcurrentQueue``` does NOT have volatile support
* Persisted services CANNOT be made volatile. Changing the ```HasPersistedState``` flag to ```false``` requires recreating the entire service from scratch
* Volatile services CANNOT be made persisted. Changing the ```HasPersistedState``` flag to ```true``` requires recreating the entire service from scratch
* ```HasPersistedState``` is a service level config. This means that **ALL** collections will either be persisted or volatile. You cannot mix volatile and persisted collections
* Quorum loss of a volatile partition results in complete data loss
* Backup and restore is NOT available for volatile services

## Next steps
* [Working with Reliable Collections](service-fabric-work-with-reliable-collections.md)
* [Transactions and Locks](service-fabric-reliable-services-reliable-collections-transactions-locks.md)
* Managing Data
  * [Backup and Restore](service-fabric-reliable-services-backup-restore.md)
  * [Notifications](service-fabric-reliable-services-notifications.md)
  * [Serialization and Upgrade](service-fabric-application-upgrade-data-serialization.md)
  * [Reliable State Manager configuration](service-fabric-reliable-services-configuration.md)
* Others
  * [Reliable Services quickstart](service-fabric-reliable-services-quick-start.md)
  * [Developer reference for Reliable Collections](/dotnet/api/microsoft.servicefabric.data.collections#microsoft_servicefabric_data_collections)
