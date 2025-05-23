---
mapped_pages:
  - https://www.elastic.co/guide/en/observability/current/apm-server-es-down.html
applies_to:
  stack: all
products:
  - id: observability
---

# APM Server or Elasticsearch is down [apm-server-es-down]

**If {{es}} is down**

APM Server does not have an internal queue to buffer requests, but instead leverages an HTTP request timeout to act as back-pressure. If {{es}} goes down, the APM Server will eventually deny incoming requests. Both the APM Server and {{apm-agent}}(s) will issue logs accordingly.

**If APM Server is down**

Some agents have internal queues or buffers that will temporarily store data if the APM Server goes down. As a general rule of thumb, queues fill up quickly. Assume data will be lost if APM Server goes down. Adjusting these queues/buffers can increase the agent’s overhead, so use caution when updating default values.

* **Go agent** - Circular buffer with configurable size: [`ELASTIC_APM_BUFFER_SIZE`](apm-agent-go://reference/configuration.md#config-api-buffer-size).
* **Java agent** - Internal buffer with configurable size: [`max_queue_size`](apm-agent-java://reference/config-reporter.md#config-max-queue-size).
* **Node.js agent** - No internal queue. Data is lost.
* **PHP agent** - No internal queue. Data is lost.
* **Python agent** - Internal [Transaction queue](apm-agent-python://reference/performance-tuning.md#tuning-queue) with configurable size and time between flushes.
* **Ruby agent** - Internal queue with configurable size: [`api_buffer_size`](apm-agent-ruby://reference/configuration.md#config-api-buffer-size).
* **RUM agent** - No internal queue. Data is lost.
* **.NET agent** - No internal queue. Data is lost.
