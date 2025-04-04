---
mapped_pages:
  - https://www.elastic.co/guide/en/security/current/agentless-integrations.html
  - https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html
applies_to:
  stack: all
  serverless:
    security: all
---

# Agentless integrations [agentless-integrations]


Agentless integrations provide a means to ingest data while avoiding the orchestration, management, and maintenance needs associated with standard ingest infrastructure. Using agentless integrations makes manual agent deployment unnecessary, allowing you to focus on your data instead of the agent that collects it.

We currently support one agentless integration: cloud security posture management (CSPM). Using this integration’s agentless deployment option, you can enable Elastic’s CSPM capabilities just by providing the necessary credentials. Agentless CSPM deployments support AWS, Azure, and GCP accounts.

To learn more about agentless CSPM deployments, refer to the getting started guides for CSPM on [AWS](../cloud/get-started-with-cspm-for-aws.md),  [Azure](../cloud/get-started-with-cspm-for-azure.md), or [GCP](../cloud/get-started-with-cspm-for-gcp.md)
