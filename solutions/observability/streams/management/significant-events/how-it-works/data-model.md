---
navigation_title: Data model
description: The index layout used by Significant Events and how to trace signals from rules through detections to discoveries and events using ES|QL.
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

# Data model [sig-events-data-model]

Every artifact in the Significant Events pipeline is stored as a searchable document. Nothing lives only in memory or application state. This means you can trace any Significant Event back to the detection that produced it, the alerting rule that fired, the Query KI that generated the rule, and the log sample that identified the underlying feature.

## Index layout [sig-events-indexes]

| Index | Written by | Purpose |
|---|---|---|
| `.kibana_streams_features-*` | KI extraction task | Feature KIs extracted from stream log samples |
| `.alerts-streams.alerts-default` | Alerting rule executor | One document per rule execution result |
| `.significant_events-detections` | Detection workflow | Immutable change-point detection records |
| `.significant_events-discoveries` | Investigator agent | Discovery records written by the Investigator, stamped `kind: handled` by the Judge |
| `.significant_events-events` | Judge agent | Promoted Significant Events |

The verdicts data stream (`.significant_events-verdicts`) was removed in a pipeline simplification (July 2026). The original four-stream design has been consolidated to three: detections, discoveries, and events. There is no separate verdicts or review index.

Data retention is enforced via Data Stream Lifecycle (DSL), not ILM. The default retention is 90 days for all `.significant_events-*` data streams. Retention is operator-overridable per data stream using the DSL API:

```
PUT _data_stream/.significant_events-discoveries/_lifecycle
{ "data_retention": "30d" }
```

Once an operator sets an override, the system detects the divergence on the next startup and leaves the data stream operator-managed rather than reverting it to the system default.

## Mapping notes [sig-events-mapping]

`.alerts-streams.alerts-default` uses `dynamic: false`. Only {{product.kibana}} alerting framework fields (`kibana.alert.*`) are explicitly indexed with typed mappings. The original source event fields are stored on each alert document but are not indexed, which prevents cross-stream mapping conflicts. ES\|QL queries against the alerts index can only filter on alerting framework fields, not on original event field values.

`.significant_events-detections` stores immutable records. Each document represents a single observed transition for a rule. The detection workflow never updates or deletes detection documents; it only appends new ones when a new transition is observed. The discovery workflow stamps processed detections `kind: handled` after the Investigator accounts for them, preventing re-processing in future cycles.

## Tracing a signal [sig-events-traceability]

You can trace a Significant Event backwards through the pipeline using ES|QL:

1. Start with a Significant Event from `.significant_events-events` and note its `discovery_slug`.
2. Query `.significant_events-discoveries` for documents with that `discovery_slug` to see the full discovery history and the detections it was based on.
3. Use the `rule_uuid` values from the discovery to query `.significant_events-detections` for the specific change-point records.
4. Use the `rule_uuid` to identify which alerting rule fired, then query `.alerts-streams.alerts-default` for the raw alert events.