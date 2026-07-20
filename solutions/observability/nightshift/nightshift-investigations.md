---
navigation_title: Investigations
applies_to:
  serverless: preview
description: Nightshift automatically investigates every Significant Event, producing a root cause hypothesis, supporting evidence, and remediation suggestions.
products:
  - id: observability
---

# Investigations [nightshift-investigations]

:::{note}
Nightshift is currently in private beta. Access is limited and by invitation only.
:::

% TODO: Link "Significant Event" to the sig events doc once path is confirmed.
When Nightshift surfaces a Significant Event, it immediately begins an autonomous investigation. The investigation attempts to determine the root cause, assess the blast radius, and propose remediation options — all without requiring manual intervention.

## How investigations work [nightshift-investigations-how]

Each investigation runs as a structured, agentic process:

1. **Memory read** — Before querying raw telemetry, the investigation agent reads relevant [memory pages](nightshift-memory.md) about your system. If Nightshift has previously encountered similar issues or has context about the affected service, it uses that knowledge as a starting point.

2. **Targeted queries** — The agent runs targeted ES|QL queries against your streams to gather evidence about the event — error rates, service dependencies, infrastructure state, and related signals.

3. **External tool calls** — If external connectors are configured (such as GitHub, Slack, or cloud provider APIs), the agent calls those tools to gather additional context, such as recent deployments, open incidents, or code changes.

4. **Code context** — If [Code Intelligence](nightshift-code-intelligence.md) is connected, the agent can query recent commits, pull requests, and code history as first-class signals.

5. **Hypothesis and remediation** — The agent synthesizes all evidence into a root cause hypothesis with a confidence assessment. It also proposes ranked remediation options where it can determine them.

6. **Memory write-back** — New findings — such as failure patterns or service relationships discovered during the investigation — are written back to [memory](nightshift-memory.md) so future investigations benefit from this context.

## What an investigation produces [nightshift-investigations-output]

A completed investigation provides:

- **Root cause hypothesis** — A plain-language explanation of what Nightshift believes caused the Significant Event, with the evidence supporting that conclusion.
- **Confidence level** — How confident the investigation is in its hypothesis, based on the quality and completeness of available evidence.
- **Blast radius** — The services, infrastructure components, and dependencies that are affected or at risk.
- **Remediation options** — Suggested next steps, ranked by feasibility and impact. These are textual suggestions only — Nightshift does not take automated remediation actions.
- **Evidence trail** — The specific signals, log patterns, and data points the investigation relied on, so you can verify its reasoning.

:::{note}
When the root cause is behind an access boundary Nightshift cannot reach — for example, a cloud infrastructure setting or an internal system with no connector — the investigation says so explicitly and stops rather than speculating. A clean hand-off to a human operator is by design.
:::

## Where to view investigations [nightshift-investigations-viewing]

### From the Nightshift landing page [nightshift-investigations-landing-page]

Each Significant Event on the [landing page](nightshift-landing-page.md) shows its investigation status. Click the event to open the detail view, then expand the **Investigation** section to see results.

### From chat [nightshift-investigations-chat]

Click **Chat** on any Significant Event to open an AI conversation with the investigation context pre-attached. You can:

- Read the full investigation narrative.
- Ask follow-up questions about specific evidence.
- Request alternative hypotheses.
- Explore remediation options in more depth.

## Triggering investigations [nightshift-investigations-triggering]

Nightshift automatically triggers an investigation for every promoted Significant Event. You can also trigger an investigation from chat for any Significant Event — useful if you want to re-run an investigation after connecting additional data sources or after [memory](nightshift-memory.md) has been updated.

Investigations can also be triggered programmatically via API or workflow for any alert — not just Significant Events. In this case, the investigation output is not yet surfaced in the standard user flows; this capability is intended for early exploration and feedback.

## Investigation states [nightshift-investigations-states]

| State | Meaning |
|-------|---------|
| **Running** | The investigation agent is actively gathering evidence and building its hypothesis. |
| **Completed** | The investigation has finished and results are available. |

## Relationship to memory [nightshift-investigations-memory]

Investigations and [memory](nightshift-memory.md) form a feedback loop: investigations read memory to start from a richer baseline, and write new findings back to memory when they're done. Over time, this means investigations for recurring issue types become progressively faster and more accurate as Nightshift accumulates knowledge about your environment.

## In this section [nightshift-investigations-nav]

- [Nightshift overview](index.md)
- [Landing page](nightshift-landing-page.md)
- [Memory](nightshift-memory.md)
- [Code Intelligence](nightshift-code-intelligence.md)
