---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/mixing-exact-search-with-stemming.html
applies_to:
  stack:
  serverless:
products:
  - id: elasticsearch
---

# Mixing exact search with stemming [mixing-exact-search-with-stemming]

When building a search application, stemming is often a must as it is desirable for a query on `skiing` to match documents that contain `ski` or `skis`. But what if a user wants to search for `skiing` specifically? The typical way to do this would be to use a [multi-field](elasticsearch://reference/elasticsearch/mapping-reference/multi-fields.md) in order to have the same content indexed in two different ways:

```console
PUT index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_exact": {
          "tokenizer": "standard",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "body": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "exact": {
            "type": "text",
            "analyzer": "english_exact"
          }
        }
      }
    }
  }
}

PUT index/_doc/1
{
  "body": "Ski resort"
}

PUT index/_doc/2
{
  "body": "A pair of skis"
}

POST index/_refresh
```

With such a setup, searching for `ski` on `body` would return both documents:

```console
GET index/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "body" ],
      "query": "ski"
    }
  }
}
```

```console-result
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 2,
        "relation": "eq"
    },
    "max_score": 0.18232156,
    "hits": [
      {
        "_index": "index",
        "_id": "1",
        "_score": 0.18232156,
        "_source": {
          "body": "Ski resort"
        }
      },
      {
        "_index": "index",
        "_id": "2",
        "_score": 0.18232156,
        "_source": {
          "body": "A pair of skis"
        }
      }
    ]
  }
}
```

On the other hand, searching for `ski` on `body.exact` would only return document `1` since the analysis chain of `body.exact` does not perform stemming.

```console
GET index/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "body.exact" ],
      "query": "ski"
    }
  }
}
```

```console-result
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.8025915,
    "hits": [
      {
        "_index": "index",
        "_id": "1",
        "_score": 0.8025915,
        "_source": {
          "body": "Ski resort"
        }
      }
    ]
  }
}
```

This is not something that is easy to expose to end users, as we would need to have a way to figure out whether they are looking for an exact match or not and redirect to the appropriate field accordingly. Also what to do if only parts of the query need to be matched exactly while other parts should still take stemming into account?

Fortunately, the `query_string` and `simple_query_string` queries have a feature that solves this exact problem: `quote_field_suffix`. This tells Elasticsearch that the words that appear in between quotes are to be redirected to a different field, see below:

```console
GET index/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "body" ],
      "quote_field_suffix": ".exact",
      "query": "\"ski\""
    }
  }
}
```

```console-result
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.8025915,
    "hits": [
      {
        "_index": "index",
        "_id": "1",
        "_score": 0.8025915,
        "_source": {
          "body": "Ski resort"
        }
      }
    ]
  }
}
```

In the above case, since `ski` was in-between quotes, it was searched on the `body.exact` field due to the `quote_field_suffix` parameter, so only document `1` matched. This allows users to mix exact search with stemmed search as they like.

::::{note}
If the choice of field passed in `quote_field_suffix` does not exist the search will fall back to using the default field for the query string.
::::


