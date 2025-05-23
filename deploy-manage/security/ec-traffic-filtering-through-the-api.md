---
mapped_pages:
  - https://www.elastic.co/guide/en/cloud/current/ec-traffic-filtering-through-the-api.html
  - https://www.elastic.co/guide/en/cloud-enterprise/current/ece-traffic-filtering-through-the-api.html
applies_to:
  deployment:
    ess:
    ece:
products:
  - id: cloud-hosted
  - id: cloud-enterprise
---

# Manage traffic filters through the API [ec-traffic-filtering-through-the-api]

This example demonstrates how to use the {{ecloud}} RESTful API or {{ece}} RESTful API or to manage different types of traffic filters. We cover the following examples:

* [Create a traffic filter rule set](ec-traffic-filtering-through-the-api.md#ec-create-a-traffic-filter-rule-set)

    * [IP traffic filter ingress rule set](ec-traffic-filtering-through-the-api.md#ec-ip-traffic-filters-ingress-rule-set)
    * {{ech}} only:
      * [IP traffic filter egress rule set](ec-traffic-filtering-through-the-api.md#ec-ip-traffic-filters-egress-rule-set)
      * [AWS Privatelink traffic filters](ec-traffic-filtering-through-the-api.md#ec-aws-privatelink-traffic-filters-rule-set)
      * [Azure Private Link traffic filters](ec-traffic-filtering-through-the-api.md#ec-azure-privatelink-traffic-filters-rule-set)
      * [GCP Private Service Connect traffic filters](ec-traffic-filtering-through-the-api.md#ec-gcp-private-service-connect-traffic-filters-rule-set)

* [Update a traffic filter rule set](ec-traffic-filtering-through-the-api.md#ec-update-a-traffic-filter-rule-set)
* [Associate a rule set with a deployment](ec-traffic-filtering-through-the-api.md#ec-associate-rule-set-with-a-deployment)
* [Delete a rule set association with a deployment](ec-traffic-filtering-through-the-api.md#ec-delete-rule-set-association-with-a-deployment)
* [Delete a traffic filter rule set](ec-traffic-filtering-through-the-api.md#ec-delete-a-rule-set)

Refer to [](traffic-filtering.md) to learn about the general concepts behind filtering access to your {{ech}} and {{ece}} deployments.

To learn more about these endpoints, refer to the reference for your deployment type:

* [{{ecloud}} API](https://www.elastic.co/docs/api/doc/cloud/group/endpoint-deploymentstrafficfilter)
* [{{ece}} API](https://www.elastic.co/docs/api/doc/cloud-enterprise/group/endpoint-deploymentstrafficfilter)


## Create a traffic filter rule set [ec-create-a-traffic-filter-rule-set]


### IP traffic filter ingress rule set [ec-ip-traffic-filters-ingress-rule-set]
```{applies_to}
deployment:
  ess:
  ece:
```

Send a request like the following to create an IP traffic filter ingress rule set:

::::{tab-set}
:group: ech-ece

:::{tab-item} {{ech}}
:sync: ech

```sh
curl \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets \
-d '
{
  "name": "My IP filtering Ingress Rule Set",
  "region": "azure-japaneast",
  "description": "",
  "type": "ip",
  "rules": [
    {
      "description": "Allow inbound traffic from IP address 192.168.131.0",
      "source": "192.168.131.0"
    },
    {
      "description": "Allow inbound traffic within CIDR block 192.168.132.6/22",
      "source": "192.168.132.6/22"
    }
  ],
  "include_by_default": false
}
'
```

`region`
:   The region is always the same region as the deployment you want to associate with a traffic filter rule set. For details, check the [list of available regions](cloud://reference/cloud-hosted/ec-regions-templates-instances.md).

`type`
:   The type of the rule set. In the JSON object, we use `ip` for the ingress IP traffic filter. Currently, we support `ip`, `egress_firewall`, `vpce` (AWS Private Link), `azure_private_endpoint` and `gcp_private_service_connect_endpoint`. These are described in further detail below.
:::
:::{tab-item} {{ece}}
:sync: ece

```sh
curl \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://$COORDINATOR_HOST:12443/api/v1/deployments/traffic-filter/rulesets \
-d '
{
  "name": "My IP filtering Ingress Rule Set",
  "region": "ece-region",
  "description": "",
  "type": "ip",
  "rules": [
    {
      "description": "Allow inbound traffic from IP address 192.168.131.0",
      "source": "192.168.131.0"
    },
    {
      "description": "Allow inbound traffic within CIDR block 192.168.132.6/22",
      "source": "192.168.132.6/22"
    }
  ],
  "include_by_default": false
}
'
```
:::
::::

If the request is successful, a response containing a $RULESET_ID is returned. $RULESET_ID is required to update or delete the rule set itself, or it can be used to associate the rule set to a deployment.

```sh
{
  "id" : "5470a0010ebf437bb9294ea9fcba0ba0"
}
```


### IP traffic filter egress rule set [ec-ip-traffic-filters-egress-rule-set]
```{applies_to}
deployment:
  ess: beta
```

Send a request like the following to create an IP traffic filter egress rule set:

```sh
curl \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets \
-d '
{
  "name": "My IP filtering Egress Rule Set",
  "region": "azure-japaneast",
  "description": "",
  "type": "egress_firewall",
  "rules": [
    {
       "description": "Allow outbound traffic to IP address 192.168.131.0",
       "egress_rule":
       {
           "target": "192.168.131.0",
           "protocol": "all"
       }
    },
    {
       "description": "Allow outbound traffic to CIDR block 192.168.132.6/22",
       "egress_rule":
       {
           "target": "192.168.132.6/22",
           "protocol": "all"
       }
    },
  ],
  "include_by_default": false
}
'
```

`protocol`
:   This can be `udp`, `tcp`, or `all`.


### AWS Privatelink traffic filters [ec-aws-privatelink-traffic-filters-rule-set]
```{applies_to}
deployment:
  ess:
```

Send a request like the following to create an AWS PrivateLink traffic filter rule set:

```sh
curl -XPOST \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets \
-d '
{
  "name": "AWS Private Link Traffic Filter",
  "region": "ap-northeast-1",
  "description": "",
  "type": "vpce",
  "rules": [
    {
      "source": "vpce-00000000000"
    }
  ],
  "include_by_default": false
}
'
```

To find the value for `source` for type `vpce`, check [Find your VPC endpoint ID](aws-privatelink-traffic-filters.md#ec-find-your-endpoint). This setting is supported only in AWS regions.


### Azure Private Link traffic filters [ec-azure-privatelink-traffic-filters-rule-set]
```{applies_to}
deployment:
  ess:
```

Send a request like the following to create an Azure Private Link traffic filter rule set:

```sh
curl -XPOST \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets \
-d '
{
  "name": "Azure Private Link Traffic Filter",
  "region": "azure-japaneast",
  "description": "",
  "type": "azure_private_endpoint",
  "rules": [
    {
      "azure_endpoint_name": "azure-demo",
      "azure_endpoint_guid": "7c0f05e4-e32b-4b10-a246-7b77f7dcc63c"
    }
  ],
  "include_by_default": false
}
'
```

To find the value for `azure_endpoint_name` and `azure_endpoint_guid` for type `azure_private_endpoint`, check [Find your private endpoint resource name](azure-private-link-traffic-filters.md#ec-find-your-resource-name) and [Find your private endpoint resource ID](azure-private-link-traffic-filters.md#ec-find-your-resource-id). This setting is supported only in Azure regions.


### GCP Private Service Connect traffic filters [ec-gcp-private-service-connect-traffic-filters-rule-set]
```{applies_to}
deployment:
  ess:
```

Send a request like the following to create a GCP Private Service Connect traffic filter rule set:

```sh
curl -XPOST \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets \
-d '
{
  "name": "GCP Private Service Connect Traffic Filter",
  "region": "gcp-asia-northeast1",
  "description": "",
  "type": "gcp_private_service_connect_endpoint",
  "rules": [
    {
      "source": "18446744072646845332"
    }
  ],
  "include_by_default": false
}
'
```

To find the value for `source` for type `gcp_private_service_connect_endpoint`, check [Find your Private Service Connect connection ID](gcp-private-service-connect-traffic-filters.md#ec-find-your-psc-connection-id). This setting is supported only in GCP regions.


## Update a traffic filter rule set [ec-update-a-traffic-filter-rule-set]
```{applies_to}
deployment:
  ess:
  ece:
```

Send a request like the following to update an IP traffic filter ingress rule set:

::::{tab-set}
:group: ech-ece

:::{tab-item} {{ech}}
:sync: ech
```sh
curl -XPUT \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID \
-d '
{
  "name": "My IP filtering Ingress Rule Set",
  "region": "azure-japaneast",
  "description": "",
  "type": "ip",
  "rules": [
    {
      "description": "Allow inbound traffic from IP address 192.168.131.0",
      "source": "192.168.131.0"
    },
    {
      "description": "Allow inbound traffic within CIDR block 192.168.132.6/22",
      "source": "192.168.132.6/22"
    }
  ],
  "include_by_default": true
}
'
```
:::
:::{tab-item} {{ece}}
:sync: ece

```sh
curl -XPUT \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://$COORDINATOR_HOST:12443/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID \
-d '
{
  "name": "My IP filtering Ingress Rule Set",
  "region": "ece-region",
  "description": "",
  "type": "ip",
  "rules": [
    {
      "description": "Allow inbound traffic from IP address 192.168.131.0",
      "source": "192.168.131.0"
    },
    {
      "description": "Allow inbound traffic within CIDR block 192.168.132.6/22",
      "source": "192.168.132.6/22"
    }
  ],
  "include_by_default": true
}
'
```
:::
::::


## Associate a rule set with a deployment [ec-associate-rule-set-with-a-deployment]
```{applies_to}
deployment:
  ess:
  ece:
```

Send a request like the following to associate a rule set with a deployment:

::::{tab-set}
:group: ech-ece

:::{tab-item} {{ech}}
:sync: ech
```sh
curl -XPOST \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID/associations \
-d '
{
   "entity_type" : "deployment",
   "id" : "'"$DEPLOYMENT_ID"'"
}
'
```
:::
:::{tab-item} {{ece}}
:sync: ece

```sh
curl -XPOST \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://$COORDINATOR_HOST:12443/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID/associations \
-d '
{
   "entity_type" : "deployment",
   "id" : "'"$DEPLOYMENT_ID"'"
}
'
```
:::
::::


## Delete a rule set association with a deployment [ec-delete-rule-set-association-with-a-deployment]
```{applies_to}
deployment:
  ess:
  ece:
```

Send a request like the following to delete a rule set association with a deployment:

::::{tab-set}
:group: ech-ece

:::{tab-item} {{ech}}
:sync: ech

```sh
curl -XDELETE \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID/associations/deployment/$DEPLOYMENT_ID \
```
:::

:::{tab-item} {{ece}}
:sync: ece

```sh
curl -XDELETE \
-H "Authorization: ApiKey $API_KEY" \
-H 'content-type: application/json' \
https://$COORDINATOR_HOST:12443/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID/associations/deployment/$DEPLOYMENT_ID \
```
:::
::::


## Delete a traffic filter rule set [ec-delete-a-rule-set]
```{applies_to}
deployment:
  ess:
  ece:
```

Send a request like the following to delete a traffic filter rule set:

::::{tab-set}
:group: ech-ece

:::{tab-item} {{ech}}
:sync: ech

```sh
curl -XDELETE \
-H "Authorization: ApiKey $API_KEY" \
https://api.elastic-cloud.com/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID \
```
:::
:::{tab-item} {{ece}}
:sync: ece
```sh
curl -XDELETE \
-H "Authorization: ApiKey $API_KEY" \
https://$COORDINATOR_HOST:12443/api/v1/deployments/traffic-filter/rulesets/$RULESET_ID \
```
:::
::::