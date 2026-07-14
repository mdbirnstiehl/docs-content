---
applies_to:
  stack: preview 9.5+
  serverless: preview
products:
  - id: observability
  - id: cloud-serverless
---

# Create a composite SLO [observability-create-a-composite-slo]

A composite SLO aggregates multiple individual SLOs into a single health indicator using a weighted average. Use composite SLOs to track reliability at the system or platform level. For example, you could create a "Checkout Flow Health" composite that combines SLOs for your payment service, cart service, and inventory service.

::::{important}
**For Observability serverless projects**, the **Editor** role or higher is required to create SLOs. To learn more, refer to [Assign user roles and privileges](/deploy-manage/users-roles/cloud-organization/user-roles.md#general-assign-user-roles).

**For {{product.elastic-stack}}**, to create and manage SLOs, you need an [appropriate license](https://www.elastic.co/subscriptions), an {{es}} cluster with both `transform` and `ingest` [node roles](elasticsearch://reference/elasticsearch/configuration-reference/node-settings.md#node-roles) present, and [SLO access](/solutions/observability/incident-management/configure-service-level-objective-slo-access.md) must be configured.
::::

::::{note}
Composite SLOs are enabled automatically for {{product.serverless-observability}} and {{ech}} deployments. For self-managed deployments, add the following to your `kibana.yml` to enable the feature:

```yaml
feature_flags.overrides:
  slo.compositeSloEnabled: true
```
::::

To create a composite SLO:

1. Find **SLOs** in the main menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Select the **Composite SLOs** tab.
1. Click **Create composite SLO**, then complete the following steps.

::::::{stepper}

:::::{step} Name and description

Give your composite SLO a name and an optional description to help others understand what system or platform it represents.

:::::

:::::{step} Member SLOs

Add the individual SLOs that make up the composite. A composite SLO can include up to 25 member SLOs.

For each member SLO, set the following:

* **SLO**: Select an existing SLO. When the selected SLO has multiple instances (created using the **Group by** field), you can optionally filter to specific instance IDs. By default, the composite SLO includes all instances of the selected SLO.
* **Weight**: A numeric weight that controls how much this SLO contributes to the composite SLI value. Weights are relative. For example, a weight of 2 counts twice as much as a weight of 1. The default weight is 1.

The composite SLI is a weighted average of all member SLI values.

:::::

:::::{step} Time window

Select a **rolling** time window duration for the composite SLO. Rolling windows use data from a fixed lookback period that moves forward as time passes. For example, the last 30 days.

You can set any rolling duration up to 1 year.

:::::

:::::{step} Budgeting method

Select the **Occurrences** budgeting method, which computes the SLI using the number of good events and the number of total events across all member SLOs.

:::::

:::::{step} Target

Set the composite SLO's target as a percentage. This target is independent of the targets set on individual member SLOs.

:::::

::::::


## Viewing composite SLOs [composite-slo-viewing]

Your composite SLO appears on the **Composite SLOs** tab of the **SLOs** page. Each row displays:

* Name
* SLI value (weighted average of member SLIs)
* Objective
* Error budget remaining
* Burn rate over 5m, 1h, 6h, and 24h windows
* The number of member SLOs by status (for example, "3 healthy of 25")
* Number of active alerts for the underlying member SLOs

Expand a row to see the individual member SLOs and their status, SLI value, error budget remaining, burn rates, weights, and active alerts.

::::{note}
Composite SLOs do not have a dedicated detail page. Use the expandable row to inspect member SLO health.
::::