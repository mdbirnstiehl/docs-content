---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/nodes-shards.html
applies_to:
  stack:
  serverless:
products:
  - id: elasticsearch
---

# Clusters, nodes, and shards [nodes-shards]

::::{note}
Nodes and shards are what make {{es}} distributed and scalable. These concepts aren’t essential if you’re just getting started. How you [deploy {{es}}](../../get-started/deployment-options.md) in production determines what you need to know:

* **Self-managed {{es}}**: You are responsible for setting up and managing nodes, clusters, shards, and replicas. This includes managing the underlying infrastructure, scaling, and ensuring high availability through failover and backup strategies.
* **{{ecloud}}**: Elastic can autoscale resources in response to workload changes. Choose from different deployment types to apply sensible defaults for your use case. A basic understanding of nodes, shards, and replicas is still important.
* **{{serverless-full}}**: You don’t need to worry about nodes, shards, or replicas. These resources are 100% automated on the serverless platform, which is designed to scale with your workload.

::::

You can add servers (*nodes*) to a cluster to increase capacity, and {{es}} automatically distributes your data and query load across all of the available nodes.

Elastic is able to distribute your data across nodes by subdividing an index into *shards*. Each index in {{es}} is a grouping of one or more physical shards, where each shard is a self-contained Lucene index containing a subset of the documents in the index. By distributing the documents in an index across multiple shards, and distributing those shards across multiple nodes, {{es}} increases indexing and query capacity.

There are two types of shards: *primaries* and *replicas*. Each document in an index belongs to one primary shard. A replica shard is a copy of a primary shard. Replicas maintain redundant copies of your data across the nodes in your cluster. This protects against hardware failure and increases capacity to serve read requests like searching or retrieving a document.

::::{tip}
The number of primary shards in an index is fixed at the time that an index is created, but the number of replica shards can be changed at any time, without interrupting indexing or query operations.
::::

Shard copies in your cluster are automatically balanced across nodes to provide scale and high availability. All nodes are aware of all the other nodes in the cluster and can forward client requests to the appropriate node. This allows {{es}} to distribute indexing and query load across the cluster.

If you’re exploring {{es}} for the first time or working in a development environment, then you can use a cluster with a single node and create indices with only one shard. However, in a production environment, you should build a cluster with multiple nodes and indices with multiple shards to increase performance and resilience.

* To learn about optimizing the number and size of shards in your cluster, refer to [Size your shards](../production-guidance/optimize-performance/size-shards.md).
* To learn about how read and write operations are replicated across shards and shard copies, refer to [Reading and writing documents](reading-and-writing-documents.md).
* To adjust how shards are allocated and balanced across nodes, refer to [Shard allocation, relocation, and recovery](shard-allocation-relocation-recovery.md).