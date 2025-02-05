---
navigation_title: "Track deployments with annotations"
---

# Track deployments with annotations [apm-transactions-annotations]


:::{image} ../../../images/observability-apm-transaction-annotation.png
:alt: Example view of transactions annotation in the Applications UI in Kibana
:class: screenshot
:::

For enhanced visibility into your deployments, we offer deployment annotations on all transaction charts. This feature enables you to easily determine if your deployment has increased response times for an end-user, or if the memory/CPU footprint of your application has changed. Being able to quickly identify bad deployments enables you to rollback and fix issues without causing costly outages.

By default, automatic deployment annotations are enabled. This means the Applications UI will create an annotation on your data when the `service.version` of your application changes.

Alternatively, you can explicitly create deployment annotations with our annotation API. The API can integrate into your CI/CD pipeline, so that each time you deploy, a POST request is sent to the annotation API endpoint:

```curl
curl -X POST \
  http://localhost:5601/api/apm/services/${SERVICE_NAME}/annotation \ <1>
-H 'Content-Type: application/json' \
-H 'kbn-xsrf: true' \
-H 'Authorization: Basic ${API_KEY}' \ <2>
-d '{
      "@timestamp": "${DEPLOY_TIME}", <3>
      "service": {
        "version": "${SERVICE_VERSION}" <4>
      },
      "message": "${MESSAGE}" <5>
    }'
```

1. The `service.name` of your application
2. An APM API key with sufficient privileges
3. The time of the deployment
4. The `service.version` to be displayed in the annotation
5. A custom message to be displayed in the annotation


See the [annotation API](../../../solutions/observability/apps/annotation-api.md) reference for more information.

::::{note}
If custom annotations have been created for the selected time period, any derived annotations, i.e., those created automatically when `service.version` changes, will not be shown.
::::


