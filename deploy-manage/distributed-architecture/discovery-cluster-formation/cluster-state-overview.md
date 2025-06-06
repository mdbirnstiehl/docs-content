---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state-overview.html
applies_to:
  stack:
products:
  - id: elasticsearch
---

# Cluster state [cluster-state-overview]

The *cluster state* is an internal data structure which keeps track of a variety of information needed by every node, including:

* The identity and attributes of the other nodes in the cluster
* Cluster-wide settings
* Index metadata, including the mapping and settings for each index
* The location and status of every shard copy in the cluster

The elected master node ensures that every node in the cluster has a copy of the same cluster state. The [cluster state API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-state) lets you retrieve a representation of this internal state for debugging or diagnostic purposes.

## Publishing the cluster state [cluster-state-publishing]

The elected master node is the only node in a cluster that can make changes to the cluster state. The elected master node processes one batch of cluster state updates at a time, computing the required changes and publishing the updated cluster state to all the other nodes in the cluster. Each publication starts with the elected master broadcasting the updated cluster state to all nodes in the cluster. Each node responds with an acknowledgement but does not yet apply the newly-received state. Once the elected master has collected acknowledgements from enough master-eligible nodes, the new cluster state is said to be *committed* and the master broadcasts another message instructing nodes to apply the now-committed state. Each node receives this message, applies the updated state, and then sends a second acknowledgement back to the master.

The elected master allows a limited amount of time for each cluster state update to be completely published to all nodes. It is defined by the `cluster.publish.timeout` setting, which defaults to `30s`, measured from the time the publication started. If this time is reached before the new cluster state is committed then the cluster state change is rejected and the elected master considers itself to have failed. It stands down and starts trying to elect a new master node.

If the new cluster state is committed before `cluster.publish.timeout` has elapsed, the elected master node considers the change to have succeeded. It waits until the timeout has elapsed or until it has received acknowledgements that each node in the cluster has applied the updated state, and then starts processing and publishing the next cluster state update. If some acknowledgements have not been received (i.e. some nodes have not yet confirmed that they have applied the current update), these nodes are said to be *lagging* since their cluster states have fallen behind the elected master’s latest state. The elected master waits for the lagging nodes to catch up for a further time, `cluster.follower_lag.timeout`, which defaults to `90s`. If a node has still not successfully applied the cluster state update within this time then it is considered to have failed and the elected master removes it from the cluster.

Cluster state updates are typically published as diffs to the previous cluster state, which reduces the time and network bandwidth needed to publish a cluster state update. For example, when updating the mappings for only a subset of the indices in the cluster state, only the updates for those indices need to be published to the nodes in the cluster, as long as those nodes have the previous cluster state. If a node is missing the previous cluster state, for example when rejoining a cluster, the elected master will publish the full cluster state to that node so that it can receive future updates as diffs.

::::{note} 
{{es}} is a peer to peer based system, in which nodes communicate with one another directly. The high-throughput APIs (index, delete, search) do not normally interact with the elected master node. The responsibility of the elected master node is to maintain the global cluster state which includes reassigning shards when nodes join or leave the cluster. Each time the cluster state is changed, the new state is published to all nodes in the cluster as described above.
::::


The performance characteristics of cluster state updates are a function of the speed of the storage on each master-eligible node, as well as the reliability and latency of the network interconnections between all nodes in the cluster. You must therefore ensure that the storage and networking available to the nodes in your cluster are good enough to meet your performance goals.


## Dangling indices [dangling-index]

When a node joins the cluster, if it finds any shards stored in its local data directory that do not already exist in the cluster state, it will consider those shards to belong to a "dangling" index. You can list, import or delete dangling indices using the [Dangling indices API](https://www.elastic.co/docs/api/doc/elasticsearch/group/endpoint-indices).

::::{note} 
The API cannot offer any guarantees as to whether the imported data truly represents the latest state of the data when the index was still part of the cluster.
::::



