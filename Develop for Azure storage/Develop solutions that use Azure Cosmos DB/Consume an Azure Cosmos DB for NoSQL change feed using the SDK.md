#azure-cosmo-db

> Source: https://learn.microsoft.com/en-us/training/modules/consume-azure-cosmos-db-sql-api-change-feed-use-sdk/
# Understand change feed features in the SDK
The .NET SDK for Azure Cosmos DB for NoSQL ships with a change feed processor that simplifies the task of reading changes from the feed. The change feed processor also natively supports distributed scenarios where event processing responsibilities are shared across multiple consumer client applications in an efficient manner.

|**Component**|**Description**|
|---|---|
|**Monitored container**|This container is monitored for any insert or update operations. These changes are then reflected in the feed.|
|**Lease container**|The lease container serves as a storage mechanism to manage state across multiple change feed consumers (clients).|
|**Host**|The host is a client application instance that listens for and reacts to changes from the change feed.|
|**Delegate**|The delegate is code within the client application that will implement business logic for each batch of changes.|

To further understand how these four elements of the change feed processor work together, let's look at an example in the following diagram. The monitored container stores items and uses 'City' as the partition key. The partition key values are distributed in ranges (each range represents a [physical partition](https://learn.microsoft.com/en-us/azure/cosmos-db/partitioning-overview#physical-partitions)) that contain items.

The diagram shows two compute instances, and the change feed processor assigns different ranges to each instance to maximize compute distribution. Each instance has a different, unique name.

Each range is read in parallel. A range's progress is maintained separately from other ranges in the lease container through a _lease_ document. The combination of the leases represents the current state of the change feed processor.

![change feed processor](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/media/change-feed-processor/changefeedprocessor.png)

