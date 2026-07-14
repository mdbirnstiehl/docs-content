---
navigation_title: Detection
description: How Significant Events uses alerting rules and the Elasticsearch change_point aggregation to detect statistically significant changes in your log streams.
applies_to:
  serverless: preview
  stack: preview 9.5+
products:
  - id: observability
  - id: elasticsearch
  - id: kibana
  - id: cloud-serverless
  - id: cloud-hosted
  - id: cloud-enterprise
  - id: cloud-kubernetes
  - id: elastic-stack
---

# Detection [sig-events-detection]

Detection is the third phase of the Significant Events pipeline. It converts a continuous stream of alerting rule results into a discrete set of statistically significant change records that the discovery workflow can act on.

Detection has two layers:

1. **Alerting rule execution** — promoted Query KIs run as Kibana alerting rules that continuously scan stream data
2. **Change detection** — a Workflows-based detection job runs the Elasticsearch `change_point` aggregation over alert firing patterns to identify genuine transitions

## Alerting rule execution [sig-events-rule-execution]

When a Query KI is promoted, it becomes a Kibana alerting rule of type `streams.rules.esql`. Each rule runs ES\|QL against the stream's data on a per-rule schedule. The execution interval depends on the Query KI's severity score:

| Severity | Score range | Interval | Lookback window |
|---|---|---|---|
| Critical | ≥ 80 | 1 minute | 2 minutes |
| High / Medium / Low / unscored | < 80 | 5 minutes | 10 minutes |

The lookback window is always twice the interval, so no events fall through the gap between runs. Deduplication is content-based — the same source event cannot produce duplicate alerts for the same rule regardless of overlap between windows.

Results are written to `.alerts-streams.alerts-default`. This index uses `dynamic: false` mapping, which means only Kibana alerting framework fields (`kibana.alert.*`) are explicitly indexed. The original source event fields are stored but not typed, so ES\|QL queries against the alerts index can only operate on alerting framework fields.

One alerting rule is created per promoted Query KI. The number of active rules directly affects the query load on your cluster.

## Change detection [sig-events-change-detection]

A detection workflow runs on a schedule and applies the Elasticsearch `change_point` aggregation to the alerts index. For each active alerting rule, the aggregation buckets alert events by time (1-minute intervals) and identifies statistically significant changes in the firing pattern.

The detection workflow uses a lookback window of the last 40 minutes (`now-40m`) for the aggregation.

The workflow only writes a detection record when it observes a **transition** — a change from the previously recorded state for that rule. A rule that has been continuously firing at the same rate does not produce a new detection. This keeps the detections index focused on genuine behavioral changes.

Each detection record written to `.significant_events-detections` is immutable. The record captures:

- `detection_id` — unique identifier for this detection event
- `rule_uuid` and `rule_name` — the alerting rule that produced the signal
- `stream_name` — the stream the rule monitors
- `change_point_type` — the type of statistical change detected
- `p_value` — the statistical significance of the change
- `@timestamp` — when the change was detected

Recovery and quiet states are **not** determined by the detection workflow. The discovery workflow reads those lifecycle states from the alerts index directly when it processes a batch.

## Relationship to discovery [sig-events-detection-to-discovery]

The detection workflow feeds `.significant_events-detections`. The discovery workflow reads from this index, collapses results by `rule_uuid` to get the most recent detection per rule, and filters out detections it has already processed. See [Discovery](./discovery.md) for how detections become discoveries.
