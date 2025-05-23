---
navigation_title: Job types
mapped_pages:
  - https://www.elastic.co/guide/en/machine-learning/current/ml-anomaly-detection-job-types.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: machine-learning
---



# Job types [ml-anomaly-detection-job-types]

{{anomaly-jobs-cap}} have many possible configuration options which enable you to fine-tune the jobs and cover your use case as much as possible. This page provides a quick overview of different types of {{anomaly-jobs}} and their capabilities. The job types available in {{kib}} are:

* single metric jobs,
* multi-metric jobs,
* population jobs,
* advanced jobs,
* categorization jobs,
* rare jobs,
* geo jobs.

## Single metric jobs [singe-metric-jobs]

Every {{anomaly-job}} has at least one detector. A detector defines the type of analysis that occurs (for example, using `max`, `average`, or `high` functions) and the field in your data that is analyzed. Single metric jobs have exactly one detector. These jobs are best for detecting anomalies in one aspect of your time series data. For example, you can monitor the request rate in your log data with the `low_count` function to find unusually low request rates that might be a sign of an error. Refer to the [*Function reference*](ml-functions.md) to learn more about the available functions.

## Multi-metric jobs [multi-metric-jobs]

Multi-metric jobs can have more than one detector configured and optionally split the analysis by a field. Conceptually, multi-metric jobs can be considered as multiple independent single metric jobs. Binding the jobs together into a multi-metric job has the advantage of an overall anomaly score (instead of an independent anomaly score for each job) and influencers that apply to all metrics in the job. Multi-metrics jobs provide better results when the influencers are shared across the detectors.

Splitting the analysis by a field enables you to model each value of that field independently. For example, you can split the analysis of your log data set by the `host` field which results in independent baselines for each host (each value of the `host` field) in your data set. If you have a `count` function that detects anomalies in the `error_code` field, and your data is split by the `host` field, then the unusual number of events in the `error_code` field is reported in the context of each host independently. In this case, an observed anomaly in one host does not affect the baseline of another host.

Multi-metric jobs are recommended for complex use cases where you want to detect anomalous behavior in multiple aspects of your data or analyze the data in the context of distinct values of a field.

## Population jobs [population-jobs]

In the case of the population jobs, the analyzed data is split by the distinct values of a field. This field defines what is called a population. The splits are analyzed in the context of all the splits to find unusual values in the population. In other words, the population analysis is a comparison of an individual entity against a collective model of all members in the population as witnessed over time.

For example, if you want to detect IP addresses with unusual request rates compared to the number of requests coming from other IP addresses, you can use a population job. That job has a `count` function to detect unusual number of requests and the analysis is split by the `client_ip` field. In this context, an event is anomalous if the request rate of an IP address is unusually high or low compared to the request rate of all IP addresses in the population. The population job builds a model of the typical number of requests for the IP addresses collectively and compares the behavior of each IP address against that collective model to detect outliers.

Refer to [Performing population analysis](/explore-analyze/machine-learning/anomaly-detection/ml-anomaly-detection-job-types.md) to learn more.

## Advanced jobs [advanced-jobs]

Advanced jobs give you all the flexibility that’s possible in the [create {{anomaly-jobs}} API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-ml-put-job). At the extreme, you can switch to directly edit the JSON that will be sent to this endpoint. All the other types of jobs described in this page *can* be created as advanced jobs, but the more specialized wizards make it easier to create jobs for common situations. You can create an advanced job if you are familiar with all the functionality that {{ml}} {{anomaly-detect}} provides and want to do something that the more specialized wizards do not allow you to do.

## Categorization jobs [categorization-jobs]

Categorization jobs cluster similar text values together, classify them into categories, and detect anomalies within the categories. Categorization works best on machine-written text like log messages that typically contains repeated strings of text; it does not work well on human-generated text because of its high variability.

The model learns the normal volume and pattern of a category over time so the job can detect anomalous behavior, such as an unusual number of events in a category by using the `count` function or messages that rarely occur by using the `rare` function.

Refer to [Detecting anomalous categories of data](ml-configuring-categories.md) to learn more.

## Rare jobs [rare-jobs]

Rare {{anomaly-jobs}} detect rare occurrences in time series data. Rare jobs use the `rare` or `freq_rare` functions and detect such events in populations as well. A *rare* job finds events in simple time series data that occur rarely compared to what the model observed over time. A *rare in a population* job finds members of a population that have rare values over time compared to the other members of the population. The *frequently rare in a population* job detects rare events that frequently occur for a member of a population compared to other members. As an example of this last type of rare job, you can create one that models URI paths and client IP interactions and detects a rare URI path that is visited by very few client IPs in the population (this is the reason why it’s rare). The client IPs that have many interactions with this URI path are anomalous compared to the rest of the population that rarely interact with the URI path.

## Geo jobs [geo-jobs]

Geo {{anomaly-jobs}} detect unusual occurrences in the geographic locations of your data. Your data set must contain geo data to be able to use the `lat_long` function in the detector to detect anomalous geo data. Geo jobs can identify, for example, transactions that are initiated from locations that are unusual compared to the locations of the rest of the transactions.
