---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/simulate-multi-component-templates.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: elasticsearch
---

# Simulate multi-component templates [simulate-multi-component-templates]

Since templates can be composed not only of multiple component templates, but also the index template itself, there are two simulation APIs to determine what the resulting index settings will be.

To simulate the settings that would be applied to a particular index name:

```console
POST /_index_template/_simulate_index/my-index-000001
```

To simulate the settings that would be applied from an existing template:

```console
POST /_index_template/_simulate/template_1
```

You can also specify a template definition in the simulate request. This enables you to verify that settings will be applied as expected before you add a new template.

```console
PUT /_component_template/ct1
{
  "template": {
    "settings": {
      "index.number_of_shards": 2
    }
  }
}

PUT /_component_template/ct2
{
  "template": {
    "settings": {
      "index.number_of_replicas": 0
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        }
      }
    }
  }
}

POST /_index_template/_simulate
{
  "index_patterns": ["my*"],
  "template": {
    "settings" : {
        "index.number_of_shards" : 3
    }
  },
  "composed_of": ["ct1", "ct2"]
}
```

The response shows the settings, mappings, and aliases that would be applied to matching indices, and any overlapping templates whose configuration would be superseded by the simulated template body or higher-priority templates.

```console-result
{
  "template" : {
    "settings" : {
      "index" : {
        "number_of_shards" : "3",   <1>
        "number_of_replicas" : "0",
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        }
      }
    },
    "mappings" : {
      "properties" : {
        "@timestamp" : {
          "type" : "date"           <2>
        }
      }
    },
    "aliases" : { }
  },
  "overlapping" : [
    {
      "name" : "template_1",        <3>
      "index_patterns" : [
        "my*"
      ]
    }
  ]
}
```

1. The number of shards from the simulated template body
2. The `@timestamp` field inherited from the `ct2` component template
3. Any overlapping templates that would have matched, but have lower priority


