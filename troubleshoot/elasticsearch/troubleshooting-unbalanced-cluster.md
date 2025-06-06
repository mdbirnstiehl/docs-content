---
navigation_title: Unbalanced clusters
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/troubleshooting-unbalanced-cluster.html
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

% marciw move so this is with other cluster topics

# Troubleshoot an unbalanced cluster [troubleshooting-unbalanced-cluster]

Elasticsearch balances shards across data tiers to achieve a good compromise between:

* shard count
* disk usage
* write load (for indices in data streams)

::::{tip}
If you're using {{ech}}, you can use AutoOps to monitor your cluster. AutoOps significantly simplifies cluster management with performance recommendations, resource utilization visibility, and real-time issue detection with resolution paths. For more information, refer to [](/deploy-manage/monitor/autoops.md).
::::


Elasticsearch does not take into account the amount or complexity of search queries when rebalancing shards. This is indirectly achieved by balancing shard count and disk usage.

There is no guarantee that individual components will be evenly spread across the nodes. This could happen if some nodes have fewer shards, or are using less disk space, but are assigned shards with higher write loads.

Use the [cat allocation command](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cat-allocation) to list workloads per node:

```console
GET /_cat/allocation?v
```

The API returns the following response:

```text
shards shards.undesired write_load.forecast disk.indices.forecast disk.indices disk.used disk.avail disk.total disk.percent host      ip        node    node.role
     1                0                 0.0                  260b         260b    47.3gb     43.4gb    100.7gb           46 127.0.0.1 127.0.0.1 CSUXak2 himrst
```

This response contains the following information that influences balancing:

* `shards` is the current number of shards allocated to the node
* `shards.undesired` is the number of shards that needs to be moved to other nodes to finish balancing
* `disk.indices.forecast` is the expected disk usage according to projected shard growth
* `write_load.forecast` is the projected total write load associated with this node

A cluster is considered balanced when all shards are in their desired locations, which means that no further shard movements are planned (all `shards.undesired` values are equal to 0).

Some operations such as node restarting, decommissioning, or changing cluster allocation settings are disruptive and might require multiple shards to move in order to rebalance the cluster.

Shard movement order is not deterministic and mostly determined by the source and target node readiness to move a shard. While rebalancing is in progress some nodes might appear busier than others.

When a shard is allocated to an undesired node it uses the resources of the current node instead of the target. This might cause a hotspot (disk or CPU) when multiple shards reside on the current node that have not been moved to their corresponding targets yet.

If a cluster takes a long time to finish rebalancing you might find the following log entries:

```text
[WARN][o.e.c.r.a.a.DesiredBalanceReconciler] [10%] of assigned shards (10/100) are not on their desired nodes, which exceeds the warn threshold of [10%]
```

This is not concerning as long as the number of such shards is decreasing and this warning appears occasionally, for example after rolling restarts or changing allocation settings.

If the cluster has this warning repeatedly for an extended period of time (multiple hours), it is possible that the desired balance is diverging too far from the current state.

If so, increase the [`cluster.routing.allocation.balance.threshold`](elasticsearch://reference/elasticsearch/configuration-reference/cluster-level-shard-allocation-routing-settings.md#shards-rebalancing-heuristics) to reduce the sensitivity of the algorithm that tries to level up the shard count and disk usage within the cluster.

And reset the desired balance using the following API call:

$$$delete-desired-balance-request-example$$$

```console
DELETE /_internal/desired_balance
```

