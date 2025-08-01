---
title: Architectures for Oracle database on Azure Virtual Machines
description: Learn about architectures for Oracle database on Azure Virtual Machines.
author: jessiehaessler
ms.service: oracle-on-azure
ms.collection: linux
ms.topic: concept-article
ms.date: 10/18/2024
ms.author: jhaessler
# Customer intent: As a cloud architect, I want to design a high availability and disaster recovery solution for an Oracle database on Azure, so that I can ensure it meets my organization's performance and resiliency requirements.
---
# Architectures for Oracle database on Azure Virtual Machines

**Applies to:** :heavy_check_mark: Linux VMs

This article includes information on deploying a highly available Oracle database on Azure. In addition, this guide dives into disaster recovery considerations. These architectures have been created based on customer deployments. This guide only applies to Oracle Database Enterprise Edition.

If you're interested in learning more about maximizing the performance of your Oracle database, see [Design and implement an Oracle database in Azure](oracle-design.md).

## Prerequisites

- An understanding of the different concepts of Azure such as [availability zones](/azure/reliability/availability-zones-overview)
- Oracle Database Enterprise Edition 12c or later
- Awareness of the licensing implications when using the solutions in this article
- Defined RPO and RTO Requirements

## High availability for Oracle databases

Achieving high availability in the cloud is an important part of every organization's planning and design. Azure offers [availability zones](/azure/reliability/availability-zones-overview) and *availability sets* to be used in regions where availability zones are unavailable. For more information about how to design for the cloud, see [Availability options for Azure Virtual Machines](../../availability.md).

In addition to cloud-native tools and offerings, Oracle provides solutions for high availability that can be set up on Azure:

- [Oracle Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/18/sbydb/introduction-to-oracle-data-guard-concepts.html#GUID-5E73667D-4A56-445E-911F-1E99092DD8D7)
- [Data Guard with FSFO](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dgbkr/index.html)
- [Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/sharding-overview.html)
- [GoldenGate](https://www.oracle.com/middleware/technologies/goldengate.html)

This guide covers reference architectures for each of these solutions.

When you migrate or create applications for the cloud, we recommend using cloud-native patterns such as [retry pattern](/azure/architecture/patterns/retry) and [circuit breaker pattern](/azure/architecture/patterns/circuit-breaker). For other patterns that could help make your application more resilient, see [Cloud Design Patterns guide](/azure/architecture/patterns/).

### Oracle RAC in the cloud

Oracle Real Application Cluster (RAC) is a solution by Oracle to help customers achieve high throughputs by having many instances accessing one database storage. This pattern is a shared-all architecture. While Oracle RAC can be used for high availability on-premises, Oracle RAC alone can't be used for high availability in the cloud. Oracle RAC only protects against instance level failures and not against rack-level or datacenter-level failures. For this reason, Oracle recommends using Oracle Data Guard with your database, whether single instance or RAC, for high availability.

Customers generally require a high SLA to run mission critical applications. Oracle doesn't currently certify or support Oracle RAC on Azure. However, Azure offers features such as availability zones and planned maintenance windows to help protect against instance-level failures. In addition to these offerings, you can use Oracle Data Guard, Oracle GoldenGate, and Oracle Sharding for high performance and resiliency. These technologies can help protect your databases from rack-level, datacenter-level, and geo-political failures.

When you run Oracle Databases on multiple [availability zones](/azure/reliability/availability-zones-overview) with Oracle Data Guard or GoldenGate, you can get an uptime SLA of 99.99%. In Azure regions where availability zones aren't yet present, you can use [availability sets](../../availability-set-overview.md) and achieve an uptime SLA of 99.95%.

> [!NOTE]
> You can have a uptime target that is much higher than the uptime SLA provided by Microsoft.

## Disaster recovery for Oracle databases

When hosting your mission-critical applications in the cloud, it's important to design for high availability and disaster recovery.

For Oracle Database Enterprise Edition, Oracle Data Guard is a useful feature for disaster recovery. You can set up a standby database instance in a [paired Azure region](/azure/reliability/cross-region-replication-azure) and set up Data Guard failover for disaster recovery. For zero data loss, we recommend that you deploy an Oracle Data Guard Far Sync instance in addition to Active Data Guard.

For long distance data replication, it is recommended to leverage Far Sync. Far Sync is an Oracle Active Data Guard Feature. 

> [!NOTE]
> If you enable Far Sync, an Active Data Guard license is needed. Contact your Oracle representative to discuss the licensing implications.

Oracle Far Sync addresses a long distance between primary database and secondary database by introducing an intermediate server known as a Far Sync Instance. This server receives redo data from the primary database and then forwards it to the standby database. Thereby the Far Sync instance is placed closer to the primary to reduce the time of communication. The Far Sync server then transfers the redo data asynchronously to the secondary database.

> [!NOTE]
> When you use Oracle Standard Edition databases, there are ISV solutions that allow you to set up high availability and disaster recovery, such as DBVisit Standby or [Tessell](https://www.tessell.com/azure/oracle).

## Reference architectures

### Oracle Data Guard

Oracle Data Guard ensures high availability, data protection, and disaster recovery for enterprise data. Data Guard maintains standby databases as transactionally consistent copies of the primary database. Depending on the distance between the primary and secondary databases and the application tolerance for latency, you can set up synchronous or asynchronous replication. 

#### Oracle Data Guard with Fast-Start Failover

Data Guard can be deployed using Fast Start Failover (FSFO). Fast-Start-Failover is a feature provided within the Data Guard Broker Configuration. This feature allows you to [automatically failover](https://docs.oracle.com/en/database/oracle/oracle-database/21/dgbkr/using-data-guard-broker-to-manage-switchovers-failovers.html#GUID-7F6C5802-E4AF-4680-91F6-AD380679A555) in case of failure. 
The default time customers use are 30 seconds, but can be adjusted upon your requirements.
This so called OperationTimeout is part of the [Data Guard properties](https://docs.oracle.com/en/database/oracle/oracle-database/19/dgbkr/oracle-data-guard-broker-properties.html#GUID-C256CDA1-B9D0-432A-95D5-2123DCFCCDCA) which you define during your deployment. 

#### How does Data Guard work with this property

Data Guards task is to monitor continuously the health and the status of the primary and the secondary Database. As soon as you enable Fast-Start-Failover (FSFO), the observer process is triggered and reviews the health status at a regular interval to ensure high availability at any given time. 

Now, if the primary Database becomes unavailable, the observer and Data Guard Broker detect this interruption. Thereby, the OperationTimeout parameter of 30 seconds specifies how long the Broker should wait for a response from the primary database before taking any further action. 

Which then results in if the primary does not respond within this 30-second window, the Observer assumes the primary is inaccessible and begins the failover process.

The Broker immediately promotes the standby database to the primary status, switching the roles and ensuring that the application can quickly resume from the standby. 

During this time, the Broker also ensures that transactions are up-to-date on the standby. With the mode that you configure, that can be Maximum Availability or Maximum Protection, a synchronous replication would provide minimal to zero data loss.
The Oracle databases are placed in multiple availability zones for high availability. Each zone is made up of one or more data centers equipped with independent power, cooling, and networking. To ensure resiliency, a minimum of three separate zones are set up in all enabled regions. The physical separation of availability zones within a region protects the data from data center failures. Additionally, two FSFO observers are set up across two availability zones to initiate the failover to the secondary database in case of failure.
After the failover happened and your previous primary database is available again, it can be reinstated. The Data Guard Broker facilitates this process.

Ultimately, if the primary database is unavailable because of a planned or an unplanned outage, Data Guard switches or fails over to your standby database.

This feature can provide additional resiliency by setting up the observer on a separate virtual machine. Thereby you deploy the observer on a lightweight VM. This approach allows for high availability and resiliency.

With Oracle Database version 12.2 and above, it's also possible to configure multiple observers with a single Oracle Data Guard broker configuration. It provides extra availability, in case one observer and the secondary database experience downtime.
For more information about Data Guard Broker and its advantages, see [Oracle Data Guard Broker Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/dgbkr/oracle-data-guard-broker-concepts.html#GUID-723E5B73-A350-4B2E-AF3C-5EA4EFC8396).

The following diagram shows an Oracle Data Guard installation without Far Sync with a recovery time of lesser than 5 minutes. 

:::image type="content" source="./media/oracle-reference-architecture/oracle-dg-architecture.svg" alt-text="Diagram that shows an Oracle Data Guard Architecture.":::

The Oracle databases are placed in multiple availability zones for high availability. Each zone is made up of one or more data centers equipped with independent power, cooling, and networking. To ensure resiliency, a minimum of three separate zones are set up in all enabled regions. The physical separation of availability zones within a region protects the data from data center failures. Additionally, two FSFO observers are set up across two availability zones to initiate the failover to the secondary database in case of failure.

> [!NOTE]
> When you plan for a symmetrical Data Guard deployment, please note that you will need one more observer in availability zone three.

Additionally, we highly recommend deploying the Oracle Enterprise Manager to keep having an overview of the database layer.
Azure Monitor is recommended to be deployed with the following metrics:
Monitor the disks:
- OS Disk IOPS Consumed Percentage  
- Data Disk IOPS Consumed Percentage 
- Data Disk Read Bytes/Sec  
- Data Disk Writes Bytes/Sec  
- Disk Queue Depth
- Disk Bandwidth in % per Lun

In addition to the above we highly recommend enabling the VM insights also.

The Virtual Machine is chosen based on your AWR assessment. Please review [Oracle Capacity Planning](/azure/cloud-adoption-framework/scenarios/oracle-iaas/oracle-capacity-planning) for further read.
We highly recommend making use of [constrained core vCPUs](../../../virtual-machines/constrained-vcpu.md) to save on licensing costs and maximize performance. 

The choice of disk type is dependent on the output of your AWR assessment. 

As mentioned above Far Sync is a capability predominantly used in scenarios where you replicate between regions overcoming long distances. Referring to this scenario, Oracle Active Data Guard Far Sync provides zero data loss protection capability for Oracle Databases. Oracle Far Sync instance needs to be installed on a separate VM. 
> Premium SSD v2 are not supported for files referring to the operating system. For further information, please visit [Deploy Premium SSD v2](/azure/virtual-machines/disks-deploy-premium-v2?tabs=azure-cli#limitations).

As Backup destination Azure Premium Files is used. This solution is the most performant. You can also use Azure Blob Storage as Backup destination. Always make sure to test which option suits you best. Please also visit [Oracle Database Backup Strategies](/azure/virtual-machines/workloads/oracle/oracle-database-backup-strategies).

#### Oracle Data Guard Far Sync

As mentioned above Far Sync is a capability predominantly used in scenarios where you replicate between regions overcoming long distances. Referring to this scenario, Oracle Far Sync provides zero data loss protection capability for Oracle Databases. The Oracle Far Sync instance needs to be installed on a separate VM. 

For zero data loss protection, there must be synchronous communication between your primary database and the Far Sync instance. The Far Sync instance receives redo from the primary in a synchronous manner and forwards it immediately to all the standby databases in an asynchronous manner. This setup also reduces the overhead on the primary database, because it only has to send the redo to the Far Sync instance rather than all the standby databases. If a Far Sync instance fails, Active Data Guard automatically uses asynchronous transport to the secondary database from the primary database to maintain near-zero data loss protection. For added resiliency, customers might deploy multiple Far Sync instances per each database instance, including primary and secondaries.

The following diagram is an architecture that uses Oracle Active Data Guard FSFO and Far Sync to achieve high availability and disaster recovery: 

:::image type="content" source="./media/oracle-reference-architecture/architecture-far-sync.png" alt-text="Diagram that shows Oracle Database using Far Sync in an Active Data Guard configuration across regions." lightbox="./media/oracle-reference-architecture/architecture-far-sync.png":::

> [!NOTE]
> When you plan for a symmetrical Far Sync deployment, please note that you will need one more Far Sync instance in the second region.

### Oracle GoldenGate

GoldenGate enables the exchange and manipulation of data at the transaction level among multiple, heterogeneous platforms across the enterprise. It moves committed transactions with transaction integrity and minimal overhead on your existing infrastructure. Its modular architecture gives you the flexibility to extract and replicate selected data records, transactional changes, and changes to data definition language (DDL) across various topologies.

Oracle GoldenGate allows you to configure your database for high availability by providing bidirectional replication. This approach allows you to set up a *multi-master* or *active-active configuration* that requires application-level awareness. The following diagram is a recommended architecture for Oracle GoldenGate active-active setup on Azure. In the following architecture, the Oracle database has been configured using a hyperthreaded [memory optimized virtual machine](../../sizes-memory.md) with [constrained core vCPUs](../../../virtual-machines/constrained-vcpu.md) to save on licensing costs and maximize performance. The architecture uses multiple premium or ultra disks (managed disks) for performance and availability.

:::image type="content" source="./media/oracle-reference-architecture/oracledb_gg_az.png" alt-text="Diagram that shows Oracle Database using availability zones with Data Guard Broker - FSFO." lightbox="./media/oracle-reference-architecture/oracledb_gg_az.png":::

> [!NOTE]
> A similar architecture can be set up using availability sets in regions where availability zones aren't currently available.

Oracle GoldenGate has processes such as *Extract*, *Pump*, and *Replicat* that help you asynchronously replicate your data from one Oracle database server to another. These processes allow you to set up a bidirectional replication to ensure high availability of your database if there's availability zone-level downtime.

In the preceding diagram, the Extract process runs on the same server as your Oracle database. The Data Pump and Replicat processes run on a separate server in the same availability zone. The Replicat process is used to receive data from the database in the other availability zone and commit the data to the Oracle database in its availability zone. Similarly, the Data Pump process sends data that the Extract process extracts to the Replicat process in the other availability zone.

While the preceding architecture diagram shows the Data Pump and Replicat processes configured on a separate server, you might set up all the Oracle GoldenGate processes on the same server, based on the capacity and usage of your server. Always consult your AWR report and the metrics in Azure to understand the usage pattern of your server.

When setting up Oracle GoldenGate bidirectional replication in different availability zones or different regions, it's important to ensure that the latency between the different components is acceptable for your application. The latency between availability zones and regions can  vary. Latency depends on multiple factors. We recommend that you set up performance tests between your application tier and your database tier in different availability zones or regions. The tests can confirm that the configuration meets your application performance requirements.

The application tier can be set up in its own subnet and the database tier can be separated into its own subnet. When possible, consider using [Azure Application Gateway](/azure/application-gateway/overview) to load-balance traffic between your application servers. Application Gateway is a robust web traffic load balancer. It provides cookie-based session affinity that keeps a user session on the same server, minimizing the conflicts on the database. Alternatives to Application Gateway are [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) and [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).

### Oracle Sharding

Sharding is a data tier pattern that was introduced in Oracle 12.2. It allows you to horizontally partition and scale your data across independent databases. It's a share-nothing architecture where each database is hosted on a dedicated virtual machine. This pattern enables high read and write throughput in addition to resiliency and increased availability.

This pattern eliminates single points of failure, provides fault isolation, and enables rolling upgrades without downtime. The downtime of one shard or a data center-level failure doesn't affect the performance or availability of the other shards in other data centers.

Sharding is suitable for high throughput OLTP applications that can't afford any downtime. All rows with the same sharding key are always guaranteed to be on the same shard. This fact increases performance, providing high consistency. Applications that use sharding must have a well-defined data model and data distribution strategy, such as consistent hash, range, list, or composite. The strategy primarily accesses data using a sharding key, for example, *customerId* or *accountNum*. Sharding also allows you to store particular sets of data closer to the end customers, thus helping meet your performance and compliance requirements.

We recommend that you replicate your shards for high availability and disaster recovery. This setup can be done using Oracle technologies such as Oracle Data Guard or Oracle GoldenGate. A unit of replication can be a shard, a part of a shard, or a group of shards. An outage or slowdown of one or more shards doesn't affect the availability of a sharded database.

For high availability, the standby shards can be placed in the same availability zone where the primary shards are placed. For disaster recovery, the standby shards can be located in another region. You might also deploy shards in multiple regions to serve traffic in those regions. To learn more about configuring high availability and replication of your sharded database, see [Shard-Level High Availability](https://docs.oracle.com/en/database/oracle/oracle-database/18/shard/sharding-high-availability.html).

Oracle Sharding primarily consists of the following components. For more information, see [Oracle Sharding Overview](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-overview.html):

- **Shard catalog**. Special-purpose Oracle database that is a persistent store for all Shard database configuration data. All configuration changes such as adding or removing shards, mapping of the data, and DDLs in a shard database are initiated on the shard catalog. The shard catalog also contains the master copy of all duplicated tables in an SDB.

  The shard catalog uses materialized views to automatically replicate changes to duplicated tables in all shards. The shard catalog database also acts as a query coordinator used to process multi-shard queries and queries that don't specify a sharding key.
  
  We recommend using Oracle Data Guard with availability zones or availability sets for shard catalog high availability as a best practice. The availability of the shard catalog has no effect on the availability of the sharded database. A downtime in the shard catalog only affects maintenance operations and multishard queries during the brief period that the Data Guard failover completes. The SDB continues to route and run online transactions. A catalog outage doesn't affect them.

- **Shard directors**. Lightweight services that need to be deployed in each region/availability zone that your shards reside in. Shard Directors are Global Service Managers deployed in the context of Oracle Sharding. For high availability, we recommend that you deploy at least one shard director in each availability zone that your shards exist in.

  When connecting to the database initially, the shard director sets up the routing information and caches the information for subsequent requests, which bypass the shard director. Once the session is established with a shard, all SQL queries and DMLs are supported and executed in the scope of the given shard. This routing is fast and is used for all OLTP workloads that perform intra-shard transactions. We recommend that you to use direct routing for all OLTP workloads that require the highest performance and availability. The routing cache automatically refreshes when a shard becomes unavailable or changes occur to the sharding topology.
  
  For high-performance, data-dependent routing, Oracle recommends using a connection pool when accessing data in the sharded database. Oracle connection pools, language-specific libraries, and drivers support Oracle Sharding. For more information, see [Oracle Sharding Overview](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-overview.html#GUID-3D41F762-BE04-486D-8018-C7A210D809F9).

- **Global service**. Global service is similar to the regular database service. In addition to all the properties of a database service, a global service has properties for sharded databases. These properties include region affinity between clients and shard and replication lag tolerance. Only one global service needs to be created to read/write data to and from a sharded database. When using Active Data Guard and setting up read-only replicas of the shards, you can create another global service for read-only workloads. The client can use these global services to connect to the database.

- **Shard databases**. Shard databases are your Oracle databases. Each database is replicated using Oracle Data Guard in a Broker configuration with FSFO enabled. You don't need to set up Data Guard failover and replication on each shard. This aspect is automatically configured and deployed when the shared database is created. If a particular shard fails, Oracle Sharing fails over database connections from the primary to the standby.

You can deploy and manage Oracle sharded databases with two interfaces: Oracle Enterprise Manager Cloud Control GUI and the `GDSCTL` command-line utility. You can even monitor the different shards for availability and performance using Cloud control. The `GDSCTL DEPLOY` command automatically creates the shards and their respective listeners. In addition, this command automatically deploys the replication configuration used for shard-level high availability specified by the administrator.

There are different ways to shard a database:

- System-managed sharding: Automatically distributes across shards using partitioning
- User-defined sharding: Allows you to specify the mapping of the data to the shards, which works well when there are regulatory or data-localization requirements
- Composite sharding: A combination of system-managed and user-defined sharding for different _shardspaces_
- Table subpartitions: Similar to a regular partitioned table

For more information, see [Sharding Methods](https://docs.oracle.com/en/database/oracle/oracle-database/18/shard/sharding-methods.html).

A sharded database looks like a single database to applications and developers. When you migrate to a sharded database, plan carefully to understand which tables are duplicated versus sharded.

Duplicated tables are stored on all shards, whereas sharded tables are distributed across different shards. We recommend that you duplicate small and dimensional tables and distribute/shard the fact tables. Data can be loaded into your sharded database using either the shard catalog as the central coordinator or by running Data Pump on each shard. For more information, see [Migrating Data to a Sharded Database](https://docs.oracle.com/en/database/oracle/oracle-database/18/shard/sharding-loading-data.html).

#### Oracle Sharding with Data Guard

Oracle Data Guard can be used for sharding with system-managed, user-defined, and composite sharding methods.

The following diagram is a reference architecture for Oracle Sharding with Oracle Data Guard used for high availability of each shard. The architecture diagram shows a _composite sharding method_. The architecture diagram likely differs for applications with different requirements for data locality, load balancing, high availability, and disaster recovery. Applications might use different method for sharding. Oracle Sharding allows you to meet these requirements and scale horizontally and efficiently by providing these options. A similar architecture can even be deployed using Oracle GoldenGate.

:::image type="content" source="./media/oracle-reference-architecture/oracledb_dg_sh_az.png" alt-text="Diagram that shows Oracle Database Sharding using availability zones with Data Guard Broker - FSFO." lightbox="./media/oracle-reference-architecture/oracledb_dg_sh_az.png":::

System-managed sharding is the easiest to configure and manage. User-defined sharding or composite sharding is well suited for scenarios where your data and application are geo-distributed or in scenarios where you need to have control over the replication of each shard.

In the preceding architecture, composite sharding is used to geodistribute the data and horizontally scale out your application tiers. Composite sharding is a combination of system-managed and user-defined sharding and thus provides the benefit of both methods. In the preceding scenario, data is first sharded across multiple shardspaces separated by region. Then, the data is further partitioned by using consistent hash across multiple shards in the shardspace.

Each shardspace contains multiple shardgroups. Each shardgroup has multiple shards and is a unit of replication. Each shardgroup contains all the data in the shardspace. Shardgroups A1 and B1 are primary shardgroups, while shardgroups A2 and B2 are standbys. You might choose to have individual shards be the unit of replication, rather than a shardgroup.

In the preceding architecture, a Global Service Manager (GSM)/shard director is deployed in every availability zone for high availability. We recommend that you deploy at least one GSM/shard director per data center/region. Additionally, an instance of the application server is deployed in every availability zone that contains a shardgroup. This setup allows the application to keep the latency between the application server and the database/shardgroup low. If a database fails, the application server in the same zone as the standby database can handle requests once the database role transition happens. Azure Application Gateway and the shard director keep track of the request and response latency and route requests accordingly.

From an application standpoint, the client system makes a request to Azure Application Gateway or other load-balancing technologies in Azure, which redirects the request to the region closest to the client. Azure Application Gateway also supports sticky sessions, so any requests coming from the same client are routed to the same application server. The application server uses connection pooling in data access drivers. This feature is available in drivers such as JDBC, ODP.NET, and OCI. The drivers can recognize sharding keys specified as part of the request. Oracle Universal Connection Pool (UCP) for JDBC clients can enable non-Oracle application clients such as Apache Tomcat and IIS to work with Oracle Sharding. For more information, see [Overview of UCP Shared Pool for Database Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/jjucp/ucp-database-sharding-overview.html).

During the initial request, the application server connects to the shard director in its region to get routing information for the shard that the request needs to be routed to. Based on the sharding key passed, the director routes the application server to the respective shard. The application server caches this information by building a map, and for subsequent requests, bypasses the shard director and routes requests straight to the shard.

#### Oracle Sharding with GoldenGate

The following diagram is a reference architecture for Oracle Sharding with Oracle GoldenGate for in-region high availability of each shard. As opposed to the preceding architecture, this architecture only portrays high availability within a single Azure region, with multiple availability zones. You can deploy a multi-region high availability sharded database, similar to the preceding example, by using Oracle GoldenGate.

:::image type="content" source="./media/oracle-reference-architecture/oracledb_gg_sh_az.png" alt-text="Diagram that shows Oracle Database Sharding using availability zones with GoldenGate." lightbox="./media/oracle-reference-architecture/oracledb_gg_sh_az.png":::

The preceding reference architecture uses the _system-managed_ sharding method to shard the data. Since Oracle GoldenGate replication is done at a chunk level, half the data replicated to one shard can be replicated to another shard. The other half can be replicated to a different shard.

The way the data gets replicated depends on the replication factor. With a replication factor of two, you have two copies of each chunk of data across your three shards in the shardgroup. Similarly, with a replication factor of three and three shards in your shardgroup, all the data in each shard is replicated to every other shard in the shardgroup. Each shard in the shardgroup can have a different replication factor. This setup helps you define your high availability and disaster recovery design efficiently within a shardgroup and across multiple shardgroups.

In the preceding architecture, shardgroup A and shardgroup B both contain the same data but reside in different availability zones. If both shardgroup A and shardgroup B have the same replication factor of three, each row/chunk of your sharded table is replicated six times across the two shardgroups. If shardgroup A has a replication factor of three and shardgroup B has a replication factor of two, each row/chunk is replicated five times across the two shardgroups.

This setup prevents data loss if an instance-level or availability zone-level failure occurs. The application layer is able to read from and write to each shard. To minimize conflicts, Oracle Sharding designates a *master chunk* for each range of hash values. This feature ensures that write requests for a particular chunk are directed to the corresponding chunk. In addition, Oracle GoldenGate provides automatic conflict detection and resolution to handle any conflicts that might arise. For more information and limitations of implementing GoldenGate with Oracle Sharding, see [Using Oracle GoldenGate with a Sharded Database](https://docs.oracle.com/en/database/oracle/oracle-database/18/shard/sharding-high-availability.html#GUID-4FC0AC46-0B8B-4670-BBE4-052228492C72).

In the preceding architecture, a GSM/shard director is deployed in every availability zone for high availability. We recommend that you deploy at least one GSM/shard director per data center or region. An instance of the application server is deployed in every availability zone that contains a shardgroup. This setup allows the application to keep the latency between the application server and the database/shardgroup low. If a database fails, the application server in the same zone as the standby database can handle requests once the database role transitions. Azure Application Gateway and the shard director keep track of the request and response latency and route requests accordingly.

From an application standpoint, the client system makes a request to Azure Application Gateway or other load-balancing technologies in Azure, which redirects the request to the region closest to the client. Azure Application Gateway also supports sticky sessions, so any requests coming from the same client are routed to the same application server. The application server uses connection pooling in data access drivers. This feature is available in drivers such as JDBC, ODP.NET, and OCI. The drivers can recognize sharding keys specified as part of the request. [Oracle Universal Connection Pool (UCP)](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/jjucp/ucp-database-sharding-overview.html) for JDBC clients can enable non-Oracle application clients such as Apache Tomcat and IIS to work with Oracle Sharding.

During the initial request, the application server connects to the shard director in its region to get routing information for the shard that the request needs to be routed to. Based on the sharding key passed, the director routes the application server to the respective shard. The application server caches this information by building a map, and for subsequent requests, bypasses the shard director and routes requests straight to the shard.

## Patching and maintenance

When you deploy your Oracle workloads to Azure, Microsoft takes care of all host operating system level patching. Microsoft communicates any planned operating system level maintenance to customers in advance. Two servers from two different availability zones are never patched simultaneously. For more information on VM maintenance and patching, see [Availability options for Azure Virtual Machines](../../availability.md).

Patching your virtual machine operating system can be automated using [Azure Automation Update Management](/azure/automation/update-management/overview). Patching and maintaining your Oracle database can be automated and scheduled using [Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines) or [Azure Automation Update Management](/azure/automation/update-management/overview) to minimize downtime. For more information about continuous delivery and blue/green deployments, see [Progressive exposure techniques](/devops/deliver/what-is-continuous-delivery#progressive-exposure-techniques).

## Architecture and design considerations

- Consider using hyperthreaded [memory optimized virtual machine](../../sizes-memory.md) with [constrained core vCPUs](../../../virtual-machines/constrained-vcpu.md) for your Oracle Database VM to save on licensing costs and maximize performance. Use multiple premium or ultra disks (managed disks) for performance and availability.
- When you use managed disks, the disk/device name might change on restart. We recommend that you use the device UUID instead of the name to ensure your mounts persist in sprite of restarting. For more information, see [Add the new file system to /etc/fstab](/previous-versions/azure/virtual-machines/linux/configure-raid#add-the-new-file-system-to-etcfstab).
- Use availability zones to achieve high availability in-region.
- Consider using ultra disks when available or premium disks for your Oracle database.
- Consider setting up a standby Oracle database in another Azure region using Oracle Data Guard.
- Consider using [proximity placement groups](../../co-location.md#proximity-placement-groups) to reduce the latency between your application and database tier.
- Azure VMs maximum network bandwidth is (typically) higher than the maximum disk throughput on the same SKU. You can achieve higher throughput on the same VM SKU or use a smaller VM SKU by using high-performance, low-latency networked storage such as [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction). for the database.
- Set up [Oracle Enterprise Manager](https://docs.oracle.com/en/enterprise-manager/) for management, monitoring, and logging.
- Consider using Oracle Automatic Storage Management for streamlined storage management for your database.
- [Introduction to Oracle Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/23/sbydb/introduction-to-oracle-data-guard-concepts.html)
- Tweak your application code to add cloud-native patterns that might help your application be more resilient. Consider patterns such as [retry pattern](/azure/architecture/patterns/retry), [circuit breaker pattern](/azure/architecture/patterns/circuit-breaker), and others defined in the [Cloud Design Patterns guide](/azure/architecture/patterns/).

## Next steps

Review the following Oracle reference articles that apply to your scenario.

- [Introduction to Oracle Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/23/sbydb/introduction-to-oracle-data-guard-concepts.html)
- [Oracle Data Guard Broker Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/dgbkr/oracle-data-guard-broker-concepts.html#GUID-723E5B73-A350-4B2E-AF3C-5EA4EFC8396)
- [Configuring Oracle GoldenGate for Active-Active High Availability](https://docs.oracle.com/en/middleware/goldengate/core/19.1/admin/configuring-oracle-goldengate-active-active-high-availability.html)
- [Oracle Sharding Overview](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-overview.html)
