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

Significant Events continuously converts raw log data into a ranked feed of incidents. The pipeline moves from broad to narrow: many alerting rule executions produce fewer statistical detections, which an AI agent correlates into even fewer discoveries, which a judge reviews and surfaces as Significant Events.

For example: you onboard a stream, Significant Events extracts what's in it ("this stream contains a Python gRPC service called `recommendationservice` running on GKE"), generates targeted detection rules ("detect gRPC connection failures"), and runs those rules continuously. Results flow through discovery and surface as correlated Significant Events across your entire environment.

## Before you get started [sig-events-prerequisites]

Significant Events requires an **Enterprise license** or an active Enterprise trial.

To use Significant Events you also need:

- The `observability:streamsEnableSignificantEvents` {{kib}} setting enabled.
- The `observability:streamsEnableSignificantEventsDiscovery` {{kib}} setting enabled (required for Discovery, rule generation, and the end-to-end pipeline).
- A [Generative AI connector](kibana://reference/connectors-kibana/gen-ai-connectors.md), which routes LLM calls and incurs additional costs.

## How the pipeline works [sig-events-pipeline]

The pipeline runs in sequential phases that build off of the output of the previous phase. KI extraction and rule generation prepare your streams for detection and run on your schedule. Detection and discovery run continuously in the background after you enable Significant Events.

### 1. Knowledge indicator extraction [sig-events-phase-ki]

An LLM analyzes log samples from your streams and identifies services, infrastructure, and failure patterns as Knowledge Indicators (KIs).

### 2. Rule generation [sig-events-phase-rules]

An LLM converts query KIs into {{esql}} alerting rules and promotes them to the alerting framework. Generated rules are stored as stream assets.

### 3. Rule execution and detections [sig-events-phase-detection]

Promoted alerting rules run against stream data on a per-rule schedule. A detection workflow identifies statistically significant transitions or changes in alert firing patterns that signal something meaningful has happened. Significant transitions are written as detection records.

### 4. Discovery [sig-events-phase-discovery]

Two LLM agents run in sequence, the investigator agent and the judge agent. The investigator agent reads unprocessed detection records, correlates them across rules and streams, and writes discovery records. The judge agent then evaluates each discovery and promotes the most significant into Significant Events.

## Further reading [sig-events-nav]

Refer to the following pages for more information on Significant Events.

- [Operator guide](./operator-guide.md) — system impact, cost drivers, and operational procedures
- [Detection](./how-it-works/detection.md) — how alerting rules and change detection work
- [Discovery](./how-it-works/discovery.md) — how the investigator and judge agents operate
- [Knowledge Indicators](./how-it-works/knowledge-indicators.md) — KI extraction, Query KIs, and downstream rule promotion
- [Data model](./how-it-works/data-model.md) — index layout and ES|QL traceability
