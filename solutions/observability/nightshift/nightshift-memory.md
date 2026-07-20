---
navigation_title: Memory
applies_to:
  serverless: preview
description: Nightshift memory is a persistent knowledge base about your systems that makes investigations faster and more accurate over time.
products:
  - id: observability
---

# Memory [nightshift-memory]

:::{note}
Nightshift is currently in private beta. Access is limited and by invitation only.
:::

Nightshift memory is a persistent knowledge base that agents read before investigating incidents and write to as they learn. Rather than re-deriving knowledge from raw telemetry on every [investigation](nightshift-investigations.md), Nightshift maintains a curated wiki of facts about your systems — services, deployment processes, infrastructure, known failure patterns, and more.

Over time, memory grows richer with each incident, making investigations progressively faster and more precise.

## What memory stores [nightshift-memory-what]

Memory is organized into wiki pages. Each page covers a topic relevant to your environment, such as:

- Service architecture and dependencies
- Deployment and CI/CD processes
- Infrastructure (cloud providers, Kubernetes clusters, regions)
- Known failure modes and past incident patterns
- On-call runbooks and escalation paths
- Code repository structure and ownership

Pages use categories to organize knowledge hierarchically (for example, `services/checkout-service` or `infrastructure/kubernetes`). Agents can search, browse, and read these pages during investigations.

## How memory is built [nightshift-memory-how-built]

Memory is populated automatically by four background workflows and through the onboarding interview.

### Memory synthesis [nightshift-memory-synthesis]

% TODO: Link "Knowledge Indicators" and "Significant Events" to the sig events doc once path is confirmed.
The Memory Synthesis workflow runs when triggered and builds wiki pages by selectively querying available information sources — Knowledge Indicators, Significant Events, existing memory pages, and connected external tools (GitHub, Slack, cloud APIs). The agent decides which sources to consult based on what's most relevant, then synthesizes the results into coherent pages organized around services, infrastructure, and operations.

### Memory consolidation [nightshift-memory-consolidation]

The Memory Consolidation workflow runs automatically every 24 hours. It reviews the full wiki for duplicates, stale entries, and disorganized content, then merges duplicates, removes outdated pages, improves categories, and adds cross-references between related topics. Consolidation only reorganizes — it doesn't invent new facts.

### Conversation scraper [nightshift-memory-scraper]

The Conversation Scraper runs every 4 hours and reads recent AI assistant conversations. It extracts durable, reusable knowledge — architectural facts, operational patterns, troubleshooting steps discovered during conversations — and writes those back to memory as wiki pages. Ephemeral content (greetings, dead-end debugging threads) is skipped.

### Gap detection [nightshift-memory-gap-detection]

Gap Detection audits the entire knowledge base against eleven required knowledge dimensions and identifies what's missing. It runs weekly on a schedule and can also be triggered manually from the Memory tab using the **Detect Gaps** button.

The eleven dimensions it checks are:

1. Services and applications
2. Deployment processes (CI/CD, rollout strategies, feature flags)
3. Infrastructure (cloud, Kubernetes, VMs, regions)
4. Observability coverage (what data flows into Elastic, what's missing)
5. Administrative controls (change freezes, approval processes)
6. Health-checking and on-call (dashboards, runbooks, escalation paths)
7. Known failure modes (past incidents, postmortems, recurring issues)
8. External tools and integrations (deployment APIs, CMDB, incident management)
9. Code repositories (structure, branching, ownership)
10. Data and request flows (end-to-end paths, queues, databases)
11. Access points and connectors (dashboards, wikis, runbooks)

After each run, Gap Detection writes a structured overview page listing coverage per dimension, the top gaps, and suggested next steps. This page seeds the onboarding interview.

## Onboarding interview [nightshift-memory-onboarding]

The onboarding interview is a conversational agent that populates memory with knowledge about your specific environment at setup time. Rather than presenting a configuration form, the agent asks targeted questions about your system to fill the gaps most important for accurate investigations.

To start the onboarding interview, open the **Memory** tab in Nightshift settings and select **Tell us about your system**.

The interview covers:

- Deployment topology and infrastructure
- Code repositories (structure, branching, ownership)
- System flows (end-to-end request paths, async queues, external dependencies)
- Access and resources (dashboards, runbooks, wikis, alert routing)

Before asking questions, the interview reads the current gap overview (if Gap Detection has already run) and focuses its questions on the highest-priority gaps. Dimensions that are already well-covered in memory are skipped.

:::{tip}
Run Gap Detection before the onboarding interview for the most targeted session. Gap Detection identifies exactly what's missing, and the interview then focuses on filling those gaps.
:::

## Managing memory [nightshift-memory-managing]

The Memory tab in Nightshift settings shows all current memory pages. From there you can:

- Browse pages by category.
- Search pages by keyword.
- View individual page content and version history.
- Run Memory Synthesis manually.
- Run Gap Detection manually and view the gap overview.

Memory pages are stored as append-only records in {{product.elasticsearch}} — every change is tracked and the full history of any page is queryable. Nothing is permanently deleted; soft-deleted pages are excluded from retrieval but remain in the audit trail.

## How agents use memory [nightshift-memory-usage]

When an [investigation](nightshift-investigations.md) starts, the investigation agent reads memory pages relevant to the affected service and event type before issuing any telemetry queries. This means:

- The agent starts with knowledge about the service's architecture, known failure modes, and past incident patterns.
- Targeted ES|QL queries replace broad exploratory queries.
- Investigations for familiar issue types complete faster and produce higher-confidence hypotheses.

Memory is also available to you directly during chat — you can ask the AI Assistant what it knows about a service, request the current gap overview, or ask it to update a memory page based on something you've just learned.

## In this section [nightshift-memory-nav]

- [Nightshift overview](index.md)
- [Landing page](nightshift-landing-page.md)
- [Investigations](nightshift-investigations.md)
- [Code Intelligence](nightshift-code-intelligence.md)
