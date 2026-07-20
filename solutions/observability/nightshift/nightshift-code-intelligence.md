---
navigation_title: Code Intelligence
applies_to:
  serverless: preview
description: Code Intelligence connects your source code repositories to Nightshift so agents can reason about code as a first-class signal alongside telemetry.
products:
  - id: observability
---

# Code Intelligence [nightshift-code-intelligence]

:::{note}
Nightshift is currently in private beta. Access is limited and by invitation only.
:::

Code Intelligence connects your source code repositories to Nightshift, making source code, commit history, and pull request activity available as signals during [investigations](nightshift-investigations.md) and KI extraction.

When Nightshift investigates an incident, code context answers questions that telemetry alone cannot: What changed recently? Which team owns the affected service? Was there a deploy in the last hour? What does this error handler actually do?

## What Code Intelligence provides [nightshift-code-intelligence-what]

**Semantic code search** — Nightshift can search your codebase by meaning, not just by keyword. When investigating an error pattern, for example, it can find the relevant handler, understand what it does, and identify related components — even if the error message doesn't literally match any function name.

**Code history** — Recent commits and pull requests are indexed alongside your source code. Investigations can identify recent deployments, correlate code changes with the timing of a Significant Event, and surface the team or individual responsible for a relevant change.

% TODO: Link "Knowledge Indicators" to the sig events doc once path is confirmed.
**Code as a KI source** — The KI extraction pipeline can use code context to generate more accurate Knowledge Indicators about your services — including technology stacks, dependencies, and component ownership that aren't always explicit in log data.

## Connect a repository [nightshift-code-intelligence-connect]

:::{note}
Code Intelligence requires {{kib}} as a dependency — the semantic search index and tooling are managed as part of {{kib}}.
:::

To connect a code repository:

1. Navigate to **Nightshift settings > Code Intelligence**.
2. Select **Connect repository**.
3. Enter your repository URL and provide the required access credentials (for private repositories).
4. Nightshift begins indexing the repository. On first index, a complete working system with code search tools is installed into {{kib}} automatically.

After indexing completes, code context is available to [investigations](nightshift-investigations.md) immediately — no additional configuration is needed.

### Supported repository types [nightshift-code-intelligence-repos]

Code Intelligence supports Git repositories, including repositories hosted on GitHub. Support for additional hosting providers is in progress.

## How Code Intelligence is used in investigations [nightshift-code-intelligence-investigations]

When an [investigation](nightshift-investigations.md) starts, the investigation agent has access to code search tools that let it:

- Search the codebase for code related to the affected service or error pattern.
- Retrieve recent commit and pull request history for relevant files or components.
- Identify which team owns the affected code.
- Correlate deployment timing (from commit or PR activity) with the timing of the Significant Event.

Code context is used alongside telemetry, [memory](nightshift-memory.md), and external connectors — not instead of them. The agent decides when to use code search based on what the investigation needs.

## Current limitations [nightshift-code-intelligence-limitations]

Code Intelligence in the private beta includes:

- **Semantic Code Search (SCS)** — Full semantic search over source code. Available for connected repositories.
- **Code History (preview)** — Commit and PR history indexing. This feature is in proof-of-concept stage; index quality and coverage may vary.

The following capabilities are planned for future milestones:

- Hosted SCS (no self-hosting required)
- Additional distribution options (Docker, npm, Homebrew)
- Automated code-level remediation (open fix PRs based on incident findings)

## In this section [nightshift-code-intelligence-nav]

- [Nightshift overview](index.md)
- [Landing page](nightshift-landing-page.md)
- [Investigations](nightshift-investigations.md)
- [Memory](nightshift-memory.md)
