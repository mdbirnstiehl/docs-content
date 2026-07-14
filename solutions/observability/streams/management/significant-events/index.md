---
navigation_title: Significant Events
description: Significant Events automatically correlates signals from your log streams into prioritized incidents using a combination of deterministic detection and AI-powered investigation.
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

# Significant Events [streams-sig-events-overview]

<!-- NEEDS: high-level architecture diagram showing what runs where (Kibana, Elasticsearch, Workflows, Agent Builder, EIS connector) and data flow — not state-machine or YAML internals. Per engineering spec, this is a required element of this page. -->

Significant Events continuously converts raw log data into a ranked feed of incidents. The pipeline moves from broad to narrow: many alerting rule executions produce fewer statistical detections, which an AI agent correlates into even fewer discoveries, which a judge reviews and promotes into the Significant Events visible in the UI.

Four steps use LLMs. Everything in between — rule execution, statistical change detection, state tracking — runs deterministically.

## How the pipeline works [sig-events-pipeline]

The pipeline has four sequential phases:

| Phase | What happens | Output |
|---|---|---|
| 1. Knowledge extraction | An LLM analyzes log samples and generates Knowledge Indicators (KIs) describing services, infrastructure, and failure patterns | KIs stored in `.kibana_streams_features-*` |
| 2. Rule generation | An LLM generates ES\|QL alerting rules from Query KIs and promotes them to the alerting framework | Rules stored as stream assets |
| 3. Detection | Alerting rules fire on a per-rule schedule; a `change_point` aggregation identifies statistically significant changes in alert firing patterns per rule | Alerts in `.alerts-streams.alerts-default`; detection records in `.significant_events-detections` |
| 4. Discovery and triage | An AI Investigator agent correlates active detections across streams into discoveries; an AI Judge agent reviews discoveries and promotes the most significant into events | Discoveries in `.significant_events-discoveries`; events in `.significant_events-events` |

Phases 1 and 2 run on-demand or on a continuous schedule you control. Phases 3 and 4 run continuously in the background once Significant Events is enabled.

### LLM call sites

The pipeline makes LLM calls at exactly four points:

1. **Feature identification** — analyzes log samples to extract structured KIs
2. **Query generation** — produces ES\|QL alerting rules from identified features
3. **Investigator** — correlates active detections across rules and streams into discoveries
4. **Judge** — reviews discoveries and decides which become Significant Events

Everything between these four points — rule execution, the `change_point` aggregation, state tracking, index writes — is deterministic.

## Prerequisites [sig-events-prerequisites]

Significant Events requires an **Enterprise license** or an active Enterprise trial.

To use Significant Events you also need:

- The `observability:streamsEnableSignificantEvents` {{product.kibana}} setting enabled.
- The `observability:streamsEnableSignificantEventsDiscovery` {{product.kibana}} setting enabled (required for Discovery, rule generation, and the end-to-end pipeline).
- A [Generative AI connector](kibana://reference/connectors-kibana/gen-ai-connectors.md), which routes LLM calls and incurs additional costs.

## What runs where [sig-events-components]

| Component | Runs on | Trigger | Reads | Writes |
|---|---|---|---|---|
| KI feature identification | Kibana (Task Manager + Workflows) | On-demand or continuous extraction | Stream log data | `.kibana_streams_features-*` |
| KI query generation | Kibana (Workflow) | On-demand after features exist | Feature KIs | Query KI assets on stream |
| Alerting rule execution | Kibana alerting → Elasticsearch | Per-rule schedule | Stream data via ES\|QL | `.alerts-streams.alerts-default` |
| Detection workflow | Kibana Workflows | Scheduled | `.alerts-streams.alerts-default` | `.significant_events-detections` |
| Discovery workflow | Kibana Workflows + Agent Builder | Scheduled | `.significant_events-detections` + KIs | `.significant_events-discoveries` |
| Triage workflow | Kibana Workflows + Agent Builder | Scheduled | `.significant_events-discoveries` | `.significant_events-events` |

## In this section [sig-events-nav]

- [Detection](./how-it-works/detection.md) — how alerting rules and change detection work
- [Discovery](./how-it-works/discovery.md) — how the Investigator and Judge agents operate
- [Knowledge Indicators](./how-it-works/knowledge-indicators.md) — KI extraction, Query KIs, and downstream rule promotion
- [Data model](./how-it-works/data-model.md) — index layout and ES\|QL traceability
- [Operator guide](./operator-guide.md) — system impact, cost drivers, and operational procedures
