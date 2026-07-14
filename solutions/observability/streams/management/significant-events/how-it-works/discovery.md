---
navigation_title: Discovery
description: How the Significant Events Investigator and Judge agents correlate detections across streams into prioritized incidents.
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

# Discovery [sig-events-discovery]

Discovery is the fourth phase of the Significant Events pipeline. Two AI agents run sequentially: the Investigator correlates active detections across streams and rules, and the Judge evaluates candidate discoveries and promotes the most significant into events.

Both agents run as Kibana Workflows using the Agent Builder platform.

## The Investigator [sig-events-investigator]

The Investigator workflow scans `.significant_events-detections` for unprocessed detections and batches them for analysis.

### How batching works

The workflow queries the detections index for change-point detections that:

- Fall within the last 24 hours (`now-24h`)
- Have not yet been processed (no `processed_by` marker document)
- Represent the most recent detection per rule (results are collapsed by `rule_uuid`)

Results are sorted by timestamp descending and p-value ascending, so the most recent and most statistically significant detections are prioritized. The batch is capped at 10 rules per invocation. If more unprocessed detections exist after a cycle, they are picked up in the next run.

For each rule in the batch, the workflow also collects the full backlog of unprocessed detection documents (not just the collapsed newest), so that all of them can be stamped as processed once the agent accounts for that rule.

### What the agent receives and returns

The Investigator agent receives the detection batch, which includes for each detection: `detection_id`, `rule_name`, `rule_uuid`, `stream_name`, `change_point_type`, and `p_value`. The agent writes discovery records directly to `.significant_events-discoveries` via an internal tool.

The agent returns a list of `written_rule_uuids` — the `rule_uuid` of every rule it accounted for in this cycle. The workflow uses this list to stamp all corresponding detection documents with a `processed_by` marker, preventing them from being re-processed in future cycles.

A detection is not re-processed if a `processed_by` marker already exists for it, even if the marker was written by a different execution.

### Concurrency

The discovery workflow uses a drop concurrency strategy with a maximum of one concurrent execution. If a new cycle triggers while a previous one is still running, the new trigger is silently dropped. The workflow has a 20-minute timeout.

## The Judge [sig-events-judge]

The Judge workflow runs after the Investigator and evaluates unreviewed discoveries.

### How batching works

The Judge queries `.significant_events-discoveries` for the most recent document per `discovery_slug` where the latest document is not yet `kind: handled`. It sorts by timestamp descending and criticality descending, and caps the batch at 10 discoveries per invocation. The lookback window is the last 7 days.

### What the agent receives and returns

The Judge agent receives the full discovery documents, which include: `title`, `summary`, `root_cause`, `criticality`, `confidence`, `cause_kis`, `evidences`, `detections`, `dependency_edges`, `infra_components`, and `stream_names`.

For each discovery, the Judge decides one of four statuses:

| Status | Meaning |
|---|---|
| `promoted` | The discovery represents a genuine incident; create a Significant Event |
| `acknowledged` | Noted but not promoted |
| `demoted` | Not significant; suppress |
| `resolved` | The underlying condition has cleared |

The Judge writes event records directly to `.significant_events-events` via an internal tool and stamps the corresponding discovery as `kind: handled`.

### Triggering investigation

For each discovery the Judge promotes, the workflow asynchronously triggers a `system-significant-events-investigation` workflow. This investigation runs independently and does not block the triage cycle. If a discovery is re-promoted (same `discovery_slug`), the concurrency key on the investigation workflow cancels any in-progress investigation for that slug before starting a new one.

### Concurrency

The triage workflow uses a drop concurrency strategy with a maximum of one concurrent execution. The timeout is 30 minutes.

## Relationship to orchestration [sig-events-orchestration]

Detection, discovery (Investigator), and triage (Judge) run as three separate workflows sequenced by an orchestrator. The orchestrator runs them in order: detection first, then discovery, then triage. If one sub-workflow fails, the orchestrator continues to the next. Each sub-workflow handles its own idle-skip logic — if there is nothing to process, it exits early without error.

<!-- NEEDS: verify exact cron intervals against scheduled_detection.yaml and scheduled_review.yaml in kbn-workflows before publishing. Based on the July 2026 architecture design, the detection workflow runs approximately every 30 minutes and the triage/review workflow runs approximately every 10 minutes. The "~1m to first detection" latency estimate in the operator guide refers to how quickly an alerting rule first fires, not the detection workflow cadence. -->
