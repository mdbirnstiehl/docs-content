---
mapped_pages:
  - https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-update-strategy.html
applies_to:
  deployment:
    eck: all
products:
  - id: cloud-kubernetes
---

# Update strategy [k8s-update-strategy]

You can use the `updateStrategy` specification to limit the number of simultaneous changes, like for example in the following cases:

* The operator takes a Pod down to restart a node and applies a new configuration value.
* The operator must spin up Pods above what is currently in the specification to migrate to a new node set.

```yaml
spec:
  updateStrategy:
    changeBudget:
      maxSurge: 3
      maxUnavailable: 1
```

`maxSurge`: Refers to the number of extra Pods that can be temporarily scheduled exceeding the number of Pods defined in the specification. This setting is useful for controlling the resource usage of the Kubernetes cluster when nodeSet configuration changes and new Pods need to be spun up to replace existing Pods. `MaxSurge` restricts the number of extra pods that can be running at any given point in time. If you have a large {{es}} cluster or a Kubernetes cluster running near capacity, not setting `maxSurge` could cause the newly created pods to temporarily use up all available spare resource capacity in the Kubernetes cluster and starve other workloads running there.

`maxUnavailable`: Refers to the number of Pods that can be unavailable out of the total number of Pods in the currently applied specification. A Pod is defined unavailable when it is not ready from a Kubernetes perspective.

The operator only tries to apply these constraints when a new specification is being applied. It is possible that the cluster state does not conform to the constraints at the beginning of the operation due to external factors. The operator will attempt to get to the desired state by adding or removing Pods as necessary while ensuring that the constraints are still satisfied.

For example, if a new specification defines a larger cluster with `maxUnavailable: 0`, the operator creates the missing Pods according to the best practices. Similarly, if a new specification defines a smaller cluster with `maxSurge: 0`, the operator safely removes the unnecessary Pods.

The operator will not enforce the change budget on version upgrades for clusters that have a non-HA setup, that is, less than three nodes. In these setups, removing a single node makes the whole cluster unavailable, and the operator will instead opt to upgrade all nodes at once. This is to avoid a situation where no progress can be made in a rolling upgrade process because the {{es}} cluster cannot form a quorum until all nodes have been upgraded.

## Specify changeBudget [k8s_specify_changebudget]

For both `maxSurge` and `maxUnavailable` you can specify the following values:

* `null` - The default value is used.
* non-negative - The value is used as is.
* negative - The value is unbounded.


## Default behavior [k8s_default_behavior]

When `updateStrategy` is not present in the specification, it defaults to the following:

```yaml
spec:
  updateStrategy:
    changeBudget:
      maxSurge: -1
      maxUnavailable: 1
```

`maxSurge` is unbounded: This means that all the required Pods are created immediately. `maxUnavailable` defaults to `1`: This ensures that the cluster has no more than one unavailable Pod at any given point in time.


## Caveats [k8s_caveats]

* With both `maxSurge` and `maxUnavailable` set to `0`, the operator cannot bring down an existing Pod nor create a new Pod.
* Due to the safety measures employed by the operator, certain `changeBudget` might prevent the operator from making any progress . For example, with `maxSurge` set to 0, you cannot remove the last data node from one `nodeSet` and add a data node to a different `nodeSet`. In this case, the operator cannot create the new node because `maxSurge` is 0, and it cannot remove the old node because there are no other data nodes to migrate the data to.
* For certain complex configurations, the operator might not be able to deduce the optimal order of operations necessary to achieve the desired outcome. If progress is blocked, you may need to update the `maxSurge` setting to a higher value than the theoretical best to help the operator make progress in that case.

In these three cases, the operator generates logs to indicate that upscaling or downscaling are limited by `maxSurge` or `maxUnavailable` settings.


