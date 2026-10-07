---
applies_to:
  serverless:
    observability: preview
description: Elastic Nightshift AI SRE memory is a knowledge base of facts about your systems that investigations use to produce better results.
products:
  - id: observability
  - id: cloud-serverless
---

# Memory [nightshift-memory]

In Elastic Nightshift AI SRE, **Memory** is a knowledge base of facts about your systems that agents read when investigating incidents. Rather than rebuilding an understanding of your environment from raw telemetry on every [investigation](./investigations.md), the AI SRE keeps knowledge about your services, infrastructure, dependencies, and known failure patterns, and uses it as a starting point.

:::{note}
Elastic Nightshift AI SRE is in private preview and isn't enabled by default. To request access, contact your Elastic account team.
:::

## View and edit memory [nightshift-memory-manage]

To see what the AI SRE knows about your systems, go to **Nightshift** → **Management** → **Cortex**. Review the existing pages to confirm the information is correct for your environment. To correct a page, select it, then select **Edit**.

You can also add information about your systems and processes that the AI SRE might not discover from your data, such as team conventions, deployment processes, or known issues. To add a page, select **New page** {icon}`plus` and enter the information you want investigations to use.

## How investigations use memory [nightshift-memory-usage]

When an [investigation](./investigations.md) starts, the investigation agent reads the knowledge relevant to the affected services before querying your telemetry. This means the agent starts with an understanding of your environment, and investigations produce more accurate conclusions than they would from raw data alone.

Investigations can't currently update or create memory pages.

## Give feedback [nightshift-memory-feedback]

Use the **Submit feedback** {icon}`comment` button at the top of the page to share your experience. Your feedback goes directly to the team.

## Learn more [nightshift-memory-nav]

- [Elastic Nightshift AI SRE](./nightshift.md): Get an overview of the AI SRE and its requirements
- [Investigations](./investigations.md): Learn how the AI SRE investigates problems and how to read the results
- [How Significant Events works](../streams/significant-events/how-it-works.md): Pipeline internals for KI extraction, rule generation, detection, and discovery
- [Knowledge Indicators](../streams/significant-events/knowledge-indicators.md): Get an in-depth overview of how KIs work
- [Operator guide](../streams/significant-events/operator-guide.md): Learn more about system impact, cost drivers, and operational procedures
