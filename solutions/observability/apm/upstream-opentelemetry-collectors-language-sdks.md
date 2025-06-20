---
mapped_pages:
  - https://www.elastic.co/guide/en/observability/current/apm-open-telemetry-direct.html
  - https://www.elastic.co/guide/en/serverless/current/observability-apm-agents-opentelemetry-opentelemetry-native-support.html
applies_to:
  stack:
  serverless:
products:
  - id: observability
  - id: apm
  - id: cloud-serverless
---

# Upstream OpenTelemetry Collectors and language SDKs [apm-open-telemetry-direct]

The {{stack}} natively supports the OpenTelemetry protocol (OTLP). This means logs, metrics, and trace data collected from your applications and infrastructure can be sent directly to the {{stack}}.

* Send data to Elastic from an upstream [OpenTelemetry Collector](/solutions/observability/apm/upstream-opentelemetry-collectors-language-sdks.md#apm-connect-open-telemetry-collector)
* Send data to Elastic from an upstream [OpenTelemetry language SDK](/solutions/observability/apm/upstream-opentelemetry-collectors-language-sdks.md#apm-instrument-apps-otel)

To compare approaches and choose the best approach for your use case, refer to [OpenTelemetry](/solutions/observability/apm/use-opentelemetry-with-apm.md).

::::{important}
The Elastic Distribution of OpenTelemetry Collector (EDOT Collector) include additional features and configurations to seamlessly integrate with Elastic. Refer to [EDOT compared to upstream OpenTelemetry](opentelemetry:///reference/compatibility/edot-vs-upstream.md) for a comparison.
::::

## Send data from an upstream OpenTelemetry Collector [apm-connect-open-telemetry-collector]

Connect your OpenTelemetry Collector instances to Elastic {{observability}} or {{obs-serverless}} using the OTLP exporter:

::::{tab-set}
:group: stack-serverless

:::{tab-item} Elastic Stack
:sync: stack

```yaml
receivers: <1>
  # ...
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
processors: <2>
  # ...
  memory_limiter:
    check_interval: 1s
    limit_mib: 2000
  batch:

exporters:
  debug:
    verbosity: detailed <3>
  otlp: <4>
    # Elastic APM server https endpoint without the "https://" prefix
    endpoint: "${env:ELASTIC_APM_SERVER_ENDPOINT}" <5> <7>
    headers:
      # Elastic APM Server secret token
      Authorization: "Bearer ${env:ELASTIC_APM_SECRET_TOKEN}" <6> <7>

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [..., memory_limiter, batch]
      exporters: [debug, otlp]
    metrics:
      receivers: [otlp]
      processors: [..., memory_limiter, batch]
      exporters: [debug, otlp]
    logs: <8>
      receivers: [otlp]
      processors: [..., memory_limiter, batch]
      exporters: [debug, otlp]
```

1. The receivers, like the [OTLP receiver](https://github.com/open-telemetry/opentelemetry-collector/tree/main/receiver/otlpreceiver), that forward data emitted by APM agents, or the [host metrics receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/hostmetricsreceiver).
2. Use the [Batch processor](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/batchprocessor/README.md) and the [memory limiter processor](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/memorylimiterprocessor/README.md). For more information, see [recommended processors](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/README.md#recommended-processors).
3. The [debug exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/debugexporter) is helpful for troubleshooting, and supports configurable verbosity levels: `basic` (default), `normal`, and `detailed`.
4. Elastic {{observability}} endpoint configuration. APM Server supports a ProtoBuf payload via both the OTLP protocol over gRPC transport [(OTLP/gRPC)](https://opentelemetry.io/docs/specs/otlp/#otlpgrpc) and the OTLP protocol over HTTP transport [(OTLP/HTTP)](https://opentelemetry.io/docs/specs/otlp/#otlphttp). To learn more about these exporters, see the OpenTelemetry Collector documentation: [OTLP/HTTP Exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/otlphttpexporter) or [OTLP/gRPC exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/otlpexporter). When adding an endpoint to an existing configuration an optional name component can be added, like `otlp/elastic`, to distinguish endpoints as described in the [OpenTelemetry Collector Configuration Basics](https://opentelemetry.io/docs/collector/configuration/#basics).
5. Hostname and port of the APM Server endpoint. For example, `elastic-apm-server:8200`.
6. Credential for Elastic APM [secret token authorization](/solutions/observability/apm/secret-token.md) (`Authorization: "Bearer a_secret_token"`) or [API key authorization](/solutions/observability/apm/api-keys.md) (`Authorization: "ApiKey an_api_key"`).
7. Environment-specific configuration parameters can be conveniently passed in as environment variables documented [here](https://opentelemetry.io/docs/collector/configuration/#environment-variables) (e.g. `ELASTIC_APM_SERVER_ENDPOINT` and `ELASTIC_APM_SECRET_TOKEN`).
8. To send OpenTelemetry logs to {{stack}} version 8.0+, declare a `logs` pipeline. {applies_to}`product: preview`

:::

:::{tab-item} Serverless
:sync: serverless

```yaml
receivers:   <1>
  # ...
  otlp:

processors:   <2>
  # ...
  memory_limiter:
    check_interval: 1s
    limit_mib: 2000
  batch:

exporters:
  logging:
    loglevel: warn   <3>
  otlp/elastic:   <4>
    # Elastic https endpoint without the "https://" prefix
    endpoint: "${ELASTIC_APM_SERVER_ENDPOINT}" <5> <7>
    headers:
      # Elastic API key
      Authorization: "ApiKey ${ELASTIC_APM_API_KEY}" <6> <7>

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [..., memory_limiter, batch]
      exporters: [logging, otlp/elastic]
    metrics:
      receivers: [otlp]
      processors: [..., memory_limiter, batch]
      exporters: [logging, otlp/elastic]
    logs:   <8>
      receivers: [otlp]
      processors: [..., memory_limiter, batch]
      exporters: [logging, otlp/elastic]
```

1. The receivers, like the [OTLP receiver](https://github.com/open-telemetry/opentelemetry-collector/tree/main/receiver/otlpreceiver), that forward data emitted by APM agents, or the [host metrics receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/hostmetricsreceiver).
2. We recommend using the [Batch processor](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/batchprocessor/README.md) and the [memory limiter processor](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/memorylimiterprocessor/README.md). For more information, see [recommended processors](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/README.md#recommended-processors).
3. The [logging exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/loggingexporter) is helpful for troubleshooting and supports various logging levels, like `debug`, `info`, `warn`, and `error`.
4. {{obs-serverless}} endpoint configuration. Elastic supports a ProtoBuf payload via both the OTLP protocol over gRPC transport [(OTLP/gRPC)](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/protocol/otlp.md#otlpgrpc) and the OTLP protocol over HTTP transport [(OTLP/HTTP)](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/protocol/otlp.md#otlphttp). To learn more about these exporters, see the OpenTelemetry Collector documentation: [OTLP/HTTP Exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/otlphttpexporter) or [OTLP/gRPC exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/otlpexporter).
5. Hostname and port of the Elastic endpoint. For example, `elastic-apm-server:8200`.
6. Credential for Elastic APM API key authorization (`Authorization: "ApiKey an_api_key"`).
7. Environment-specific configuration parameters can be conveniently passed in as environment variables documented [here](https://opentelemetry.io/docs/collector/configuration/#configuration-environment-variables) (e.g. `ELASTIC_APM_SERVER_ENDPOINT` and `ELASTIC_APM_API_KEY`).
8. To send OpenTelemetry logs to your project, declare a `logs` pipeline. {applies_to}`product: preview`

:::

::::

You’re now ready to export traces and metrics from your services and applications.

::::{important}
When using the OpenTelemetry Collector, send data through the [`OTLP` exporter](https://github.com/open-telemetry/opentelemetry-collector/tree/main/exporter/otlphttpexporter). Using other methods, like the [`elasticsearch` exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/elasticsearchexporter), bypasses all of the validation and data processing that Elastic performs. In addition, your data will not be viewable in your Observability project if you use the `elasticsearch` exporter.
::::

## Send data from an upstream OpenTelemetry SDK [apm-instrument-apps-otel]

::::{note}
The following instructions show how to send data directly from an upstream OpenTelemetry SDK to Elastic, which is appropriate when getting started. However, sending data from an OpenTelemetry SDK to the OpenTelemetry Collector is preferred, as the Collector processes and exports data to Elastic. Read more about when and how to use a collector in the [OpenTelemetry documentation](https://opentelemetry.io/docs/collector/#when-to-use-a-collector).
::::

To export traces and metrics to Elastic, instrument your services and applications with the OpenTelemetry API, SDK, or both. For example, if you are a Java developer, you need to instrument your Java app with the [OpenTelemetry agent for Java](https://github.com/open-telemetry/opentelemetry-java-instrumentation). See the [OpenTelemetry Instrumentation guides](https://opentelemetry.io/docs/instrumentation/) to download the OpenTelemetry agent or SDK for your language.

Define environment variables to configure the OpenTelemetry agent or SDK and enable communication with Elastic APM. For example, if you are instrumenting a Java app, define the following environment variables:

::::{tab-set}
:group: stack-serverless

:::{tab-item} Elastic Stack
:sync: stack

```bash
export OTEL_RESOURCE_ATTRIBUTES=service.name=checkoutService,service.version=1.1,deployment.environment=production
export OTEL_EXPORTER_OTLP_ENDPOINT=https://apm_server_url:8200
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer an_apm_secret_token"
export OTEL_METRICS_EXPORTER="otlp" \
export OTEL_LOGS_EXPORTER="otlp" \ <1>
java -javaagent:/path/to/opentelemetry-javaagent-all.jar \
     -classpath lib/*:classes/ \
     com.mycompany.checkout.CheckoutServiceServer
```

1. The OpenTelemetry logs intake via APM Server is currently in technical preview. {applies_to}`product: preview`

`OTEL_RESOURCE_ATTRIBUTES`
:   Fields that describe the service and the environment that the service runs in. See [attributes](/solutions/observability/apm/attributes.md) for more information.

`OTEL_EXPORTER_OTLP_ENDPOINT`
:   APM Server URL. The host and port that APM Server listens for events on.

`OTEL_EXPORTER_OTLP_HEADERS`
:   Authorization header that includes the Elastic APM Secret token or API key: `"Authorization=Bearer an_apm_secret_token"` or `"Authorization=ApiKey an_api_key"`.

    For information on how to format an API key, see [API keys](/solutions/observability/apm/api-keys.md).

    Note the required space between `Bearer` and `an_apm_secret_token`, and `ApiKey` and `an_api_key`.

    ::::{note}
    If you are using a version of the Python OpenTelemetry agent *before* 1.27.0, the content of the header *must* be URL-encoded. You can use the Python standard library’s `urllib.parse.quote` function to encode the content of the header.
    ::::

`OTEL_METRICS_EXPORTER`
:   Metrics exporter to use. See [exporter selection](https://opentelemetry.io/docs/specs/otel/configuration/sdk-environment-variables/#exporter-selection) for more information.

`OTEL_LOGS_EXPORTER`
:   Logs exporter to use. See [exporter selection](https://opentelemetry.io/docs/specs/otel/configuration/sdk-environment-variables/#exporter-selection) for more information.

:::

:::{tab-item} Serverless
:sync: serverless

```bash
export OTEL_RESOURCE_ATTRIBUTES=service.name=checkoutService,service.version=1.1,deployment.environment=production
export OTEL_EXPORTER_OTLP_ENDPOINT=https://apm_server_url:8200
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=ApiKey an_apm_api_key"
export OTEL_METRICS_EXPORTER="otlp" \
export OTEL_LOGS_EXPORTER="otlp" \   <1>
java -javaagent:/path/to/opentelemetry-javaagent-all.jar \
     -classpath lib/*:classes/ \
     com.mycompany.checkout.CheckoutServiceServer
```

1. The OpenTelemetry logs intake via Elastic is currently in technical preview. {applies_to}`product: preview`

`OTEL_RESOURCE_ATTRIBUTES`
:   Fields that describe the service and the environment that the service runs in. See [attributes](/solutions/observability/apm/attributes.md) for more information.

`OTEL_EXPORTER_OTLP_ENDPOINT`
:   Elastic URL. The host and port that Elastic listens for APM events on.

`OTEL_EXPORTER_OTLP_HEADERS`
:   Authorization header that includes the Elastic APM API key: `"Authorization=ApiKey an_api_key"`. Note the required space between `ApiKey` and `an_api_key`.

    For information on how to format an API key, refer to [Secure communication with APM agents](/solutions/observability/apm/use-apm-securely.md).

    ::::{note}
    If you are using a version of the Python OpenTelemetry agent *before* 1.27.0, the content of the header *must* be URL-encoded. You can use the Python standard library’s `urllib.parse.quote` function to encode the content of the header.

    ::::

`OTEL_METRICS_EXPORTER`
:   Metrics exporter to use. See [exporter selection](https://opentelemetry.io/docs/specs/otel/configuration/sdk-environment-variables/#exporter-selection) for more information.

`OTEL_LOGS_EXPORTER`
:   Logs exporter to use. See [exporter selection](https://opentelemetry.io/docs/specs/otel/configuration/sdk-environment-variables/#exporter-selection) for more information.

:::

::::

You are now ready to collect traces and [metrics](/solutions/observability/apm/collect-metrics.md) before [verifying metrics](/solutions/observability/apm/collect-metrics.md#apm-open-telemetry-verify-metrics) and [visualizing metrics](/solutions/observability/apm/collect-metrics.md#apm-open-telemetry-visualize).

## Proxy requests to APM Server [apm-open-telemetry-proxy-apm]

```{applies_to}
product: preview
```

APM Server supports both the [OTLP/gRPC](https://opentelemetry.io/docs/specs/otlp/#otlpgrpc) and [OTLP/HTTP](https://opentelemetry.io/docs/specs/otlp/#otlphttp) protocol on the same port as Elastic APM agent requests. For ease of setup, use OTLP/HTTP when proxying or load balancing requests to Elastic.

If you use the OTLP/gRPC protocol, requests to Elastic must use either HTTP/2 over TLS or HTTP/2 Cleartext (H2C). No matter which protocol is used, OTLP/gRPC requests will have the header: `"Content-Type: application/grpc"`.

When using a layer 7 (L7) proxy like AWS ALB, proxy the requests in a way that ensures they follow the rules outlined previously. For example, with ALB you can create rules to select an alternative backend protocol based on the headers of requests coming into ALB. In this example, you’d select the gRPC protocol when the  `"Content-Type: application/grpc"` header exists on a request.

Many L7 load balancers handle HTTP and gRPC traffic separately and rely on explicitly defined routes and service configurations to correctly proxy requests. Since APM Server serves both protocols on the same port, it may not be compatible with some L7 load balancers. For example, to work around this issue in [Ingress NGINX Controller for Kubernetes](https://github.com/kubernetes/ingress-nginx), either:

* Use the `otlp` exporter in the EDOT collector. Set annotation `nginx.ingress.kubernetes.io/backend-protocol: "GRPC"` on the K8s Ingress object proxying to APM Server.
* Use the `otlphttp` exporter in the EDOT collector. Set annotation `nginx.ingress.kubernetes.io/backend-protocol: "HTTP"` (or `"HTTPS"` if APM Server expects TLS) on the K8s Ingress object proxying to APM Server.

The preferred approach is to deploy a L4 (TCP) load balancer (e.g. [NLB](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html) on AWS) in front of APM Server, which forwards raw TCP traffic transparently without protocol inspection.

For more information on how to configure an AWS ALB to support gRPC, see this AWS blog post: [Application Load Balancer Support for End-to-End HTTP/2 and gRPC](https://aws.amazon.com/blogs/aws/new-application-load-balancer-support-for-end-to-end-http-2-and-grpc/).

For more information on how APM Server services gRPC requests, see [Muxing gRPC and HTTP/1.1](https://github.com/elastic/apm-server/blob/main/dev_docs/otel.md#muxing-grpc-and-http11).

:::{include} _snippets/apm-server-vs-mis.md
:::

## Next steps [apm-open-telemetry-direct-next]

* [Collect metrics](/solutions/observability/apm/collect-metrics.md)
* Add [resource attributes](/solutions/observability/apm/attributes.md)
* Learn about the [limitations of this integration](/solutions/observability/apm/limitations.md)
