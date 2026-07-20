---
navigation_title: Nightshift
applies_to:
  serverless: preview
description: Nightshift automatically detects, investigates, and helps you remediate observability incidents using AI.
products:
  - id: observability
---

# Nightshift [nightshift]

Nightshift is an AI-powered observability product built into Elastic. It analyzes your telemetry, automatically detects what matters, and surfaces actionable intelligence so your team can spend less time finding problems and more time fixing them.

Nightshift works by:

1. **Extracting Knowledge Indicators (KIs)**: Nightshift reads your stream data and extracts stable facts: what services are running, what infrastructure is in use, what technologies are present, and how they relate to each other.
2. **Generating and running detection rules**: Based on KIs, Nightshift generates ES|QL detection rules and runs them continuously. When rule firing patterns change, it detects the change.
3. **Surfacing Significant Events**: When detections correlate into something meaningful, Nightshift promotes a Significant Event — a concise summary of what's happening, with severity, evidence, and suggested next steps.
4. **Investigating automatically**: For each Significant Event, Nightshift triggers an autonomous [investigation](nightshift-investigations.md): it reads your system memory, runs targeted queries, and produces a root cause hypothesis with remediation options.
5. **Learning over time**: The [memory](nightshift-memory.md) system captures knowledge from each incident so future investigations are faster and more precise.

## Requirements [nightshift-requirements]

- **Enterprise license**: Nightshift requires an active Elastic Enterprise license or trial on {{product.serverless-elasticsearch}}.
- **Streams**: Your data must be organized in [Streams](/solutions/observability/streams/streams.md). Nightshift attaches to streams and runs the KI extraction and detection pipeline per stream.

## Private beta features [nightshift-private-beta-features]

| Feature | Description |
|---------|-------------|
| [Nightshift landing page](nightshift-landing-page.md) | The Nightshift home in {{kib}}: a live list of open Significant Events, with filtering, detail views, and one-click chat. |
| [Significant Events and Knowledge Indicators](../streams/management/significant-events.md) | How Nightshift extracts KIs, generates detection rules, and surfaces Significant Events. |
| [Investigations](nightshift-investigations.md) | Autonomous root cause analysis triggered on every Significant Event — reads memory, runs queries, proposes remediation. |
| [Memory](nightshift-memory.md) | A persistent knowledge base about your systems that makes investigations faster and more accurate over time. |
| [Code Intelligence](nightshift-code-intelligence.md) | Connect your source code repositories so Nightshift can reason about code as a first-class signal alongside telemetry. |

## Get started [nightshift-get-started]

1. Enable Nightshift from **Stack Management** → **Nightshift settings**. This requires an Enterprise license.
2. Nightshift begins extracting KIs from your streams and generating detection rules automatically — no manual configuration is required.
3. Navigate to the [Nightshift landing page](nightshift-landing-page.md) to see your first Significant Events as they surface.
4. (Optional) Run the [onboarding interview](nightshift-memory.md#nightshift-memory-onboarding) to teach Nightshift about your system so investigations start with richer context.
5. (Optional) [Connect a code repository](nightshift-code-intelligence.md#nightshift-code-intelligence-connect) to enable code-aware investigations.