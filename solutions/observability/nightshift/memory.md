---
applies_to:
  serverless:
    observability: preview
description: Nightshift memory is a knowledge base of facts about your systems that investigations use to produce better results.
products:
  - id: observability
  - id: cloud-serverless
---

# Memory [nightshift-memory]

Nightshift **Memory** is a knowledge base of facts about your systems that agents read when investigating incidents. Rather than rebuilding an understanding of your environment from raw telemetry on every [investigation](./investigations.md), Nightshift keeps knowledge about your services, infrastructure, dependencies, and known failure patterns, and uses it as a starting point.

:::{note}
Nightshift is in private preview and isn't enabled by default. To request access, contact your Elastic account team.
:::

## View and edit what Nightshift knows [nightshift-memory-manage]

You can review the knowledge Nightshift has about your systems by going to **Nightshift** → **Management** → **Cortex**. From here, you can review existing pages to confirm the information is correct for your environment. Select a page and select **Edit** to update any incorrect information about your systems or processes.

You can also add information about your systems and processes that Nightshift might not discover from your data, like team conventions, deployment processes, or known issues. To add a page, Select **New page** {icon}`plus` and add .

## How investigations use memory [nightshift-memory-usage]

When an [investigation](./investigations.md) starts, the investigation agent reads the knowledge relevant to the affected services before querying your telemetry. This means the agent starts with an understanding of your environment, and investigations produce more accurate conclusions than they would from raw data alone.

Investigation can't currently update or create memory pages.

## Give feedback [nightshift-memory-feedback]

Use the **Submit feedback** {icon}`comment` button at the top of the page to share your experience. Your feedback goes directly to the team.

## Learn more [nightshift-memory-nav]

- [Nightshift overview](./nightshift.md): Get an overview of Nightshift, requirements, and how to get started
- [Investigations](./investigations.md): Learn how Nightshift investigates problems and how to read the results
- [How Significant Events works](../streams/significant-events/how-it-works.md): Pipeline internals for KI extraction, rule generation, detection, and discovery
- [Knowledge Indicators](../streams/significant-events/knowledge-indicators.md): Get an in-depth overview of how KIs work
- [Operator guide](../streams/significant-events/operator-guide.md): Learn more about system impact, cost drivers, and operational procedures