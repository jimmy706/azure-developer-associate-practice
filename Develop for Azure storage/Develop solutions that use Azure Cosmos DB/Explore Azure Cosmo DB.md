# Identify key benefits of Azure Cosmos DB

> Azure Cosmo DB is a fully managed NoSQL database that provides low latency, elastic scalability of throughput, well-defined semantics for data consistency, and high availability.

You can configure your databases to be globally distributed and available in any of the Azure regions. To lower the latency, place the data close to where your users are.

## Key benefits of global distribution
With its novel multi-master replication protocol, every region supports both writes and reads. The multi-master capability also enables:

- Unlimited elastic write and read scalability.
- 99.999% read and write availability all around the world.
- Guaranteed reads and writes served in less than 10 milliseconds at the 99th percentile.
Your application can perform near real-time reads and writes against all the regions you chose for your database. Azure Cosmos DB internally handles the data replication between regions with consistency level guarantees of the level you've selected.

Running a database in multiple regions worldwide increases the availability of a database. If one region is unavailable, other regions automatically handle application requests. Azure Cosmos DB offers 99.999% read and write availability for multi-region databases.

# Explore the resource hierarchy
The Azure Cosmos DB account is the fundamental unit of global distribution and high availability. Your Azure Cosmos DB account contains a unique DNS name and you can manage an account by using the Azure portal or the Azure CLI, or by using different language-specific SDKs. For globally distributing your data and throughput across multiple Azure regions, you can add and remove Azure regions to your account at any time.

## Elements of Azure Cosmo DB Account
An Azure Cosmos DB container is the fundamental unit of scalability. You can virtually have an unlimited provisioned throughput (RU/s) and storage on a container. Azure Cosmos DB transparently partitions your container using the logical partition key that you specify in order to elastically scale your provisioned throughput and storage.

Currently, you can create a maximum of 50 Azure Cosmos DB accounts under an Azure subscription (this is a soft limit that can be increased via support request). After you create an account under your Azure subscription, you can manage the data in your account by creating databases, containers, and items.

The following image shows the hierarchy of different entities in an Azure Cosmos DB account:

![Azure Cosmo DB Account](https://learn.microsoft.com/en-us/training/wwl-azure/explore-azure-cosmos-db/media/cosmos-entities.png)

### Azure Cosmos DB databases
You can create one or multiple Azure Cosmos DB databases under your account. A database is analogous to a namespace. 
>*A database is the unit of management for a set of Azure Cosmos DB containers*.

### Azure Cosmos DB Containers
>An Azure Cosmos DB container is the unit of scalability both for provisioned throughput and storage.

A container is horizontally partitioned and then replicated across multiple regions. The items that you add to the container are automatically grouped into logical partitions, which are distributed across physical partitions, based on the partition key. The throughput on a container is evenly distributed across the physical partitions. When you create a container, throughput can be configured by following modes:
- **Dedicated provisioned:**  The throughput provisioned on a container is exclusively reserved for that container and it's backed by the SLAs.
- **Shared provisioned:** These containers share the provisioned throughput with the other containers in the same database (excluding containers that have been configured with dedicated provisioned throughput). In other words, the provisioned throughput on the database is shared among all the “shared throughput” containers.
Items in a container can have arbitrary schemas. For example, an item that represents a person and an item that represents an automobile can be placed in the same container. By default, all items that you add to a container are automatically indexed without requiring explicit index or schema management.

### Azure Cosmos DB Items
Depending on which API you use, an Azure Cosmos DB item can represent either a document in a collection, a row in a table, or a node or edge in a graph.

# Explore consistency levels
Azure Cosmos DB approaches data consistency as a spectrum of choices instead of two extremes. Strong consistency and eventual consistency are at the ends of the spectrum, but there are many consistency choices along the spectrum. Developers can use these options to make precise choices and granular tradeoffs with respect to high availability and performance.

Azure Cosmos DB offers five well-defined levels. From strongest to weakest, the levels are:
- Strong
- Bounded staleness
- Session
- Consistent prefix
- Eventual
![consistency levels](https://learn.microsoft.com/en-us/training/wwl-azure/explore-azure-cosmos-db/media/five-consistency-levels.png)

The consistency levels are region-agnostic and are guaranteed for all operations regardless of the region from which the reads and writes are served, the number of regions associated with your Azure Cosmos DB account, or whether your account is configured with a single or multiple write regions.

Read consistency applies to a single read operation scoped within a partition-key range or a logical partition. The read operation can be issued by a remote client or a stored procedure.

# Choice the right consistency level
Each of the consistency models can be used for specific real-world scenarios. Each provides precise availability and performance tradeoffs and is backed by comprehensive SLAs. The following simple considerations help you make the right choice in many common scenarios.

## Configure default consistency level
You can configure the default consistency level on your Azure Cosmos DB account at any time. The default consistency level configured on your account applies to all Azure Cosmos DB databases and containers under that account. All reads and queries issued against a container or a database use the specified consistency level by default.

Read consistency applies to a single read operation scoped within a logical partition. The read operation can be issued by a remote client or a stored procedure.

## Guarantees associated with consistency levels
### Strong consistency
Strong consistency offers a linearizability guarantee. Linearizability refers to serving requests concurrently. The reads are guaranteed to return the most recent committed version of an item. A client never sees an uncommitted or partial write. Users are always guaranteed to read the latest committed write.
![strong consistency](https://learn.microsoft.com/en-us/azure/cosmos-db/media/consistency-levels/strong-consistency.gif)

### Bounded staleness consistency
In bounded staleness consistency, the reads are guaranteed to honor the consistent-prefix guarantee. The reads might lag behind writes by at most "K" versions (that is, "updates") of an item or by "T" time interval, whichever is reached first. In other words, when you choose bounded staleness, the "staleness" can be configured in two ways:

- The number of versions (K) of the item
- The time interval (T) reads might lag behind the writes

For a single region account, the minimum value of K and T is 10 write operations or 5 seconds. For multi-region accounts the minimum value of K and T is 100,000 write operations or 300 seconds.

![bounded staleness consistency](https://learn.microsoft.com/en-us/azure/cosmos-db/media/consistency-levels/bounded-staleness-consistency.gif)

### Session consistency
In session consistency, within a single client session reads are guaranteed to honor the consistent-prefix, monotonic reads, monotonic writes, read-your-writes, and write-follows-reads guarantees. This assumes a single "writer" session or sharing the session token for multiple writers.

![session consistency](https://learn.microsoft.com/en-us/azure/cosmos-db/media/consistency-levels/session-consistency.gif)

### Consistent prefix consistency

In consistent prefix, updates made as single document writes see eventual consistency. Updates made as a batch within a transaction, are returned consistent to the transaction in which they were committed. Write operations within a transaction of multiple documents are always visible together.

Assume two write operations are performed on documents _Doc 1_ and _Doc 2_, within transactions T1 and T2. When client does a read in any replica, the user sees either “_Doc 1_ v1 and _Doc 2_ v1” or “_Doc 1_ v2 and _Doc 2_ v2”, but never “_Doc 1_ v1 and _Doc 2_ v2” or “_Doc 1_ v2 and _Doc 2_ v1” for the same read or query operation.

![consistent prefix consistency](https://learn.microsoft.com/en-us/azure/cosmos-db/media/consistency-levels/consistent-prefix.gif)

### Eventual consistency
In eventual consistency, there's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge.

Eventual consistency is the weakest form of consistency because a client may read the values that are older than the ones it read before. Eventual consistency is ideal where the application doesn't require any ordering guarantees. Examples include count of Retweets, Likes, or nonthreaded comments.
![eventual consistency](https://learn.microsoft.com/en-us/azure/cosmos-db/media/consistency-levels/eventual-consistency.gif)

# Explore supported APIs
Azure Cosmos DB offers multiple database APIs, which include:

- Azure Cosmos DB for NoSQL
- Azure Cosmos DB for MongoDB
- Azure Cosmos DB for PostgreSQL
- Azure Cosmos DB for Apache Cassandra
- Azure Cosmos DB for Table
- Azure Cosmos DB for Apache Gremlin

By using these APIs, you can model real world data using documents, key-value, graph, and column-family data models. These APIs allow your applications to treat Azure Cosmos DB as if it were various other databases technologies, without the overhead of management, and scaling approaches.

## Considerations when choosing an API
API for NoSQL is native to Azure Cosmos DB.

API for MongoDB, PostgreSQL, Cassandra, Gremlin, and Table implement the wire protocol of open-source database engines. These APIs are best suited if the following conditions are true:

- If you have existing MongoDB, PostgreSQL Cassandra, or Gremlin applications
- If you don't want to rewrite your entire data access layer
- If you want to use the open-source developer ecosystem, client-drivers, expertise, and resources for your database
### API for NoSQL
The Azure Cosmos DB API for NoSQL stores data in document format. It offers the best end-to-end experience as we have full control over the interface, service, and the SDK client libraries. Any new feature that is rolled out to Azure Cosmos DB is first available on API for NoSQL accounts. NoSQL accounts provide support for querying items using the Structured Query Language (SQL) syntax.
### API for MongoDB
The Azure Cosmos DB API for MongoDB stores data in a document structure, via BSON format. It's compatible with MongoDB wire protocol; however, it doesn't use any native MongoDB related code. The API for MongoDB is a great choice if you want to use the broader MongoDB ecosystem and skills, without compromising on using Azure Cosmos DB features.
### API for PostgreSQL
Azure Cosmos DB for PostgreSQL is a managed service for running PostgreSQL at any scale, with the [Citus open source](https://github.com/citusdata/citus) superpower of distributed tables. It stores data either on a single node, or distributed in a multi-node configuration.

# Discover request units
With Azure Cosmos DB, you pay for the throughput you provision and the storage you consume on an hourly basis. Throughput must be provisioned to ensure that sufficient system resources are available for your Azure Cosmos database always.

The cost of all database operations is normalized by Azure Cosmos DB and is expressed by _request units_ (or RUs, for short). A request unit represents the system resources such as CPU, IOPS, and memory that are required to perform the database operations supported by Azure Cosmos DB.

The cost to do a point read, which is fetching a single item by its ID and partition key value, for a 1-KB item is 1RU. All other database operations are similarly assigned a cost using RUs. No matter which API you use to interact with your Azure Cosmos container, costs are measured by RUs. Whether the database operation is a write, point read, or query, costs are measured in RUs.

The following image shows the high-level idea of RUs:
![request units](https://learn.microsoft.com/en-us/training/wwl-azure/explore-azure-cosmos-db/media/request-units.png)

The type of Azure Cosmos DB account you're using determines the way consumed RUs get charged. There are three modes in which you can create an account:

- **Provisioned throughput mode**: In this mode, you provision the number of RUs for your application on a per-second basis in increments of 100 RUs per second. To scale the provisioned throughput for your application, you can increase or decrease the number of RUs at any time in increments or decrements of 100 RUs. You can make your changes either programmatically or by using the Azure portal. You can provision throughput at container and database granularity level.
    
- **Serverless mode**: In this mode, you don't have to provision any throughput when creating resources in your Azure Cosmos DB account. At the end of your billing period, you get billed for the number of request units that have been consumed by your database operations.
    
- **Autoscale mode:** In this mode, you can automatically and instantly scale the throughput (RU/s) of your database or container based on its usage. This scaling operation doesn't affect the availability, latency, throughput, or performance of the workload. This mode is well suited for mission-critical workloads that have variable or unpredictable traffic patterns, and require SLAs on high performance and scale.
