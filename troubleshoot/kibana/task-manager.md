---
navigation_title: Task Manager
mapped_pages:
  - https://www.elastic.co/guide/en/kibana/current/task-manager-troubleshooting.html
applies_to:
  stack: preview
products:
  - id: kibana
---

# Troubleshoot {{kib}} Task Manager [task-manager-troubleshooting]


Task Manager is used by a wide range of services in {{kib}}, such as [Alerting](../../deploy-manage/production-guidance/kibana-alerting-production-considerations.md), Actions, Reporting, and Telemetry. Unexpected behavior in these services might be a downstream issue originating in Task Manager.

This page describes how to resolve common problems you might encounter with Task Manager. If your problem isn’t described here, review open issues in the following GitHub repositories:

* [{{kib}}](https://github.com/elastic/kibana/issues) ([Task Manager issues](https://github.com/elastic/kibana/issues?q=is%3Aopen+is%3Aissue+label%3A%22Feature%3ATask+Manager%22))

Have a question? Contact us in the [discuss forum](https://discuss.elastic.co/).


### Tasks with small schedule intervals run late [task-manager-health-scheduled-tasks-small-schedule-interval-run-late]

**Problem**:

Tasks are scheduled to run every 2 seconds, but seem to be running late.

**Solution**:

Task Manager polls for tasks at the cadence specified by the [`xpack.task_manager.poll_interval`](kibana://reference/configuration-reference/task-manager-settings.md#task-manager-settings) setting, which is 3 seconds by default. This means that a task could run late if it uses a schedule that is smaller than this setting.

You can adjust the [`xpack.task_manager.poll_interval`](kibana://reference/configuration-reference/task-manager-settings.md#task-manager-settings) setting.  However, this will add additional load to both {{kib}} and {{es}} instances in the cluster, as they will perform more queries.


### Tasks run late [task-manager-health-tasks-run-late]

**Problem**:

The most common symptom of an underlying problem in Task Manager is that tasks appear to run late. For instance, recurring tasks might run at an inconsistent cadence, or long after their scheduled time.

**Solution**:

By default, {{kib}} polls for tasks at a rate of 10 tasks every 3 seconds.

If many tasks are scheduled to run at the same time, pending tasks will queue in {{es}}. Each {{kib}} instance then polls for pending tasks at a rate of up to 10 tasks at a time, at 3 second intervals. It is possible for pending tasks in the queue to exceed this capacity and run late as a result.

This type of delay is known as *drift*.The root cause for drift depends on the specific usage, and there are no hard and fast rules for addressing drift.

For example:

* If drift is caused by **an excess of concurrent tasks** relative to the available capacity of {{kib}} instances in the cluster, expand the throughput of the cluster.
* If drift is caused by **long running tasks** that overrun their scheduled cadence,  reconfigure the tasks in question.

Refer to [Diagnose a root cause for drift](#task-manager-diagnosing-root-cause) for step-by-step instructions on identifying the correct resolution.

*Drift* is often addressed by adjusting the scaling the deployment to better suit your usage. For details on scaling Task Manager, see [Scaling guidance](../../deploy-manage//production-guidance/kibana-task-manager-scaling-considerations.md#task-manager-scaling-guidance).

## Diagnose a root cause for drift [task-manager-diagnosing-root-cause]

::::{warning}
This functionality is in technical preview and may be changed or removed in a future release. Elastic will work to fix any issues, but features in technical preview are not subject to the support SLA of official GA features.
::::


The following guide helps you identify a root cause for *drift* by making sense of the output from the [Health monitoring](../../deploy-manage/monitor/kibana-task-manager-health-monitoring.md) endpoint.

By analyzing the different sections of the output, you can evaluate different theories that explain the drift in a deployment.

* [Evaluate the Configuration](#task-manager-health-evaluate-the-configuration)

    * [{{kib}} is configured to poll for tasks at a reduced rate](#task-manager-theory-reduced-polling-rate)

* [Evaluate the Runtime](#task-manager-health-evaluate-the-runtime)

    * [{{kib}} is not actually polling as frequently as it should](#task-manager-theory-actual-polling-frequently)
    * [{{kib}} is polling as frequently as it should, but that isn’t often enough to keep up with the workload](#task-manager-theory-insufficient-throughput)
    * [Tasks run for too long, overrunning their schedule](#task-manager-theory-long-running-tasks)
    * [Tasks take multiple attempts to succeed](#task-manager-theory-high-fail-rate)

* [Evaluate the Workload](#task-manager-health-evaluate-the-workload)
* [Evaluate the Capacity Estimation](#task-manager-health-evaluate-the-capacity-estimation)

Retrieve the latest monitored health stats of a {{kib}} instance Task Manager:

```bash
$ curl -X GET api/task_manager/_health
```

The API returns the following:

```json
{
  "id": "15415ecf-cdb0-4fef-950a-f824bd277fe4",
  "timestamp": "2021-02-16T11:38:10.077Z",
  "status": "OK",
  "last_update": "2021-02-16T11:38:09.934Z",
  "stats": {
    "configuration": {
      "timestamp": "2021-02-16T11:29:05.055Z",
      "value": {
        "request_capacity": 1000,
        "monitored_aggregated_stats_refresh_rate": 60000,
        "monitored_stats_running_average_window": 50,
        "monitored_task_execution_thresholds": {
          "default": {
            "error_threshold": 90,
            "warn_threshold": 80
          },
          "custom": {}
        },
        "poll_interval": 3000,
        "max_workers": 10
      },
      "status": "OK"
    },
    "runtime": {
      "timestamp": "2021-02-16T11:38:09.934Z",
      "value": {
        "polling": {
          "last_successful_poll": "2021-02-16T11:38:09.934Z",
          "last_polling_delay": "2021-02-16T11:29:05.053Z",
          "duration": {
            "p50": 13,
            "p90": 128,
            "p95": 143,
            "p99": 168
          },
          "claim_conflicts": {
            "p50": 0,
            "p90": 0,
            "p95": 0,
            "p99": 0
          },
          "claim_mismatches": {
            "p50": 0,
            "p90": 0,
            "p95": 0,
            "p99": 0
          },
          "result_frequency_percent_as_number": {
            "Failed": 0,
            "NoAvailableWorkers": 0,
            "NoTasksClaimed": 80,
            "RanOutOfCapacity": 0,
            "RunningAtCapacity": 0,
            "PoolFilled": 20
          }
        },
        "drift": {
          "p50": 99,
          "p90": 1245,
          "p95": 1845,
          "p99": 2878
        },
        "load": {
          "p50": 0,
          "p90": 0,
          "p95": 10,
          "p99": 20
        },
        "execution": {
          "duration": {
            "alerting:.index-threshold": {
              "p50": 95,
              "p90": 1725,
              "p95": 2761,
              "p99": 2761
            },
            "alerting:xpack.uptime.alerts.monitorStatus": {
              "p50": 149,
              "p90": 1071,
              "p95": 1171,
              "p99": 1171
            },
            "actions:.index": {
              "p50": 166,
              "p90": 166,
              "p95": 166,
              "p99": 166
            }
          },
          "persistence": {
            "recurring": 88,
            "non_recurring": 4,
          },
          "result_frequency_percent_as_number": {
            "alerting:.index-threshold": {
              "Success": 100,
              "RetryScheduled": 0,
              "Failed": 0,
              "status": "OK"
            },
            "alerting:xpack.uptime.alerts.monitorStatus": {
              "Success": 100,
              "RetryScheduled": 0,
              "Failed": 0,
              "status": "OK"
            },
            "actions:.index": {
              "Success": 10,
              "RetryScheduled": 0,
              "Failed": 90,
              "status": "error"
            }
          }
        }
      },
      "status": "OK"
    },
    "workload": {
      "timestamp": "2021-02-16T11:38:05.826Z",
      "value": {
        "count": 26,
        "task_types": {
          "alerting:.index-threshold": {
            "count": 2,
            "status": {
              "idle": 2
            }
          },
          "actions:.index": {
            "count": 14,
            "status": {
              "idle": 2,
              "running": 2,
              "failed": 10
            }
          },
          "alerting:xpack.uptime.alerts.monitorStatus": {
            "count": 10,
            "status": {
              "idle": 10
            }
          },
        },
        "schedule": [
          ["10s", 2],
          ["1m", 2],
          ["60s", 2],
          ["5m", 2],
          ["60m", 4],
          ["3600s", 1],
          ["720m", 1]
        ],
        "non_recurring": 18,
        "owner_ids": 0,
        "overdue": 10,
        "overdue_non_recurring": 10,
        "estimated_schedule_density": [0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 1, 1, 1, 0, 0, 3, 0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 1, 1, 1, 0],
        "capacity_requirements": {
          "per_minute": 6,
          "per_hour": 28,
          "per_day": 2
        }
      },
      "status": "OK"
    },
    "capacity_estimation": {
      "timestamp": "2021-02-16T11:38:06.826Z",
      "value": {
        "observed": {
          "observed_kibana_instances": 1,
          "max_throughput_per_minute_per_kibana": 200,
          "max_throughput_per_minute": 200,
          "minutes_to_drain_overdue": 1,
          "avg_recurring_required_throughput_per_minute": 28,
          "avg_recurring_required_throughput_per_minute_per_kibana": 28,
          "avg_required_throughput_per_minute": 28,
          "avg_required_throughput_per_minute_per_kibana": 28
        },
        "proposed": {
          "min_required_kibana": 1,
          "provisioned_kibana": 1,
          "avg_recurring_required_throughput_per_minute_per_kibana": 28,
          "avg_required_throughput_per_minute_per_kibana": 28
        }
      }
      "status": "OK"
    }
  }
}
```

### Evaluate the Configuration [task-manager-health-evaluate-the-configuration]

$$$task-manager-theory-reduced-polling-rate$$$
**Theory**: {{kib}} is configured to poll for tasks at a reduced rate.

**Diagnosis**: Evaluating the health stats, you can see the following output under `stats.configuration.value`:

```json
{
  "request_capacity": 1000,
  "monitored_aggregated_stats_refresh_rate": 60000,
  "monitored_stats_running_average_window": 50,
  "monitored_task_execution_thresholds": {
    "default": {
      "error_threshold": 90,
      "warn_threshold": 80
    },
    "custom": {}
  },
  "poll_interval": 3000, <1>
  "max_workers": 10 <2>
}
```

1. `poll_interval` is set to the default value of 3000 milliseconds
2. `max_workers` is set to the default value of 10 workers


You can infer from this output that the {{kib}} instance polls for work every 3 seconds and can run 10 concurrent tasks.

Now suppose the output under `stats.configuration.value` is the following:

```json
{
  "request_capacity": 1000,
  "monitored_aggregated_stats_refresh_rate": 60000,
  "monitored_stats_running_average_window": 50,
  "monitored_task_execution_thresholds": {
    "default": {
      "error_threshold": 90,
      "warn_threshold": 80
    },
    "custom": {}
  },
  "poll_interval": 60000, <1>
  "max_workers": 1 <2>
}
```

1. `poll_interval` is set to 60000 milliseconds, far higher than the default
2. `max_workers` is set to 1 worker, far lower than the default


You can infer from this output that the {{kib}} instance only polls for work once a minute and only picks up one task at a time. This throughput is unlikely to support mission critical services, such as Alerting or Reporting, and tasks will usually run late.

There are two possible reasons for such a configuration:

* These settings have been configured manually, which can be resolved by reconfiguring these settings. For details, see [Task Manager Settings](kibana://reference/configuration-reference/task-manager-settings.md).
* {{kib}} has reduced its own throughput in reaction to excessive load on the {{es}} cluster.

    Task Manager is equipped with a reactive self-healing mechanism in response to an increase in load related errors in {{es}}. This mechanism will increase the `poll_interval` setting (reducing the rate at which it queries {{es}}), and decrease the `max_workers` (reducing the amount of operations it executes against {{es}}). Once the error rate reduces, these settings are incrementally dialed up again, returning them to the configured settings.

    This scenario can be identified by searching the {{kib}} Server Log for messages such as:

    ```txt
    Max workers configuration is temporarily reduced after Elasticsearch returned 25 "too many request" error(s).
    ```

    Deeper investigation into the high error rate experienced by the {{es}} cluster is required.



### Evaluate the Runtime [task-manager-health-evaluate-the-runtime]

$$$task-manager-theory-actual-polling-frequently$$$
**Theory**: {{kib}} is not polling as frequently as it should

**Diagnosis**: Evaluating the health stats, you see the following output under `stats.runtime.value.polling`:

```json
{
  "last_successful_poll": "2021-02-16T11:38:09.934Z", <1>
  "last_polling_delay": "2021-02-14T11:29:05.053Z",
  "duration": { <2>
    "p50": 13,
    "p90": 128,
    "p95": 143,
    "p99": 168
  },
  "claim_conflicts": { <3>
    "p50": 0,
    "p90": 0,
    "p95": 0,
    "p99": 2
  },
  "claim_mismatches": {
    "p50": 0,
    "p90": 0,
    "p95": 0,
    "p99": 0
  },
  "result_frequency_percent_as_number": { <4>
    "Failed": 0,
    "NoAvailableWorkers": 0,
    "NoTasksClaimed": 80,
    "RanOutOfCapacity": 0,
    "RunningAtCapacity": 0,
    "PoolFilled": 20
  }
}
```

1. Ensure the last successful polling cycle was completed no more than a couple of multiples of `poll_interval` in the past.
2. Ensure the duration of polling cycles is usually below 100ms. Longer durations are possible, but unexpected.
3. Ensure {{kib}} instances in the cluster are not encountering a high rate of version conflicts.
4. Ensure the majority of polling cycles result in positive outcomes, such as `RunningAtCapacity` or `PoolFilled`.


You can infer from this output that the {{kib}} instance is polling regularly. This assessment is based on the following:

* Comparing the `last_successful_poll` to the `timestamp` (value of `2021-02-16T11:38:10.077Z`) at the root, where you can see the last polling cycle took place 1 second before the monitoring stats were exposed by the health monitoring API.
* Comparing the `last_polling_delay` to the `timestamp` (value of `2021-02-16T11:38:10.077Z`) at the root, where you can see the last polling cycle delay took place 2 days ago, suggesting {{kib}} instances are not conflicting often.
* The `p50` of the `duration` shows that at least 50% of polling cycles take, at most, 13 milliseconds to complete.
* Evaluating the `result_frequency_percent_as_number`:

    * 80% of the polling cycles completed without claiming any tasks (suggesting that there aren’t any overdue tasks).
    * 20% completed with Task Manager claiming tasks that were then executed.
    * None of the polling cycles ended up occupying all of the available workers, as `RunningAtCapacity` has a frequency of 0%, suggesting there is enough capacity in Task Manager to handle the workload.


All of these stats are tracked as a running average, which means that they give a snapshot of a period of time (by default {{kib}} tracks up to 50 cycles), rather than giving a complete history.

Suppose the output under `stats.runtime.value.polling.result_frequency_percent_as_number` was the following:

```json
{
  "Failed": 30, <1>
  "NoAvailableWorkers": 20, <2>
  "NoTasksClaimed": 10,
  "RanOutOfCapacity": 10, <3>
  "RunningAtCapacity": 10, <4>
  "PoolFilled": 20
}
```

1. 30% of polling cycles failed, which is a high rate.
2. 20% of polling cycles are skipped as Task Manager has no capacity left to run tasks.
3. 10% of polling cycles result in Task Manager claiming more tasks than it has capacity to run.
4. 10% of polling cycles result in Task Manager claiming precisely as many tasks as it has capacity to run.


You can infer from this output that Task Manager is not healthy, as the failure rate is high, and Task Manager is fetching tasks it has no capacity to run. Analyzing the {{kib}} Server Log should reveal the underlying issue causing the high error rate and capacity issues.

The high `NoAvailableWorkers` rate of 20% suggests that there are many tasks running for durations longer than the `poll_interval`. For details on analyzing long task execution durations, see the [long running tasks](#task-manager-theory-long-running-tasks) theory.

$$$task-manager-theory-insufficient-throughput$$$
**Theory**: {{kib}} is polling as frequently as it should, but that isn’t often enough to keep up with the workload

**Diagnosis**: Evaluating the health stats, you can see the following output of `drift` and `load` under `stats.runtime.value`:

```json
{
  "drift": { <1>
    "p50": 99,
    "p90": 1245,
    "p95": 1845,
    "p99": 2878
  },
  "load": { <2>
    "p50": 0,
    "p90": 0,
    "p95": 10,
    "p99": 20
  },
}
```

1. `drift` shows us that at least 95% of tasks are running within 2 seconds of their scheduled time.
2. `load` shows us that Task Manager is idle at least 90% of the time, and never uses more than 20% of its available workers.


You can infer from these stats that this {{kib}} has plenty of capacity, and any delays you might be experiencing are unlikely to be addressed by expanding the throughput.

Suppose the output of `drift` and `load` was the following:

```json
{
  "drift": { <1>
    "p50": 2999,
    "p90": 3845,
    "p95": 3845.75,
    "p99": 4078
  },
  "load": { <2>
    "p50": 80,
    "p90": 100,
    "p95": 100,
    "p99": 100
  }
}
```

1. `drift` shows us that all tasks are running 3 to 4 seconds after their scheduled time.
2. `load` shows us that at least half of the time Task Manager is running at a load of 80%.


You can infer from these stats that this {{kib}} is using most of its capacity, but seems to keep up with the work most of the time. This assessment is based on the following:

* The `p90` of `load` is at 100%, and `p50` is also quite high at 80%. This means that there is little to no room for maneuvering, and a spike of work might cause Task Manager to exceed its capacity.
* Tasks run soon after their scheduled time, which is to be expected. A `poll_interval` of `3000` milliseconds would often experience a consistent drift of somewhere between `0` and `3000` milliseconds. A `p50 drift` of `2999` suggests that there is room for improvement, and you could benefit from a higher throughput.

For details on achieving higher throughput by adjusting your scaling strategy, see [Scaling guidance](../../deploy-manage/production-guidance/kibana-task-manager-scaling-considerations.md#task-manager-scaling-guidance).

$$$task-manager-theory-long-running-tasks$$$
**Theory**: Tasks run for too long, overrunning their schedule

**Diagnosis**: The [Insufficient throughput to handle the scheduled workload](#task-manager-theory-insufficient-throughput) theory analyzed a hypothetical scenario where both drift and load were unusually high.

Suppose an alternate scenario, where `drift` is high, but `load` is not, such as the following:

```json
{
    "drift": { <1>
        "p50": 9799,
        "p90": 83845,
        "p95": 90328,
        "p99": 123845
    },
    "load": { <2>
        "p50": 40,
        "p90": 75,
        "p95": 80,
        "p99": 100
    }
}
```

1. `drift` shows that most (if not all) tasks are running at least 32 seconds too late.
2. `load` shows that, for the most part, you have capacity to run more concurrent tasks.


In the preceding scenario, the  tasks are running far too late, but you have sufficient capacity to run more concurrent tasks. A high capacity allows {{kib}} to run multiple different tasks concurrently. If a task is already running when its next schedule run is due, {{kib}} will avoid running it a second time, and instead wait for the first execution to complete.

If a task takes longer to execute than the cadence of its schedule, then that task will always overrun and experience a high drift. For example, suppose a task is scheduled to execute every 3 seconds, but takes 6 seconds to complete. It will consistently suffer from a drift of, at least, 3 seconds.

Evaluating the health stats in this hypothetical scenario, you see the following output under `stats.runtime.value.execution.duration`:

```json
{
  "alerting:.index-threshold": { <1>
    "p50": 95,
    "p90": 1725,
    "p95": 2761,
    "p99": 2761
  },
  "alerting:.es-query": { <2>
    "p50": 7149,
    "p90": 40071,
    "p95": 45282,
    "p99": 121845
  },
  "actions:.index": {
    "p50": 166,
    "p90": 166,
    "p95": 166,
    "p99": 166
  }
}
```

1. 50% of the tasks backing index threshold alerts complete in less than 100 milliseconds.
2. 50% of the tasks backing Elasticsearch query alerts complete in 7 seconds, but at least 10% take longer than 40 seconds.


You can infer from these stats that the high drift the Task Manager is experiencing is most likely due to Elasticsearch query alerts that are running for a long time.

Resolving this issue is context dependent and changes from case to case. In the preceding example, this would be resolved by modifying the queries in these alerts to make them faster, or improving the {{es}} throughput to speed up the exiting query.

$$$task-manager-theory-high-fail-rate$$$
**Theory**: Tasks take multiple attempts to succeed

**Diagnosis**: A high error rate could cause a task to appear to run late, when in fact it runs on time, but experiences a high failure rate.

Evaluating the preceding health stats, you see the following output under `stats.runtime.value.execution.result_frequency_percent_as_number`:

```json
{
  "alerting:.index-threshold": { <1>
    "Success": 100,
    "RetryScheduled": 0,
    "Failed": 0,
    "status": "OK"
  },
  "alerting:xpack.uptime.alerts.monitorStatus": {
    "Success": 100,
    "RetryScheduled": 0,
    "Failed": 0,
    "status": "OK"
  },
  "actions:.index": { <2>
    "Success": 8,
    "RetryScheduled": 0,
    "Failed": 92,
    "status": "error" <3>
  }
}
```

1. 100% of the tasks backing index threshold alerts successfully complete.
2. 92% of the tasks backing ES index actions fail to complete.
3. The tasks backing ES index actions have exceeded the default `monitored_task_execution_thresholds` *error* configuration.


You can infer from these stats that most `actions:.index` tasks, which back the ES Index {{kib}} action, fail. Resolving that would require deeper investigation into the {{kib}} Server Log, where the exact errors are logged, and addressing these specific errors.

$$$task-manager-theory-spikes-in-non-recurring-tasks$$$
**Theory**: Spikes in non-recurring tasks are consuming a high percentage of the available capacity

**Diagnosis**: Task Manager uses ad-hoc non-recurring tasks to load balance operations across multiple {{kib}} instances.

Evaluating the preceding health stats, you see the following output under `stats.runtime.value.execution.persistence`:

```json
{
  "recurring": 88, <1>
  "non_recurring": 12, <2>
},
```

1. 88% of executed tasks are recurring tasks
2. 12% of executed tasks are non-recurring tasks


You can infer from these stats that the majority of executions consist of recurring tasks at 88%. You can use the `execution.persistence` stats to evaluate the ratio of consumed capacity, but on their own, you should not make assumptions about the sufficiency of the available capacity.

To assess the capacity, you should evaluate these stats against the `load` under `stats.runtime.value`:

```json
{
    "load": {
        "p50": 40,
        "p90": 40,
        "p95": 60,
        "p99": 80
    }
}
```

You can infer from these stats that it is very unusual for Task Manager to run out of capacity, so the capacity is likely sufficient to handle the amount of non-recurring tasks.

Suppose you have an alternate scenario, where you see the following output under `stats.runtime.value.execution.persistence`:

```json
{
  "recurring": 60, <1>
  "non_recurring": 40, <2>
},
```

1. 60% of executed tasks are recurring tasks
2. 40% of executed tasks are non-recurring tasks


You can infer from these stats that even though most executions are recurring tasks, a substantial percentage of executions are non-recurring tasks at 40%.

Evaluating the `load` under `stats.runtime.value`, you see the following:

```json
{
    "load": {
        "p50": 70,
        "p90": 100,
        "p95": 100,
        "p99": 100
    }
}
```

You can infer from these stats that it is quite common for this {{kib}} instance to run out of capacity. Given the high rate of non-recurring tasks, it would be reasonable to assess that there is insufficient capacity in the {{kib}} cluster to handle the amount of tasks.

Keep in mind that these stats give you a glimpse at a moment in time, and even though there has been insufficient capacity in recent minutes, this might not be true in other times where fewer non-recurring tasks are used. We recommend tracking these stats over time and identifying the source of these tasks before making sweeping changes to your infrastructure.


### Evaluate the Workload [task-manager-health-evaluate-the-workload]

Predicting the required throughput a deployment might need to support Task Manager is difficult, as features can schedule an unpredictable number of tasks at a variety of scheduled cadences.

[Health monitoring](../../deploy-manage/monitor/kibana-task-manager-health-monitoring.md) provides statistics that make it easier to monitor the adequacy of the existing throughput. By evaluating the workload, the required throughput can be estimated, which is used when following the Task Manager [Scaling guidance](../../deploy-manage/production-guidance/kibana-task-manager-scaling-considerations.md#task-manager-scaling-guidance).

Evaluating the preceding health stats in the previous example, you see the following output under `stats.workload.value`:

```json
{
  "count": 26, <1>
  "task_types": {
    "alerting:.index-threshold": {
      "count": 2, <2>
      "status": {
        "idle": 2
      }
    },
    "actions:.index": {
      "count": 14,
      "status": {
        "idle": 2,
        "running": 2,
        "failed": 10 <3>
      }
    },
    "alerting:xpack.uptime.alerts.monitorStatus": {
      "count": 10,
      "status": {
        "idle": 10
      }
    },
  },
  "non_recurring": 0, <4>
  "owner_ids": 1, <5>
  "schedule": [ <6>
    ["10s", 2],
    ["1m", 2],
    ["90s", 2],
    ["5m", 8]
  ],
  "overdue_non_recurring": 0, <7>
  "overdue": 0, <8>
  "estimated_schedule_density": [ <9>
    0, 1, 0, 0, 0, 1, 0, 1, 0, 1,
    0, 0, 0, 1, 0, 0, 1, 1, 1, 0,
    0, 3, 0, 0, 0, 1, 0, 1, 0, 1,
    0, 0, 0, 1, 0, 0, 1, 1, 1, 0
  ],
  "capacity_requirements": { <10>
    "per_minute": 14,
    "per_hour": 240,
    "per_day": 0
  }
}
```

1. There are 26 tasks in the system, including regular tasks, recurring tasks, and failed tasks.
2. There are 2 `idle` index threshold alert tasks, meaning they are scheduled to run at some point in the future.
3. Of the 14 tasks backing the ES index action, 10 have failed and 2 are running.
4. There are no non-recurring tasks in the queue.
5. There is one Task Manager actively executing tasks. There might be additional idle Task Managers, but they aren’t actively executing tasks at this moment in time.
6. A histogram of all scheduled recurring tasks shows that 2 tasks are scheduled to run every 10 seconds, 2  tasks are scheduled to run once a minute, and so on.
7. There are no overdue non-recurring tasks. Non-recurring tasks are usually scheduled to execute immediately, so overdue non-recurring tasks are often a symptom of a congested system.
8. There are no overdue tasks, which means that all tasks that **should** have run by now **have** run.
9. This histogram shows the tasks scheduled to run throughout the upcoming 20 polling cycles. The histogram represents the entire deployment, rather than just this {{kib}} instance.
10. The capacity required to handle the recurring tasks in the system. These are buckets, rather than aggregated sums, and we recommend [evaluating the Capacity Estimation](#task-manager-health-evaluate-the-capacity-estimation) section, rather than evaluating these buckets  yourself.


The `workload` section summarizes the work load across the cluster, listing the tasks in the system, their types, schedules, and current status.

You can infer from these stats that a default deployment should suffice. This assessment is based on the following:

* The estimated schedule density is low.
* There aren’t many tasks in the system relative to the default capacity.

Suppose the output of `stats.workload.value` looked something like this:

```json
{
  "count": 2191, <1>
  "task_types": {
    "alerting:.index-threshold": {
      "count": 202,
      "status": {
        "idle": 183,
        "claiming": 2,
        "running": 19
      }
    },
    "alerting:.es-query": {
      "count": 225,
      "status": {
        "idle": 225,
      }
    },
    "actions:.index": {
      "count": 89,
      "status": {
        "idle": 24,
        "running": 2,
        "failed": 63
      }
    },
    "alerting:xpack.uptime.alerts.monitorStatus": {
      "count": 87,
      "status": {
        "idle": 74,
        "running": 13
      }
    },
  },
  "non_recurring": 0,
  "owner_ids": 1,
  "schedule": [ <2>
    ["10s", 38],
    ["1m", 101],
    ["90s", 55],
    ["5m", 89],
    ["20m", 62],
    ["60m", 106],
    ["1d", 61]
  ],
  "overdue_non_recurring": 0,
  "overdue": 0, <5>
  "estimated_schedule_density": [  <3>
    10, 1, 0, 10, 0, 20, 0, 1, 0, 1,
    9, 0, 3, 10, 0, 0, 10, 10, 7, 0,
    0, 31, 0, 12, 16, 31, 0, 10, 0, 10,
    3, 22, 0, 10, 0, 2, 10, 10, 1, 0
  ],
  "capacity_requirements": {
    "per_minute": 329, <4>
    "per_hour": 4272, <5>
    "per_day": 61 <6>
  }
}
```

1. There are 2,191 tasks in the system.
2. The scheduled tasks are distributed across a variety of cadences.
3. The schedule density shows that you expect to exceed the default 10 concurrent tasks.
4. There are 329 task executions that recur  within the space of every minute.
5. There are 4,273 task executions that recur within the space of every hour.
6. There are 61 task executions that recur within the space of every day.


You can infer several important attributes of your workload from this output:

* There are many tasks in your system and ensuring these tasks run on their scheduled cadence will require attention to the Task Manager throughput.
* Assessing the high frequency tasks (tasks that recur at a cadence of a couple of minutes or less), you must support a throughput of approximately 330 task executions per minute (38 every 10 seconds + 101 every minute).
* Assessing the medium frequency tasks (tasks that recur at a cadence of an hour or less), you must support an additional throughput of over 4,272 task executions per hour (55 every 90 seconds + 89 every 5 minutes, + 62 every 20 minutes + 106 each hour). You can average the needed throughput for the hour by counting these tasks as an additional 70 - 80 tasks per minute.
* Assessing the estimated schedule density, there are cycles that are due to run upwards of 31 tasks concurrently, and along side these cycles, there are empty cycles. You can expect Task Manager to load balance these tasks throughout the empty cycles, but this won’t leave much capacity to handle spikes in fresh tasks that might be scheduled in the future.

These rough calculations give you a lower bound to the required throughput, which is *at least* 410 tasks per minute to ensure recurring tasks are executed, at their scheduled time. This throughput doesn’t account for nonrecurring tasks that might have been scheduled, nor does it account for tasks (recurring or otherwise) that might be scheduled in the future.

Given these inferred attributes, it would be safe to assume that a single {{kib}} instance with default settings **would not** provide the required throughput. It is possible that scaling horizontally by adding a couple more {{kib}} instances will.

For details on scaling Task Manager, see [Scaling guidance](../../deploy-manage/production-guidance/kibana-task-manager-scaling-considerations.md#task-manager-scaling-guidance).


### Evaluate the Capacity Estimation [task-manager-health-evaluate-the-capacity-estimation]

Task Manager is constantly evaluating its runtime operations and workload. This enables Task Manager to make rough estimates about the sufficiency of its capacity.

As the name suggests, these are estimates based on historical data and should not be used as predictions. These estimations should be evaluated alongside the detailed [Health monitoring](../../deploy-manage/monitor/kibana-task-manager-health-monitoring.md) stats before making changes to infrastructure. These estimations assume all {{kib}} instances are configured identically.

We recommend using these estimations when following the Task Manager [Scaling guidance](../../deploy-manage/production-guidance/kibana-task-manager-scaling-considerations.md#task-manager-scaling-guidance).

Evaluating the health stats in the previous example, you can see the following output under `stats.capacity_estimation.value`:

```json
{
  "observed": {
    "observed_kibana_instances": 1, <1>
    "minutes_to_drain_overdue": 1, <2>
    "max_throughput_per_minute_per_kibana": 200,
    "max_throughput_per_minute": 200, <3>
    "avg_recurring_required_throughput_per_minute": 28, <4>
    "avg_recurring_required_throughput_per_minute_per_kibana": 28,
    "avg_required_throughput_per_minute": 28, <5>
    "avg_required_throughput_per_minute_per_kibana": 28
  },
  "proposed": {
    "min_required_kibana": 1, <6>
    "provisioned_kibana": 1, <7>
    "avg_recurring_required_throughput_per_minute_per_kibana": 28,
    "avg_required_throughput_per_minute_per_kibana": 28
  }
}
```

1. These estimates assume that there is one {{kib}} instance actively executing tasks.
2. Based on past throughput the overdue tasks in the system could be executed within 1 minute.
3. Assuming all {{kib}} instances in the cluster are configured the same as this instance, the maximum available throughput is 200 tasks per minute.
4. On average, the recurring tasks in the system have historically required a throughput of 28 tasks per minute.
5. On average, regardless of whether they are recurring or otherwise, the tasks in the system have historically required a throughput of 28 tasks per minute.
6. One {{kib}} instance should be sufficient to run the current recurring workload.
7. We propose waiting for the workload to change before additional {{kib}} instances are provisioned.


The `capacity_estimation` section is made up of two subsections:

* `observed` estimates the current capacity by observing historical runtime and workload statistics
* `proposed` estimates the baseline {{kib}} cluster size and the expected throughput under such a deployment strategy

You can infer from these estimates that the current system is under-utilized and has enough capacity to handle many more tasks than it currently does.

Suppose an alternate scenario, where you see the following output under `stats.capacity_estimation.value`:

```json
{
  "observed": {
    "observed_kibana_instances": 2, <1>
    "max_throughput_per_minute_per_kibana": 200,
    "max_throughput_per_minute": 400, <2>
    "minutes_to_drain_overdue": 12, <3>
    "avg_recurring_required_throughput_per_minute": 354, <4>
    "avg_recurring_required_throughput_per_minute_per_kibana": 177, <5>
    "avg_required_throughput_per_minute": 434, <6>
    "avg_required_throughput_per_minute_per_kibana": 217
  },
  "proposed": {
    "min_required_kibana": 2, <7>
    "provisioned_kibana": 3, <8>
    "avg_recurring_required_throughput_per_minute_per_kibana": 118, <9>
    "avg_required_throughput_per_minute_per_kibana": 145 <10>
  }
}
```

1. These estimates assume that there are two {{kib}} instance actively executing tasks.
2. The maximum available throughput in the system currently is 400 tasks per minute.
3. Based on past throughput the overdue tasks in the system should be executed within 12 minutes.
4. On average, the recurring tasks in the system have historically required a throughput of 354 tasks per minute.
5. On average, each {{kib}} instance utilizes 177 tasks per minute of its capacity to execute recurring tasks.
6. On average the tasks in the system have historically required a throughput of 434 tasks per minute.
7. The system estimates that at least two {{kib}} instances are required to run the current recurring workload.
8. The system recommends provisioning three {{kib}} instances to handle the workload.
9. Once a third {{kib}} instance is provisioned, the capacity utilized by each instance to execute recurring tasks should drop from 177 to 118 tasks per minute.
10. Taking into account historical ad-hoc task execution, we estimate the throughput required of each {{kib}} instance will drop from 217 task per minute to 145, once a third {{kib}} instance is provisioned.


Evaluating by these estimates, we can infer some interesting attributes of our system:

* These estimates are produced based on the assumption that there are two {{kib}} instances in the cluster. This number is based on the number of {{kib}} instances actively executing tasks in recent minutes. At times this number might fluctuate if {{kib}} instances remain idle, so validating these estimates against what you know about the system is recommended.
* There appear to be so many overdue tasks that it would take 12 minutes of executions to catch up with that backlog. This does not take into account tasks that might become overdue during those 12 minutes. Although this congestion might be temporary, the system could also remain consistently under provisioned and might never drain the backlog entirely.
* Evaluating the recurring tasks in the workload, the system requires a throughput of 354 tasks per minute on average to execute tasks on time, which is lower then the estimated maximum throughput of 400 tasks per minute. Once we take into account historical throughput though, we estimate the required throughput at 434 tasks per minute. This suggests that, historically, approximately 20% of tasks have been ad-hoc non-recurring tasks, the scale of which are harder to predict than recurring tasks.

You can infer from these estimates that the capacity in the current system is insufficient and at least one additional {{kib}} instance is required to keep up with the workload.

For details on scaling Task Manager, see [Scaling guidance](../../deploy-manage/production-guidance/kibana-task-manager-scaling-considerations.md#task-manager-scaling-guidance).


### Inline scripts are disabled in {{es}} [task-manager-cannot-operate-when-inline-scripts-are-disabled]

**Problem**:

Tasks are not running, and the server logs contain the following error message:

```txt
[warning][plugins][taskManager] Task Manager cannot operate when inline scripts are disabled in Elasticsearch
```

**Solution**:

Inline scripts are a hard requirement for Task Manager to function. To enable inline scripting, see the Elasticsearch documentation for [configuring allowed script types setting](../../explore-analyze/scripting/modules-scripting-security.md#allowed-script-types-setting).


### What do I do if the Task’s `runAt` is in the past? [task-runat-is-in-the-past]

**Problem**:

Tasks' property `runAt` is in the past.

**Solution**:

Wait a bit before declaring it as a lost cause, as Task Manager might just be falling behind on its work. You should take a look at the Kibana log and see what you can find that relates to Task Manager. In a healthy environment you should see a log line that indicates that Task Manager was successfully started when Kibana was:

```txt
server log [12:41:33.672] [info][plugins][taskManager][taskManager] TaskManager is identified by the Kibana UUID: 5b2de169-2785-441b-ae8c-186a1936b17d
```

If you see that message and no other errors that relate to Task Manager, it’s most likely that Task Manager is running fine and has simply not had the chance to pick the task up yet. If, on the other hand, the runAt is severely overdue, then it’s worth looking for other Task Manager or alerting-related errors, as something else may have gone wrong. It’s worth looking at the status field, as it might have failed, which would explain why it hasn’t been picked up or it might be running which means the task might simply be a very long running one.


### What do I do if the Task is marked as failed? [task-marked-failed]

**Problem**:

Tasks marked as failed.

**Solution**:

Broadly speaking the Alerting framework is meant to gracefully handle the cases where a task is failing by rescheduling a fresh run in the future. If this fails to happen, then that means something has gone wrong in the underlying implementation and this isn’t expected. Ideally you should try and find any log lines that relate to this rule and its task, and use these to help us investigate further.


### Task Manager Kibana Log [task-manager-kibana-log]

Task manager will write log lines to the Kibana Log on certain occasions. Below are some common log lines and what they mean.

Task Manager has run out of Available Workers:

```txt
server log [12:41:33.672] [info][plugins][taskManager][taskManager] [Task Ownership]: Task Manager has skipped Claiming Ownership of available tasks at it has ran out Available Workers.
server log [12:41:33.672] [warn][plugins][taskManager][taskManager] taskManager plugin is now degraded: Task Manager is unhealthy - Reason: setting HealthStatus.Error because of expired hot timestamps
```

This log message tells us that Task Manager is not managing to keep up with the sheer amount of work it has been tasked with completing. This might mean that rules are not running at the frequency that was expected (instead of running every 5 minutes, it runs every 7-8 minutes, just as an example).

By default Task Manager is limited to 10 tasks and this can be bumped up by setting a higher number in the [`kibana.yml`](/deploy-manage/stack-settings.md) file using the `xpack.task_manager.capacity` configuration. It is important to keep in mind that a higher number of tasks running at any given time means more load on both Kibana and Elasticsearch; only change this setting if increasing load in your environment makes sense.

Another approach to addressing this might be to tell workers to run at a higher rate, rather than adding more of them, which would be configured using xpack.task_manager.poll_interval. This value dictates how often Task Manager checks to see if there’s more work to be done and uses milliseconds (by default it is 3000, which means an interval of 3 seconds).

Before changing either of these numbers it’s highly recommended to investigate what Task Manager can’t keep up - Are there an unusually high number of rules in the system? Are rules failing often, forcing Task Manager to re-run them constantly? Is Kibana under heavy load? There could be a variety of issues, none of which should be solved by simply changing these configurations.

Task TaskType failed in attempt to run:

```txt
server log [12:41:33.672] [info][plugins][taskManager][taskManager] Task TaskType "alerting:example.always-firing" failed in attempt to run: Unable to load resource ‘/api/something’
```

This log message tells us that when Task Manager was running one of our rules, it’s task errored and, as a result, failed. In this case we can tell that the rule that failed was of type alerting:example.always-firing and that the reason it failed was Unable to load resource ‘/api/something’ . This is a contrived example, but broadly, if you see a message with this kind of format, then this tells you a lot about where the problem might be.

For example, in this case, we’d expect to see a corresponding log line from the Alerting framework itself, saying that the rule failed. You should look in the Kibana log for a line similar to the log line below (probably shortly before the Task Manager log line):

Executing Rule "27559295-44e4-4983-aa1b-94fe043ab4f9" has resulted in Error: Unable to load resource ‘/api/something’

This would confirm that the error did in fact happen in the rule itself (rather than the Task Manager) and it would help us pin-point the specific ID of the rule which failed: 27559295-44e4-4983-aa1b-94fe043ab4f9

We can now use the ID to find out more about that rule by using the http endpoint to find that rule’s configuration and current state to help investigate what might have caused the issue.
