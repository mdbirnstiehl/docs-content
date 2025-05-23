---
navigation_title: Security
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/search-application-security.html
applies_to:
  stack: beta
  serverless: beta
products:
  - id: elasticsearch
---



# Security [search-application-security]


When building a frontend application for search use cases, there are two main approaches to returning search results:

1. The client (user’s browser) makes API requests to the application backend, which in turn makes a request to {{es}}. The {{es}} cluster is not exposed to the end user.
2. **The client (user’s browser) makes API requests directly to the search service - in this case the {{es}} cluster is reachable to the client.**

This guide describes best practices when taking the second approach. Specifically, we will explain how to use search applications with frontend apps that make direct requests to the [Search Application Search API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-search-application-search).

This approach has a few advantages:

* No need to maintain a passthrough query system between frontend applications and {{es}}
* Direct requests to {{es}} result in faster response times
* Query configuration is managed in one place: your search application configuration in {{es}}

We will cover:

* [Using {{es}} API keys with role restrictions](#search-application-security-key-restrictions)
* [Parameter validation in the Search Application Search API](#search-application-security-parameter-validation)
* [Working with CORS](#search-application-security-cors)


## Using {{es}} API keys with role restrictions [search-application-security-key-restrictions] 

When frontend applications can make direct API requests to {{es}}, it’s important to limit the operations they can perform. In our case, frontend applications should only be able to call the Search Application **Search API**. To ensure this, we will create {{es}} API keys with [role restrictions](../../../deploy-manage/users-roles/cluster-or-deployment-auth/role-restriction.md). A role restriction is used to specify under what conditions a role should be effective.

The following {{es}} API key has access to the `website-product-search` search application, only through the Search Application Search API:

```console
POST /_security/api_key
{
  "name": "my-restricted-api-key",
  "expiration": "7d",
  "role_descriptors": {
    "my-restricted-role-descriptor": {
      "indices": [
        {
          "names": ["website-product-search"], <1>
          "privileges": ["read"]
        }
      ],
      "restriction":  {
        "workflows": ["search_application_query"] <2>
      }
    }
  }
}
```

1. `indices.name` must be the name(s) of the Search Application(s), not the underlying {{es}} indices.
2. `restriction.workflows` must be set to the concrete value `search_application_query`.


::::{important} 
It is crucial to specify the workflow restriction. Without this the {{es}} API key can directly call `_search` and issue arbitrary {{es}} queries. This is insecure when dealing with untrusted clients.

::::


The response will look like this:

```console-result
{
  "id": "v1CCJYkBvb5Pg9T-_JgO",
  "name": "my-restricted-api-key",
  "expiration": 1689156288526,
  "api_key": "ztVI-1Q4RjS8qFDxAVet5w",
  "encoded": "djFDQ0pZa0J2YjVQZzlULV9KZ086enRWSS0xUTRSalM4cUZEeEFWZXQ1dw"
}
```

The encoded value can then be directly used in the Authorization header. Here’s an example using cURL:

```shell
curl -XPOST "http://localhost:9200/_application/search_application/website-product-search/_search" \
 -H "Content-Type: application/json" \
 -H "Authorization: ApiKey djFDQ0pZa0J2YjVQZzlULV9KZ086enRWSS0xUTRSalM4cUZEeEFWZXQ1dw" \
 -d '{
  "params": {
    "field_name": "color",
    "field_value": "red",
    "agg_size": 5
  }
}'
```

::::{tip} 
If `expiration` is not present, by default {{es}} API keys never expire. The API key can be invalidated using the [invalidate API key API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-invalidate-api-key).

::::


::::{tip} 
{{es}} API keys with role restrictions can also use field and document level security. This further limits how frontend applications query a search application.

::::



## Parameter validation with search applications [search-application-security-parameter-validation] 

Your search applications use [search templates](search-application-api.md) to render queries. The template parameters are passed to the Search Application Search API. In the case of APIs used by frontend applications or untrusted clients, we need to have strict parameter validation. Search applications define a JSON schema that describes which parameters the Search Application Search API allows.

The following example defines a search application with strict parameter validation:

```console
PUT _application/search_application/website-product-search
{
  "indices": [
    "website-products"
  ],
  "template": {
    "script": {
      "source": {
        "query": {
          "term": {
            "{{field_name}}": "{{field_value}}"
          }
        },
        "aggs": {
          "color_facet": {
            "terms": {
              "field": "color",
              "size": "{{agg_size}}"
            }
          }
        }
      },
      "params": {
        "field_name": "product_name",
        "field_value": "hello world",
        "agg_size": 5
      }
    },
    "dictionary": {
      "properties": {
        "field_name": {
          "type": "string",
          "enum": ["name", "color", "description"]
        },
        "field_value": {
          "type": "string"
        },
        "agg_size": {
          "type": "integer",
          "minimum": 1,
          "maximum": 10
        }
      },
      "required": [
        "field_name"
      ],
      "additionalProperties": false
    }
  }
}
```

Using that definition, the Search Application Search API performs the following parameter validation:

* It only accepts the `field_name`, `field_value` and `aggs_size` parameters
* `field_name` is restricted to only take the values "name", "color" and "description"
* `agg_size` defines the size of the term aggregation and it can only take values between `1` and `10`


## Working with CORS [search-application-security-cors] 

Using this approach means that your user’s browser will make requests to the {{es}} API directly. {{es}} supports [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), but this feature is disabled by default. Therefore the browser will block these requests.

There are two workarounds for this:

* [Enable CORS on {{es}}](#search-application-security-cors-elasticsearch)
* [Proxy the request through a server that supports CORS](#search-application-security-cors-proxy-request)


### Enable CORS on {{es}} [search-application-security-cors-elasticsearch] 

This is the simplest option. Enable CORS on {{es}} by adding the following to your [`elasticsearch.yml`](/deploy-manage/stack-settings.md) file:

```yaml
http.cors.allow-origin: "*" # Only use unrestricted value for local development
# Use a specific origin value in production, like `http.cors.allow-origin: "https://<my-website-domain.example>"`
http.cors.enabled: true
http.cors.allow-credentials: true
http.cors.allow-methods: OPTIONS, POST
http.cors.allow-headers: X-Requested-With, X-Auth-Token, Content-Type, Content-Length, Authorization, Access-Control-Allow-Headers, Accept
```

### Proxy the request through a server that supports CORS [search-application-security-cors-proxy-request] 

If you are unable to enable CORS on {{es}}, you can proxy the request through a server that supports CORS. This is more complicated, but is a viable option.


## Learn more [search-application-security-learn-more] 

* [Role restrictions](../../../deploy-manage/users-roles/cluster-or-deployment-auth/role-restriction.md)
* [Document level security](../../../deploy-manage/users-roles/cluster-or-deployment-auth/controlling-access-at-document-field-level.md)
* [Field level security](../../../deploy-manage/users-roles/cluster-or-deployment-auth/controlling-access-at-document-field-level.md)
* [APIs](search-application-api.md)

    * [PUT Search Application API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-search-application-put)
    * [Search Application Search API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-search-application-search)


