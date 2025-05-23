---
navigation_title: Red or yellow health status
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/red-yellow-cluster-status.html
applies_to:
  stack:
  deployment:
    eck:
    ess:
    ece:
    self:
products:
  - id: elasticsearch
---

# Red or yellow cluster health status [red-yellow-cluster-status]

A red or yellow cluster health status indicates one or more shards are not assigned to a node.

* **Red health status**: The cluster has some unassigned primary shards, which means that some operations such as searches and indexing may fail.
* **Yellow health status**: The cluster has no unassigned primary shards but some unassigned replica shards. This increases your risk of data loss and can degrade cluster performance.

When your cluster has a red or yellow health status, it will continue to process searches and indexing where possible, but may delay certain management and cleanup activities until the cluster returns to green health status. For instance, some [{{ilm-init}}](../../manage-data/lifecycle/index-lifecycle-management.md) actions require the index on which they operate to have a green health status.

In many cases, your cluster will recover to green health status automatically. If the cluster doesn’t automatically recover, then you must [manually address](#fix-red-yellow-cluster-status) the remaining problems so management and cleanup activities can proceed. See [this video](https://www.youtube.com/watch?v=v2mbeSd1vTQ) for a walkthrough of monitoring allocation health.

::::{tip}
If you're using {{ech}}, you can use AutoOps to monitor your cluster. AutoOps significantly simplifies cluster management with performance recommendations, resource utilization visibility, and real-time issue detection with resolution paths. For more information, refer to [](/deploy-manage/monitor/autoops.md).
::::



## Diagnose your cluster status [diagnose-cluster-status]

**Check your cluster status**

Use the [cluster health API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-health).

```console
GET _cluster/health?filter_path=status,*_shards
```

A healthy cluster has a green `status` and zero `unassigned_shards`. A yellow status means only replicas are unassigned. A red status means one or more primary shards are unassigned.

**View unassigned shards**

To view unassigned shards, use the [cat shards API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cat-shards).

```console
GET _cat/shards?v=true&h=index,shard,prirep,state,node,unassigned.reason&s=state
```

Unassigned shards have a `state` of `UNASSIGNED`. The `prirep` value is `p` for primary shards and `r` for replicas.

To understand why an unassigned shard is not being assigned and what action you must take to allow {{es}} to assign it, use the [cluster allocation explanation API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-allocation-explain).

```console
GET _cluster/allocation/explain?filter_path=index,node_allocation_decisions.node_name,node_allocation_decisions.deciders.*
{
  "index": "my-index",
  "shard": 0,
  "primary": false
}
```


## Fix a red or yellow cluster status [fix-red-yellow-cluster-status]

A shard can become unassigned for several reasons. The following tips outline the most common causes and their solutions.


### Single node cluster [fix-cluster-status-only-one-node]

{{es}} will never assign a replica to the same node as the primary shard. A single-node cluster will always have yellow status. To change to green, set [number_of_replicas](elasticsearch://reference/elasticsearch/index-settings/index-modules.md#dynamic-index-number-of-replicas) to 0 for all indices.

Therefore, if the number of replicas equals or exceeds the number of nodes, some shards won’t be allocated.


### Recover lost nodes [fix-cluster-status-recover-nodes]

Shards often become unassigned when a data node leaves the cluster. This can occur for several reasons:

* A manual node restart will cause a temporary unhealthy cluster state until the node recovers.
* When a node becomes overloaded or fails, it can temporarily disrupt the cluster’s health, leading to an unhealthy state. Prolonged garbage collection (GC) pauses, caused by out-of-memory errors or high memory usage during intensive searches, can trigger this state. See [Reduce JVM memory pressure](#fix-cluster-status-jvm) for more JVM-related issues.
* Network issues can prevent reliable node communication, causing shards to become out of sync. Check the logs for repeated messages about nodes leaving and rejoining the cluster.

After you resolve the issue and recover the node, it will rejoin the cluster. {{es}} will then automatically allocate any unassigned shards.

You can monitor this process by [checking your cluster health](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-health). The number of unallocated shards should progressively decrease until green status is reached.

To avoid wasting resources on temporary issues, {{es}} [delays allocation](../../deploy-manage/distributed-architecture/shard-allocation-relocation-recovery/delaying-allocation-when-node-leaves.md) by one minute by default. If you’ve recovered a node and don’t want to wait for the delay period, you can call the [cluster reroute API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-reroute) with no arguments to start the allocation process. The process runs asynchronously in the background.

```console
POST _cluster/reroute
```


### Fix allocation settings [fix-cluster-status-allocation-settings]

Misconfigured allocation settings can result in an unassigned primary shard. These settings include:

* [Shard allocation](../../deploy-manage/distributed-architecture/shard-allocation-relocation-recovery/index-level-shard-allocation.md) index settings
* [Allocation filtering](elasticsearch://reference/elasticsearch/configuration-reference/cluster-level-shard-allocation-routing-settings.md#cluster-shard-allocation-filtering) cluster settings
* [Allocation awareness](../../deploy-manage/distributed-architecture/shard-allocation-relocation-recovery/shard-allocation-awareness.md) cluster settings

To review your allocation settings, use the [get index settings](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-get-settings) and [cluster get settings](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-get-settings) APIs.

```console
GET my-index/_settings?flat_settings=true&include_defaults=true

GET _cluster/settings?flat_settings=true&include_defaults=true
```

You can change the settings using the [update index settings](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-put-settings) and [cluster update settings](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-put-settings) APIs.


### Allocate or reduce replicas [fix-cluster-status-allocation-replicas]

To protect against hardware failure, {{es}} will not assign a replica to the same node as its primary shard. If no other data nodes are available to host the replica, it remains unassigned. To fix this, you can:

* Add a data node to the same tier to host the replica.
* Change the `index.number_of_replicas` index setting to reduce the number of replicas for each primary shard. We recommend keeping at least one replica per primary for high availability.

```console
PUT _settings
{
  "index.number_of_replicas": 1
}
```


### Free up or increase disk space [fix-cluster-status-disk-space]

{{es}} uses a [low disk watermark](elasticsearch://reference/elasticsearch/configuration-reference/cluster-level-shard-allocation-routing-settings.md#disk-based-shard-allocation) to ensure data nodes have enough disk space for incoming shards. By default, {{es}} does not allocate shards to nodes using more than 85% of disk space.

To check the current disk space of your nodes, use the [cat allocation API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cat-allocation).

```console
GET _cat/allocation?v=true&h=node,shards,disk.*
```

If your nodes are running low on disk space, you have a few options:

* Upgrade your nodes to increase disk space.
* Add more nodes to the cluster.
* Delete unneeded indices to free up space. If you use {{ilm-init}}, you can update your lifecycle policy to use [searchable snapshots](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-searchable-snapshot.md) or add a delete phase. If you no longer need to search the data, you can use a [snapshot](../../deploy-manage/tools/snapshot-and-restore.md) to store it off-cluster.
* If you no longer write to an index, use the [force merge API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-forcemerge) or {{ilm-init}}'s [force merge action](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-forcemerge.md) to merge its segments into larger ones.

    ```console
    POST my-index/_forcemerge
    ```

* If an index is read-only, use the [shrink index API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-shrink) or {{ilm-init}}'s [shrink action](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-shrink.md) to reduce its primary shard count.

    ```console
    POST my-index/_shrink/my-shrunken-index
    ```

* If your node has a large disk capacity, you can increase the low disk watermark or set it to an explicit byte value.

    ```console
    PUT _cluster/settings
    {
      "persistent": {
        "cluster.routing.allocation.disk.watermark.low": "90%",
        "cluster.routing.allocation.disk.watermark.high": "95%"
      }
    }
    ```


::::{important}
This is usually a temporary solution and may cause instability if disk space is not freed up.

::::



### Re-enable shard allocation [fix-cluster-status-reenable-allocation]

You typically disable allocation during a [restart](../../deploy-manage/maintenance/start-stop-services/full-cluster-restart-rolling-restart-procedures.md) or other cluster maintenance. If you forgot to re-enable allocation afterward, {{es}} will be unable to assign shards. To re-enable allocation, reset the `cluster.routing.allocation.enable` cluster setting.

```console
PUT _cluster/settings
{
  "persistent" : {
    "cluster.routing.allocation.enable" : null
  }
}
```

See [this video](https://www.youtube.com/watch?v=MiKKUdZvwnI) for walkthrough of troubleshooting "no allocations are allowed".


### Reduce JVM memory pressure [fix-cluster-status-jvm]

Shard allocation requires JVM heap memory. High JVM memory pressure can trigger [circuit breakers](elasticsearch://reference/elasticsearch/configuration-reference/circuit-breaker-settings.md) that stop allocation and leave shards unassigned. See [High JVM memory pressure](high-jvm-memory-pressure.md).


### Recover data for a lost primary shard [fix-cluster-status-restore]

If a node containing a primary shard is lost, {{es}} can typically replace it using a replica on another node. If you can’t recover the node and replicas don’t exist or are irrecoverable, [Allocation Explain](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-allocation-explain) will report `no_valid_shard_copy` and you’ll need to do one of the following:

* restore the missing data from [snapshot](../../deploy-manage/tools/snapshot-and-restore.md)
* index the missing data from its original data source
* accept data loss on the index-level by running [Delete Index](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-delete)
* accept data loss on the shard-level by executing [Cluster Reroute](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-reroute) allocate_stale_primary or allocate_empty_primary command with `accept_data_loss: true`

    ::::{warning}
    Only use this option if node recovery is no longer possible. This process allocates an empty primary shard. If the node later rejoins the cluster, {{es}} will overwrite its primary shard with data from this newer empty shard, resulting in data loss.
    ::::


    ```console
    POST _cluster/reroute
    {
      "commands": [
        {
          "allocate_empty_primary": {
            "index": "my-index",
            "shard": 0,
            "node": "my-node",
            "accept_data_loss": "true"
          }
        }
      ]
    }
    ```


See [this video](https://www.youtube.com/watch?v=6OAg9IyXFO4) for a walkthrough of troubleshooting `no_valid_shard_copy`.
