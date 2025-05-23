---
mapped_pages:
  - https://www.elastic.co/guide/en/machine-learning/current/ml-rare-functions.html
products:
  - id: machine-learning
---

# Rare functions [ml-rare-functions]

The rare functions detect values that occur rarely in time or rarely for a population.

The `rare` analysis detects anomalies according to the number of distinct rare values. This differs from `freq_rare`, which detects anomalies according to the number of times (frequency) rare values occur.

::::{note}
* The `rare` and `freq_rare` functions should not be used in conjunction with `exclude_frequent`.
* You cannot create forecasts for {{anomaly-jobs}} that contain `rare` or `freq_rare` functions.
* You cannot add rules with conditions to detectors that use `rare` or `freq_rare` functions.
* Shorter bucket spans (less than 1 hour, for example) are recommended when looking for rare events. The functions model whether something happens in a bucket at least once. With longer bucket spans, it is more likely that entities will be seen in a bucket and therefore they appear less rare. Picking the ideal bucket span depends on the characteristics of the data with shorter bucket spans typically being measured in minutes, not hours.
* To model rare data, a learning period of at least 20 buckets is required for typical data.

::::


The {{ml-features}} include the following rare functions:

* [`rare`](ml-rare-functions.md#ml-rare)
* [`freq_rare`](ml-rare-functions.md#ml-freq-rare)


## Rare [ml-rare]

The `rare` function detects values that occur rarely in time or rarely for a population. It detects anomalies according to the number of distinct rare values.

This function supports the following properties:

* `by_field_name` (required)
* `over_field_name` (optional)
* `partition_field_name` (optional)

For more information about those properties, see the [create {{anomaly-jobs}} API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-ml-put-job).

```js
{
  "function" : "rare",
  "by_field_name" : "status"
}
```

If you use this `rare` function in a detector in your {{anomaly-job}}, it detects values that are rare in time. It models status codes that occur over time and detects when rare status codes occur compared to the past. For example, you can detect status codes in a web access log that have never (or rarely) occurred before.

```js
{
  "function" : "rare",
  "by_field_name" : "status",
  "over_field_name" : "clientip"
}
```

If you use this `rare` function in a detector in your {{anomaly-job}}, it detects values that are rare in a population. It models status code and client IP interactions that occur. It defines a rare status code as one that occurs for few client IP values compared to the population. It detects client IP values that experience one or more distinct rare status codes compared to the population. For example in a web access log, a `clientip` that experiences the highest number of different rare status codes compared to the population is regarded as highly anomalous. This analysis is based on the number of different status code values, not the count of occurrences.

::::{note}
To define a status code as rare the {{ml-features}} look at the number of distinct status codes that occur, not the number of times the status code occurs. If a single client IP experiences a single unique status code, this is rare, even if it occurs for that client IP in every bucket.
::::



## Freq_rare [ml-freq-rare]

The `freq_rare` function detects values that occur rarely for a population. It detects anomalies according to the number of times (frequency) that rare values occur.

This function supports the following properties:

* `by_field_name` (required)
* `over_field_name` (required)
* `partition_field_name` (optional)

For more information about those properties, see the [create {{anomaly-jobs}} API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-ml-put-job).

```js
{
  "function" : "freq_rare",
  "by_field_name" : "uri",
  "over_field_name" : "clientip"
}
```

If you use this `freq_rare` function in a detector in your {{anomaly-job}}, it detects values that are frequently rare in a population. It models URI paths and client IP interactions that occur. It defines a rare URI path as one that is visited by few client IP values compared to the population. It detects the client IP values that experience many interactions with rare URI paths compared to the population. For example in a web access log, a client IP that visits one or more rare URI paths many times compared to the population is regarded as highly anomalous. This analysis is based on the count of interactions with rare URI paths, not the number of different URI path values.

::::{note}
Defining a URI path as rare happens the same way as you can see in the case of the status codes above: the analytics consider the number of distinct values that occur and not the number of times the URI path occurs. If a single client IP visits a single unique URI path, this is rare, even if it occurs for that client IP in every bucket.
::::


