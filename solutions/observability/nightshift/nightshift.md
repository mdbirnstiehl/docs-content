---
applies_to:
  serverless:
    observability: preview
description: Nightshift provides automated investigations and on-call assistance for Elastic Observability, triggered by your alerts.
products:
  - id: observability
  - id: cloud-serverless
---

# Nightshift

Nightshift is an AI SRE built into Elastic {{observability}}. It learns how your systems work, watches your alerts, investigates problems, and delivers findings where your team already works, such as Slack.

Nightshift is built from the following engines:

1. **Context engine**: Nightshift extracts knowledge about your systems from your data: which services are running, what infrastructure they use, and how they depend on each other. This knowledge is stored as [Knowledge Indicators](../streams/significant-events/knowledge-indicators.md). You can review and correct what Nightshift knows at any time, and add knowledge of your own for investigations to use.
2. **Detection engine**: Nightshift generates detection rules from its knowledge of your systems and runs them continuously. When it finds something meaningful, it surfaces a significant event as an alert, so you can manage and route it like any other Elastic alert.
3. **Investigation engine**: When an alert fires — whether it's one of your existing alert rules or a significant event Nightshift detected — it can trigger an [investigation](./investigations.md). The investigation gathers evidence, determines the likely root cause, and reports its findings. You can also trigger investigations manually.

Nightshift doesn't take remediation actions on your systems. Investigation findings and suggested next steps are informational only.

## Requirements [nightshift-requirements]

- **An {{obs-serverless}} project**: Nightshift runs on {{serverless-full}} during the private preview. It isn't available on self-managed or {{ech}} deployments.
- **Private preview access**: Nightshift must be enabled for your project. Contact your Elastic account team to request access.
- **Data to monitor**: Nightshift works with the data you already have — in your local project, or in remote serverless projects connected through cross-project search. You don't need to select data or set anything up to start investigating.

Nightshift uses the [Elastic {{infer-cap}} Service (EIS)](/explore-analyze/elastic-inference/eis.md). You don't need to configure an LLM connector or select a model.

### Private preview limitations [nightshift-preview-limitations]

- Signals and alerts from remote {{ech}} clusters aren't supported.
- Connecting external knowledge sources, such as GitHub, isn't available yet.
- The Nightshift Slack app isn't available yet.

## Nightshift UI [nightshift-landing-page]

The Nightshift home page is your central view of your Nightshift investigations. It shows your [investigations](./investigations.md) grouped by severity, so you can see what's being investigated and what needs your attention.

From the home page you can:

- Select an investigation and see more details about its findings.
- Select **Start investigation** to trigger a manual investigation.
- Select **Show all events** to view significant events.
- Review and edit what Nightshift knows about your systems.

## Chat with Nightshift [nightshift-chat]

You can ask the Nightshift agent about a specific investigation from chat — to get more detail, request alternative explanations, or steer the investigation. Refer to [Investigations](./investigations.md).

## Give feedback [nightshift-feedback]

Use the **Submit feedback** {icon}`comment` button at the top of the page to share your experience. Your feedback goes directly to the team.

## Learn more [nightshift-landing-page-nav]

- [Investigations](./investigations.md): Learn how Nightshift investigates problems and how to read the results
- [Memory](./memory.md): Learn how Nightshift stores and uses system knowledge to improve investigation quality over time
- [Significant Events](../streams/significant-events/index.md): Get an overview of how Nightshift detects significant events in your data
- [How Significant Events works](../streams/significant-events/how-it-works.md): Pipeline internals for KI extraction, rule generation, detection, and discovery
- [Knowledge Indicators](../streams/significant-events/knowledge-indicators.md): Get an in-depth overview of how KIs work
- [Operator guide](../streams/significant-events/operator-guide.md): Learn more about system impact, cost drivers, and operational procedures
