---
applies_to:
  serverless:
    observability: preview
description: Nightshift investigations determine the root cause of a problem automatically and report their findings, with notifications in Slack.
products:
  - id: observability
  - id: cloud-serverless
---

# Investigations [nightshift-investigations]

An investigation is an automated analysis of a problem in your systems. When an investigation runs, Nightshift agents gather evidence from your data and what Nightshift already knows about your systems, then report what happened, what likely caused it, and what to look at next.

:::{note}
Nightshift is in private preview and isn't enabled by default. To request access, contact your Elastic account team.
:::

## What triggers an investigation [nightshift-investigations-triggers]

Investigations start in the following ways:

- **From an alert**: Select **Investigate** on an alert, from the alert details or the alerts table, to start an ad hoc investigation into it. If the alert already has one, select **View investigation** to open it. Triggering investigations this way isn't supported for alerts created by v2 rules yet. After you trigger an investigation, you can also find and track it from the [Nightshift home page](./nightshift.md).
- **From the Nightshift home page**: Select **Start investigation** from the Nightshift homepage to start a manual investigation.

## How investigations work [nightshift-investigations-how]

Each investigation runs as an agentic process in the background:

- **Context first**: The investigation agent starts from what Nightshift already knows about your systems — the services involved, their dependencies, and the knowledge you've added — instead of rebuilding that understanding from raw telemetry each time. Refer to [Memory](./memory.md).
- **Evidence gathering**: The agent runs targeted queries against your data to collect evidence about the problem and the services it affects.
- **Findings**: The agent synthesizes what it found into a conclusion about the likely root cause.

## Read investigation results [nightshift-investigations-output]

A completed investigation shows:

- **Summary**: The observed issue and what the investigation found.
- **Subject**: What was being investigated, for example, the alerts or significant events that triggered it.
- **Hypotheses**: The explanations the investigation considered and tested, so you can validate its conclusions.
- **Conclusion**: The best-supported explanation of why the issue occurred, with visualizations of the findings.
- **Impact**: What the issue affects, so you can prioritize it, involve the right teams, and explain it to the business.
- **Proposed actions**: Recommended next steps. Nightshift doesn't take remediation actions on your systems.

Each investigation has a severity (critical, high, medium, or low), which is used to group investigations on the Nightshift home page and in notifications.

## Manage investigations [nightshift-investigations-lifecycle]

Investigations are listed on the [Nightshift home page](./nightshift.md), grouped by severity, so you can see what's running and filter for the investigations that matter to you.

## Ask questions about an investigation [nightshift-investigations-chat]

You can ask the Nightshift agent about a specific investigation from chat — to get more detail about its findings, request alternative explanations, or steer the investigation toward something it didn't cover.

## Add custom context [nightshift-investigations-custom-context]

You can save short notes about your environment like team conventions, service ownership, known quirks, and Nightshift applies them to every investigation and every chat with the Nightshift agent. Select {icon}`boxes_vertical` → **Custom context** to add, edit, and delete context.

## Get results in Slack [nightshift-investigations-slack]

You can get notified in Slack when an investigation completes, so your on-call team receives results where they already work. Add the Nightshift investigation workflow as an action on your alert rules and choose the Slack channel to send notifications to.

<!-- DRAFT NOTE: The interim mechanism is copying a workflow template (nightshift_investigate_alert_slack_template.yaml, currently a GitHub attachment) and adding it as an action on v1 alert rules. The template needs a public home before this can be documented as a full how-to. -->

## Give feedback [nightshift-investigations-feedback]

Use the **Submit feedback** {icon}`comment` button at the top of the page to share your experience. Your feedback goes directly to the team.

## Learn more [nightshift-investigations-nav]

- [Nightshift overview](./nightshift.md): Get an overview of Nightshift, requirements, and how to get started
- [Memory](./memory.md): Learn how Nightshift stores and uses system knowledge to improve investigation quality over time
- [How Significant Events works](../streams/significant-events/how-it-works.md): Pipeline internals for KI extraction, rule generation, detection, and discovery
- [Knowledge Indicators](../streams/significant-events/knowledge-indicators.md): Get an in-depth overview of how KIs work
- [Operator guide](../streams/significant-events/operator-guide.md): Learn more about system impact, cost drivers, and operational procedures