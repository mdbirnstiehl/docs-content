---
mapped_pages:
  - https://www.elastic.co/guide/en/observability/current/apm-errors.html
  - https://www.elastic.co/guide/en/serverless/current/observability-apm-errors.html
applies_to:
  stack:
  serverless:
products:
  - id: observability
  - id: apm
  - id: cloud-serverless
---

# Errors [apm-errors]

*Errors* are groups of exceptions with a similar exception or log message. The **Errors** overview provides a high-level view of the exceptions that APM agents catch, or that users manually report with APM agent APIs. Like errors are grouped together to make it easy to quickly see which errors are affecting your services, and to take actions to rectify them.

A service returning a 5xx code from a request handler, controller, etc., will not create an exception that an APM agent can catch, and will therefore not show up in this view.

:::{image} /solutions/images/observability-apm-errors-overview.png
:alt: APM Errors overview
:screenshot:
:::

Selecting an error group ID or error message brings you to the **Error group**.

:::{image} /solutions/images/observability-apm-error-group.png
:alt: APM Error group
:screenshot:
:::

The error group details page visualizes the number of error occurrences over time and compared to a recent time range. This allows you to quickly determine if the error rate is changing or remaining constant. You’ll also see the top 5 affected transactions—enabling you to quickly narrow down which transactions are most impacted by the selected error.

Further down, you’ll see an Error sample. The error shown is always the most recent to occur. The sample includes the exception message, culprit, stack trace where the error occurred, and additional contextual information to help debug the issue—all of which can be copied with the click of a button.

In some cases, you might also see a Transaction sample ID. This feature allows you to make a connection between the errors and transactions, by linking you to the specific transaction where the error occurred. This allows you to see the whole trace, including which services the request went through.