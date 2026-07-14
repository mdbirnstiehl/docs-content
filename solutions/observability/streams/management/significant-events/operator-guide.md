---
navigation_title: Operator guide
description: System impact, cost drivers, and operational procedures for cluster operators running Significant Events.
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

# Operator guide [sig-events-operator]

This page covers what Significant Events does to your cluster, what to monitor, and how to disable, re-enable, or recover from a degraded state.

## What runs where [sig-events-op-components]

| Component | Runs on | Trigger | Reads | Writes |
|---|---|---|---|---|
| KI feature identification | Kibana (Task Manager + Workflows) | On-demand or continuous extraction schedule | Stream log data | `.kibana_streams_features-*` |
| KI query generation | Kibana (Workflow) | On-demand after features exist | Feature KIs | Query KI assets on stream |
| Alerting rule execution | Kibana alerting → Elasticsearch | Per-rule schedule | Stream data via ES\|QL | `.alerts-streams.alerts-default` |
| Detection workflow | Kibana Workflows | Scheduled | `.alerts-streams.alerts-default` | `.significant_events-detections` |
| Discovery workflow | Kibana Workflows + Agent Builder | Scheduled | `.significant_events-detections` + KIs | `.significant_events-discoveries` |
| Triage workflow | Kibana Workflows + Agent Builder | Scheduled | `.significant_events-discoveries` | `.significant_events-events` |

## System impact [sig-events-op-impact]

These are observable signals, not hard guarantees. Actual behavior depends on your data volume, stream count, and cluster sizing.

**Alerting rule query load**

One ES\|QL alerting rule runs per promoted Query KI. Each rule fires on its own schedule. A `change_point` aggregation runs across all active rules' alert results on each detection cycle. Alert volume in `.alerts-streams.alerts-default` grows proportionally to the number of promoted rules and the rate at which they fire.

**Pipeline lag**

The time from a log event to a Significant Event in the UI depends on how quickly each phase completes:

- **Detection**: change-point records appear in `.significant_events-detections` after the detection workflow processes new alerts. The detection workflow uses a 40-minute lookback window.
- **Discovery**: unprocessed detections are picked up by the discovery workflow on its next run. The discovery workflow batches up to 10 rules per cycle.
- **Triage**: new discoveries are evaluated by the Judge on the triage workflow's next run. The triage workflow batches up to 10 discoveries per cycle.

<!-- NEEDS: verify exact cron intervals against scheduled_detection.yaml and scheduled_review.yaml in kbn-workflows before publishing. Based on the July 2026 architecture design: the detection workflow runs approximately every 30 minutes; the triage/review workflow runs approximately every 10 minutes. The "~1m to first detection" figure refers to how quickly an alerting rule first fires, not the detection workflow cadence. -->

**Kibana memory**

The Investigator and Judge agents run as Agent Builder tool chains within Kibana. Agent runs add memory pressure. Observe Kibana memory under realistic load; no committed sizing numbers exist until your specific workload is measured.

**Ingest assumptions**

Streams must have recent log data for KI extraction to produce useful results. Empty streams or streams with very sparse data produce weak KIs, which produces weak Query KIs, which produces fewer promoted rules.

## Cost drivers [sig-events-op-costs]

LLM costs scale with the number of streams, the number of promoted rules, and whether continuous extraction is enabled. The pipeline has four LLM call sites with different cost profiles:

| Phase | Cadence | Cost profile |
|---|---|---|
| Feature identification | Per stream onboarding + optional continuous (up to 5 streams per 10-minute run) | Highest token volume; uses a fast classification model |
| Query generation | Per stream after features exist | Medium; requires reasoning and ES\|QL validation |
| Investigator | Each discovery cycle when unprocessed detections exist | Bursty; scales with the number of active alerting rules firing |
| Judge | Each triage cycle | Lower frequency; scales with the number of open discoveries |

Continuous extraction is the largest cost multiplier. When enabled, feature identification runs on a recurring schedule across all eligible streams. Enabling it on a large number of streams significantly increases token consumption.

<!-- NEEDS: free-token mechanics are undefined as of July 2026 (budget amount, cap behavior, overrun monitoring, and admin-facing messaging are an open product decision — issue #155). Do not publish cost guidance until this is resolved. -->

Do not use model names or model-specific pricing as a basis for cost estimates. Model selection may change across releases.

## Disable and re-enable [sig-events-op-disable]

### Disable Significant Events entirely

Set `observability:streamsEnableSignificantEvents` to `false` in Kibana settings. This stops the background pipeline.

<!-- NEEDS: a single-click disable control (issue #426) is planned for M1 that stops all background workflows and rules without requiring a Kibana settings change. Document the exact navigation path once it ships. -->

### Stop background KI refresh only

To stop continuous extraction without disabling Significant Events:

1. Navigate to **Significant Events** → **Settings**.
2. Under **Continuous KI extraction**, turn off **Enable continuous KI extraction**.

Disabling continuous extraction cancels all in-flight feature identification tasks and force-deletes the continuous extraction workflow. Already-extracted KIs are not deleted. Manually-triggered extractions continue to work.

### Re-enable continuous extraction

Re-enabling continuous extraction creates a fresh workflow from the current definition. Workflow state from previous runs is not preserved. Streams that were mid-extraction when continuous extraction was disabled will be re-queued on the next eligible run.

To increase the interval between extraction runs or reduce the number of streams processed, adjust:

- **Extraction interval** (`observability:streamsContinuousKiExtractionIntervalHours`, default 12 hours) — increase to reduce how often each stream is re-processed
- **Excluded streams** (`observability:streamsContinuousKiExtractionExcludedStreamPatterns`) — add glob patterns to exclude high-volume or low-value streams

Setting the extraction interval to `0` bypasses the per-stream interval gate entirely and allows streams to be reprocessed on every run. This is useful for debugging but not for production use.

## Recovery procedures [sig-events-op-recovery]

### KI extraction running constantly

**Symptom**: Continuous extraction appears to be running continuously or processing streams more frequently than expected.

**Action**: Check the continuous extraction setting. If enabled, verify the **Extraction interval** is set to an appropriate value (default is 12 hours). Check the number of streams eligible for extraction — more streams means more runs per cycle.

### High LLM cost

**Symptom**: Unexpectedly high token usage or inference costs.

**Action**:

1. Disable continuous extraction first — this is the primary cost multiplier.
2. Check how many streams are eligible for continuous extraction. Use the **Excluded streams** setting to narrow the scope.
3. If costs remain high after disabling continuous extraction, check the number of promoted Query KIs. Each promoted KI adds an alerting rule, which feeds the Investigator on every discovery cycle.

### Incidents not clearing

**Symptom**: A Significant Event remains open after the underlying condition appears to have resolved.

**Action**: This is expected behavior. The Judge must independently verify that the underlying condition has resolved. The Judge re-evaluates open discoveries on each triage cycle. Detections clearing alone is not sufficient — the Judge reads lifecycle state from the alerts index and makes an explicit `resolved` decision. Allow at least one full triage cycle after the underlying alerts clear before expecting the Significant Event to resolve.

### Recovering from a degraded state

If workflows or tasks are stuck or producing unexpected results:

1. Disable continuous extraction to stop background KI refresh.
2. Wait for in-flight tasks to reach a terminal state before taking further action.
3. Re-enable continuous extraction with a higher extraction interval or narrower stream exclusions to reduce load.

For cluster-level index resets or internal runbook procedures, contact Elastic Support. Internal purge scripts and `agent-builder` CLI commands are not documented here.
