---
applies_to:
  serverless:
    observability: preview
description: Elastic Nightshift AI SRE investigations determine the root cause of a problem automatically and report their findings, with notifications in Slack.
products:
  - id: observability
  - id: cloud-serverless
---

# Investigations [nightshift-investigations]

An investigation is an automated analysis of an issue in your systems. When an investigation runs, Elastic Nightshift AI SRE agents gather evidence from your data and what AI SRE already knows about your systems, then report what happened, what likely caused it, and what to look at next.

:::{note}
AI SRE is in private preview and isn't enabled by default. To request access, contact your Elastic account team.
:::

## What triggers an investigation [nightshift-investigations-triggers]

Investigations start in the following ways:

- **From an alert**: Select **Investigate** on an alert, from the alert details or the alerts table, to start an ad hoc investigation into it. If the alert already has one, select **View investigation** to open it. Triggering investigations this way isn't supported for alerts created by v2 rules yet. After you trigger an investigation, you can also find and track it from the [home page](./nightshift.md).
- **From a significant event**: Open a significant event's details and select **Run investigation** start an investigation manually.
- **From the home page**: Select **Start investigation** from the Elastic Nightshift AI SRE home page and enter a prompt to start a manual investigation.

## How investigations work [nightshift-investigations-how]

Each investigation runs as an agentic process in the background:

- **Context first**: The investigation agent starts from what AI SRE knows about your systems like the services involved, their dependencies, and any other knowledge you've added. Refer to [Memory](./memory.md) for more information on adding context about your system and processes.
- **Evidence gathering**: The agent runs targeted queries against your data to collect evidence about the problem and the services it affects.
- **Findings**: The agent synthesizes what it found into a conclusion about the likely root cause.

## Read investigation results [nightshift-investigations-output]

A completed investigation shows:

- **What happened**: A short factual summary of the observed issue and what the investigation found.
- **Impact**: What the issue affects, including a chart of the affected activity, so you can prioritize it and involve the right teams.
- **Subject**: What was being investigated, for example, the alert or signficiant event that triggered it.
- **Investigation**: The hypotheses the investigation tested, each with a confidence score and the evidence and charts that support it, so you can validate its conclusions.
- **Conclusion**: The best-supported explanation of why the issue occurred.
- **Proposed actions**: Recommended next steps, with the most direct fix tagged **Recommended**. Select an action to see why it's suggested and copy any commands it includes.

Each investigation has a severity (critical, high, medium, or low), which is used to group investigations on the home page.

## Manage investigations [nightshift-investigations-lifecycle]

Investigations are listed on the [Elastic Nightshift AI SRE home page](./nightshift.md), grouped by severity, with sections for in-progress and failed or canceled runs, so you can see what's running and find the investigations that matter to you. Each investigation shows its status, such as **Running** or **Completed**.

## Ask questions about an investigation [nightshift-investigations-chat]

You can ask the AI SRE agent about a specific investigation from chat — to get more detail about its findings, request alternative explanations, or steer the investigation toward something it didn't cover.

## Add custom context [nightshift-investigations-custom-context]

You can save short notes about your environment like team conventions, service ownership, known quirks, and AI SRE applies them to every investigation and every chat with the AI SRE agent. Select {icon}`boxes_vertical` → **Custom context** to add, edit, and delete context.

## Give feedback [nightshift-investigations-feedback]

Use the **Submit feedback** {icon}`comment` button at the top of the page to share your experience. Your feedback goes directly to the team.

## Learn more [nightshift-investigations-nav]

- [Elastic Nightshift AI SRE overview](./nightshift.md): Get an overview of Elastic Nightshift AI SRE, requirements, and how to get started
- [Memory](./memory.md): Learn how AI SRE stores and uses system knowledge to improve investigation quality over time
- [How Significant Events works](../streams/significant-events/how-it-works.md): Pipeline internals for KI extraction, rule generation, detection, and discovery
- [Knowledge Indicators](../streams/significant-events/knowledge-indicators.md): Get an in-depth overview of how KIs work
- [Operator guide](../streams/significant-events/operator-guide.md): Learn more about system impact, cost drivers, and operational procedures
