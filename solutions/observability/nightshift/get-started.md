---
applies_to:
  serverless:
    observability: preview
description: Set up Nightshift in your Elastic Observability Serverless project and run your first investigation.
products:
  - id: observability
  - id: cloud-serverless
---

# Get started with Nightshift

This page walks you through setting up Nightshift and getting to your first investigation.

:::{note}
Nightshift is in private preview and isn't enabled by default. To request access, contact your Elastic account team.
:::

## Before you begin [nightshift-get-started-prereqs]

Make sure you meet the [requirements](./nightshift.md#nightshift-requirements): an {{obs-serverless}} project with private preview access, and data in your project or in a remote serverless project.

## Step 1: Choose the data Nightshift monitors [nightshift-get-started-data]

Tell Nightshift what to watch by defining the data it monitors with an {{esql}} query. This lets you express monitoring intent in a query language you already know — for example, scoping Nightshift to the logs of a specific set of services — without reorganizing your data first.

You can use data in your local project or in remote serverless projects. Signals and alerts from remote {{ech}} clusters aren't supported during the private preview.

After you select data, Nightshift starts learning about your systems: it extracts what services are running, what infrastructure they use, and how they depend on each other. You can review and correct this knowledge at any time.

## Step 2: Run your first investigation [nightshift-get-started-investigate]

Select **Investigate** on an alert, from the alert details or the alerts table, to start an ad hoc [investigation](./investigations.md) into it.

## Step 3: Read the results [nightshift-get-started-results]

Open the investigation from the Nightshift home page or the alert. Review what happened, the hypotheses, the conclusion, and the impact, then follow the proposed actions. Refer to [Read investigation results](./investigations.md#nightshift-investigations-output).

## Next steps [nightshift-get-started-next]

- Get notified in Slack when investigations complete by adding the Nightshift investigation workflow as an action on your alert rules. Refer to [Get results in Slack](./investigations.md#nightshift-investigations-slack).
- Review what Nightshift has learned about your systems and correct anything that's wrong. Refer to [Memory](./memory.md).
