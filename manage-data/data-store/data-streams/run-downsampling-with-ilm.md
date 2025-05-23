---
navigation_title: Run downsampling with ILM
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/downsampling-ilm.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: elasticsearch
---



# Run downsampling with ILM [downsampling-ilm]


This is a simplified example that allows you to see quickly how [downsampling](./downsampling-time-series-data-stream.md) works as part of an ILM policy to reduce the storage size of a sampled set of metrics. The example uses typical Kubernetes cluster monitoring data. To test out downsampling with ILM, follow these steps:

1. Check the [prerequisites](#downsampling-ilm-prereqs).
2. [Create an index lifecycle policy](#downsampling-ilm-policy).
3. [Create an index template](#downsampling-ilm-create-index-template).
4. [Ingest time series data](#downsampling-ilm-ingest-data).
5. [View the results](#downsampling-ilm-view-results).


## Prerequisites [downsampling-ilm-prereqs]

Refer to [time series data stream prerequisites](./set-up-tsds.md#tsds-prereqs).

Before running this example you may want to try the [Run downsampling manually](./run-downsampling-manually.md) example.


## Create an index lifecycle policy [downsampling-ilm-policy]

Create an ILM policy for your time series data. While not required, an ILM policy is recommended to automate the management of your time series data stream indices.

To enable downsampling, add a [Downsample action](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-downsample.md) and set [`fixed_interval`](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-downsample.md#ilm-downsample-options) to the downsampling interval at which you want to aggregate the original time series data.

In this example, an ILM policy is configured for the `hot` phase. The downsample takes place after the index is rolled over and the [index time series end time](elasticsearch://reference/elasticsearch/index-settings/time-series.md#index-time-series-end-time) has lapsed as the source index is still expected to receive major writes until then. {{ilm-cap}} will not proceed with any action that expects the index to not receive writes anymore until the [index’s end time](elasticsearch://reference/elasticsearch/index-settings/time-series.md#index-time-series-end-time) has passed. The {{ilm-cap}} actions that wait on the end time before proceeding are: - [Delete](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-delete.md) - [Downsample](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-downsample.md) - [Force merge](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-forcemerge.md) - [Read only](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-readonly.md) - [Searchable snapshot](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-searchable-snapshot.md) - [Shrink](elasticsearch://reference/elasticsearch/index-lifecycle-actions/ilm-shrink.md)

```console
PUT _ilm/policy/datastream_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover" : {
            "max_age": "5m"
          },
          "downsample": {
  	        "fixed_interval": "1h"
  	      }
        }
      }
    }
  }
}
```


## Create an index template [downsampling-ilm-create-index-template]

This creates an index template for a basic data stream. The available parameters for an index template are described in detail in [Set up a time series data stream](set-up-data-stream.md).

For simplicity, in the time series mapping all `time_series_metric` parameters are set to type `gauge`, but the `counter` metric type may also be used. The `time_series_metric` values determine the kind of statistical representations that are used during downsampling.

The index template includes a set of static [time series dimensions](time-series-data-stream-tsds.md#time-series-dimension): `host`, `namespace`, `node`, and `pod`. The time series dimensions are not changed by the downsampling process.

```console
PUT _index_template/datastream_template
{
    "index_patterns": [
        "datastream*"
    ],
    "data_stream": {},
    "template": {
        "settings": {
            "index": {
                "mode": "time_series",
                "number_of_replicas": 0,
                "number_of_shards": 2
            },
            "index.lifecycle.name": "datastream_policy"
        },
        "mappings": {
            "properties": {
                "@timestamp": {
                    "type": "date"
                },
                "kubernetes": {
                    "properties": {
                        "container": {
                            "properties": {
                                "cpu": {
                                    "properties": {
                                        "usage": {
                                            "properties": {
                                                "core": {
                                                    "properties": {
                                                        "ns": {
                                                            "type": "long"
                                                        }
                                                    }
                                                },
                                                "limit": {
                                                    "properties": {
                                                        "pct": {
                                                            "type": "float"
                                                        }
                                                    }
                                                },
                                                "nanocores": {
                                                    "type": "long",
                                                    "time_series_metric": "gauge"
                                                },
                                                "node": {
                                                    "properties": {
                                                        "pct": {
                                                            "type": "float"
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    }
                                },
                                "memory": {
                                    "properties": {
                                        "available": {
                                            "properties": {
                                                "bytes": {
                                                    "type": "long",
                                                    "time_series_metric": "gauge"
                                                }
                                            }
                                        },
                                        "majorpagefaults": {
                                            "type": "long"
                                        },
                                        "pagefaults": {
                                            "type": "long",
                                            "time_series_metric": "gauge"
                                        },
                                        "rss": {
                                            "properties": {
                                                "bytes": {
                                                    "type": "long",
                                                    "time_series_metric": "gauge"
                                                }
                                            }
                                        },
                                        "usage": {
                                            "properties": {
                                                "bytes": {
                                                    "type": "long",
                                                    "time_series_metric": "gauge"
                                                },
                                                "limit": {
                                                    "properties": {
                                                        "pct": {
                                                            "type": "float"
                                                        }
                                                    }
                                                },
                                                "node": {
                                                    "properties": {
                                                        "pct": {
                                                            "type": "float"
                                                        }
                                                    }
                                                }
                                            }
                                        },
                                        "workingset": {
                                            "properties": {
                                                "bytes": {
                                                    "type": "long",
                                                    "time_series_metric": "gauge"
                                                }
                                            }
                                        }
                                    }
                                },
                                "name": {
                                    "type": "keyword"
                                },
                                "start_time": {
                                    "type": "date"
                                }
                            }
                        },
                        "host": {
                            "type": "keyword",
                            "time_series_dimension": true
                        },
                        "namespace": {
                            "type": "keyword",
                            "time_series_dimension": true
                        },
                        "node": {
                            "type": "keyword",
                            "time_series_dimension": true
                        },
                        "pod": {
                            "type": "keyword",
                            "time_series_dimension": true
                        }
                    }
                }
            }
        }
    }
}
```


## Ingest time series data [downsampling-ilm-ingest-data]

Use a bulk API request to automatically create your TSDS and index a set of ten documents.

**Important:** Before running this bulk request you need to update the timestamps to within three to five hours after your current time. That is, search `2022-06-21T15` and replace with your present date, and adjust the hour to your current time plus three hours.

```console
PUT /datastream/_bulk?refresh
{"create": {}}
{"@timestamp":"2022-06-21T15:49:00Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":91153,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":463314616},"usage":{"bytes":307007078,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":585236},"rss":{"bytes":102728},"pagefaults":120901,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:45:50Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":124501,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":982546514},"usage":{"bytes":360035574,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":1339884},"rss":{"bytes":381174},"pagefaults":178473,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:44:50Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":38907,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":862723768},"usage":{"bytes":379572388,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":431227},"rss":{"bytes":386580},"pagefaults":233166,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:44:40Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":86706,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":567160996},"usage":{"bytes":103266017,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":1724908},"rss":{"bytes":105431},"pagefaults":233166,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:44:00Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":150069,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":639054643},"usage":{"bytes":265142477,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":1786511},"rss":{"bytes":189235},"pagefaults":138172,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:42:40Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":82260,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":854735585},"usage":{"bytes":309798052,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":924058},"rss":{"bytes":110838},"pagefaults":259073,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:42:10Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":153404,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":279586406},"usage":{"bytes":214904955,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":1047265},"rss":{"bytes":91914},"pagefaults":302252,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:40:20Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":125613,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":822782853},"usage":{"bytes":100475044,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":2109932},"rss":{"bytes":278446},"pagefaults":74843,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:40:10Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":100046,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":567160996},"usage":{"bytes":362826547,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":1986724},"rss":{"bytes":402801},"pagefaults":296495,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
{"create": {}}
{"@timestamp":"2022-06-21T15:38:30Z","kubernetes":{"host":"gke-apps-0","node":"gke-apps-0-0","pod":"gke-apps-0-0-0","container":{"cpu":{"usage":{"nanocores":40018,"core":{"ns":12828317850},"node":{"pct":2.77905e-05},"limit":{"pct":2.77905e-05}}},"memory":{"available":{"bytes":1062428344},"usage":{"bytes":265142477,"node":{"pct":0.01770037710617187},"limit":{"pct":9.923134671484496e-05}},"workingset":{"bytes":2294743},"rss":{"bytes":340623},"pagefaults":224530,"majorpagefaults":0},"start_time":"2021-03-30T07:59:06Z","name":"container-name-44"},"namespace":"namespace26"}}
```


## View the results [downsampling-ilm-view-results]

Now that you’ve created and added documents to the data stream, check to confirm the current state of the new index.

```console
GET _data_stream
```

If the ILM policy has not yet been applied, your results will be like the following. Note the original `index_name`: `.ds-datastream-<timestamp>-000001`.

```console-result
{
  "data_streams": [
    {
      "name": "datastream",
      "timestamp_field": {
        "name": "@timestamp"
      },
      "indices": [
        {
          "index_name": ".ds-datastream-2022.08.26-000001",
          "index_uuid": "5g-3HrfETga-5EFKBM6R-w"
        },
        {
          "index_name": ".ds-datastream-2022.08.26-000002",
          "index_uuid": "o0yRTdhWSo2pY8XMvfwy7Q"
        }
      ],
      "generation": 2,
      "status": "GREEN",
      "template": "datastream_template",
      "ilm_policy": "datastream_policy",
      "hidden": false,
      "system": false,
      "allow_custom_routing": false,
      "replicated": false,
      "rollover_on_write": false,
      "time_series": {
        "temporal_ranges": [
          {
            "start": "2022-08-26T13:29:07.000Z",
            "end": "2022-08-26T19:29:07.000Z"
          }
        ]
      }
    }
  ]
}
```

Next, run a search query:

```console
GET datastream/_search
```

The query returns your ten newly added documents.

```console-result
{
  "took": 17,
  "timed_out": false,
  "_shards": {
    "total": 4,
    "successful": 4,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10,
      "relation": "eq"
    },
...
```

By default, index lifecycle management checks every ten minutes for indices that meet policy criteria. Wait for about ten minutes (maybe brew up a quick coffee or tea ☕ ) and then re-run the `GET _data_stream` request.

```console
GET _data_stream
```

After the ILM policy has taken effect, the original `.ds-datastream-2022.08.26-000001` index is replaced with a new, downsampled index, in this case `downsample-6tkn-.ds-datastream-2022.08.26-000001`.

```console-result
{
  "data_streams": [
    {
      "name": "datastream",
      "timestamp_field": {
        "name": "@timestamp"
      },
      "indices": [
        {
          "index_name": "downsample-6tkn-.ds-datastream-2022.08.26-000001",
          "index_uuid": "qRane1fQQDCNgKQhXmTIvg"
        },
        {
          "index_name": ".ds-datastream-2022.08.26-000002",
          "index_uuid": "o0yRTdhWSo2pY8XMvfwy7Q"
        }
      ],
...
```

Run a search query on the datastream (note that when querying downsampled indices there are [a few nuances to be aware of](./downsampling-time-series-data-stream.md#querying-downsampled-indices-notes)).

```console
GET datastream/_search
```

The new downsampled index contains just one document that includes the `min`, `max`, `sum`, and `value_count` statistics based off of the original sampled metrics.

```console-result
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 4,
    "successful": 4,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "downsample-6tkn-.ds-datastream-2022.08.26-000001",
        "_id": "0eL0wC_4-45SnTNFAAABgtpz0wA",
        "_score": 1,
        "_source": {
          "@timestamp": "2022-08-26T14:00:00.000Z",
          "_doc_count": 10,
          "kubernetes.host": "gke-apps-0",
          "kubernetes.namespace": "namespace26",
          "kubernetes.node": "gke-apps-0-0",
          "kubernetes.pod": "gke-apps-0-0-0",
          "kubernetes.container.cpu.usage.nanocores": {
            "min": 38907,
            "max": 153404,
            "sum": 992677,
            "value_count": 10
          },
          "kubernetes.container.memory.available.bytes": {
            "min": 279586406,
            "max": 1062428344,
            "sum": 7101494721,
            "value_count": 10
          },
          "kubernetes.container.memory.pagefaults": {
            "min": 74843,
            "max": 302252,
            "sum": 2061071,
            "value_count": 10
          },
          "kubernetes.container.memory.rss.bytes": {
            "min": 91914,
            "max": 402801,
            "sum": 2389770,
            "value_count": 10
          },
          "kubernetes.container.memory.usage.bytes": {
            "min": 100475044,
            "max": 379572388,
            "sum": 2668170609,
            "value_count": 10
          },
          "kubernetes.container.memory.workingset.bytes": {
            "min": 431227,
            "max": 2294743,
            "sum": 14230488,
            "value_count": 10
          },
          "kubernetes.container.cpu.usage.core.ns": 12828317850,
          "kubernetes.container.cpu.usage.limit.pct": 0.000027790500098490156,
          "kubernetes.container.cpu.usage.node.pct": 0.000027790500098490156,
          "kubernetes.container.memory.majorpagefaults": 0,
          "kubernetes.container.memory.usage.limit.pct": 0.00009923134348355234,
          "kubernetes.container.memory.usage.node.pct": 0.017700377851724625,
          "kubernetes.container.name": "container-name-44",
          "kubernetes.container.start_time": "2021-03-30T07:59:06.000Z"
        }
      }
    ]
  }
}
```

Use the [data stream stats API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-data-streams-stats-1) to get statistics for the data stream, including the storage size.

```console
GET /_data_stream/datastream/_stats?human=true
```

```console-result
{
  "_shards": {
    "total": 4,
    "successful": 4,
    "failed": 0
  },
  "data_stream_count": 1,
  "backing_indices": 2,
  "total_store_size": "16.6kb",
  "total_store_size_bytes": 17059,
  "data_streams": [
    {
      "data_stream": "datastream",
      "backing_indices": 2,
      "store_size": "16.6kb",
      "store_size_bytes": 17059,
      "maximum_timestamp": 1661522400000
    }
  ]
}
```

This example demonstrates how downsampling works as part of an ILM policy to reduce the storage size of metrics data as it becomes less current and less frequently queried.

You can also try our [Run downsampling manually](./run-downsampling-manually.md) example to learn how downsampling can work outside of an ILM policy.
