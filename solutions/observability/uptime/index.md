---
mapped_pages:
  - https://www.elastic.co/guide/en/observability/current/uptime-intro.html
applies_to:
  stack: deprecated 8.15.0
  serverless: unavailable
products:
  - id: observability
---

# Uptime monitoring (deprecated) [uptime-intro]

::::{admonition} Deprecated in 8.15.0.
:class: warning

Use [Synthetic monitoring](/solutions/observability/synthetics/index.md) instead of the {{uptime-app}}.
::::

::::{important}
The {{uptime-app}} is for viewing result data from lightweight monitors running through {{heartbeat}} and [configured with a traditional `heartbeat.yml` file](/solutions/observability/uptime/get-started.md). This is for TCP, HTTP or ICMP monitors that you have configured and run from your own infrastructure with {{heartbeat}} natively.

For browser-based monitors, a richer management and reporting experience, and more capabilities such as triaging and responding to alerts, use the [{{synthetics-app}}](/solutions/observability/synthetics/index.md) instead of the {{uptime-app}}.

Note that the {{uptime-app}} is hidden from the interface when there is no recent {{heartbeat}} data. To see the app, you may need to turn on the **Always show legacy Uptime app** setting (`observability:enableLegacyUptimeApp`) under {{kib}} Advanced Settings. To learn how, refer to [Advanced Settings](kibana://reference/advanced-settings.md).

::::

The {{uptime-app}} uses {{agent}} to periodically check the status of your services and applications. Monitor the availability of network endpoints and services using [Lightweight HTTP/S, TCP, and ICMP monitors](/solutions/observability/synthetics/index.md#monitoring-uptime).

## Lightweight HTTP/S, TCP, and ICMP monitors [uptime-lightweight]

In the {{uptime-app}}, you can monitor the status of network endpoints using the following lightweight checks:

|     |     |
| --- | --- |
| **HTTP monitor** | Monitor your website. The HTTP monitor checks to make sure specific endpoints return the correctstatus code and display the correct text. |
| **ICMP monitor** | Check the availability of your hosts. The ICMP monitor uses ICMP (v4 and v6) EchoRequests to check the network reachability of the hosts you are pinging. This will tell you whether thehost is available and connected to the network, but doesn’t tell you if a service on the host is running ornot. |
| **TCP monitor** | Monitor the services running on your hosts. The TCP monitor checks individual portsto make sure the service is accessible and running. |

:::{image} /solutions/images/observability-uptime-app.png
:alt: {{uptime-app}} in {{kib}}
:screenshot:
:::

To set up your first monitor, refer to [Get started with Uptime](/solutions/observability/uptime/get-started.md).

## TLS Certificates [view-certificate-status]

The TLS Certificates page in the {{uptime-app}} lists the TLS certificates that are being monitored and shows the TLS certificate data in your indices.

In addition to the common name, associated monitors, issuer information, and SHA fingerprints, an assigned status is derived from the threshold values in the [Settings](/solutions/observability/uptime/configure-settings.md) page.

:::{image} /solutions/images/observability-tls-certificates.png
:alt: TLS certificates
:screenshot:
:::

The table entries can be sorted by *status* and *valid until*. You can use the search bar at the top of the view to find values in most of the TLS-related fields in your Uptime indices.

Additionally, you can select the **Alerts and rules** dropdown at the top of the page, and create a [TLS rule](../incident-management/create-tls-certificate-rule.md) to receive an alert when your certificate is about to expire.
