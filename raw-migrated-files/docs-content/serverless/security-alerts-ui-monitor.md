# Monitor and troubleshoot rule executions [security-alerts-ui-monitor]

Several tools can help you gain insight into the performance of your detection rules:

* [Rule Monitoring tab](../../../troubleshoot/security/detection-rules.md#rule-monitoring-tab) — The current state of all detection rules and their most recent executions. Go to the **Rule Monitoring** tab to get an overview of which rules are running, how long they’re taking, and if they’re having any trouble.
* [Execution results](../../../troubleshoot/security/detection-rules.md#rule-execution-logs) — Historical data for a single detection rule’s executions over time. Consult the execution results to understand how a particular rule is running and whether it’s creating the alerts you expect.
* [Detection rule monitoring dashboard](../../../solutions/security/dashboards/detection-rule-monitoring-dashboard.md) — Visualizations to help you monitor the overall health and performance of {{elastic-sec}}'s detection rules. Consult this dashboard for a high-level view of whether your rules are running successfully and how long they’re taking to run, search data, and create alerts.

Refer to the [Troubleshoot missing alerts](../../../troubleshoot/security/detection-rules.md#troubleshoot-signals) section below for strategies on adjusting rules if they aren’t creating the expected alerts.


## Rule Monitoring tab [rule-monitoring-tab]

To view a summary of all rule executions, including the most recent failures and execution times, select the **Rule Monitoring** tab on the **Rules*** page (***Rules*** → ***Detection rules (SIEM)** → **Rule Monitoring**).

:::{image} ../../../images/serverless--detections-monitor-table.png
:alt:  detections monitor table
:class: screenshot
:::

On the **Rule Monitoring** tab, you can [sort and filter rules](../../../solutions/security/detect-and-alert/manage-detection-rules.md#sort-filter-rules) just like you can on the **Installed Rules** tab.

::::{tip}
To sort the rules list, click any column header. To sort in descending order, click the column header again.

::::


For detailed information on a rule, the alerts it generated, and associated errors, click on its name in the table. This also allows you to perform the same actions that are available on the [**Installed Rules** tab](../../../solutions/security/detect-and-alert/manage-detection-rules.md), such as modifying or deleting rules, activating or deactivating rules, exporting or importing rules, and duplicating prebuilt rules.


## Execution results [rule-execution-logs]

Each detection rule execution is logged, including the execution type, the execution’s success or failure, any warning or error messages, how long it took to search for data, create alerts, and complete. This can help you troubleshoot a particular rule if it isn’t behaving as expected (for example, if it isn’t creating alerts or takes a long time to run).

To access a rule’s execution log, go to **Rules** → **Detection rules (SIEM)**, click the rule’s name to open its details, then scroll down and select the **Execution results** tab. Within the Execution log table, you can click the arrow at the end of a row to expand a long warning or error message.

:::{image} ../../../images/serverless--detections-rule-execution-logs.png
:alt: Rule execution log on the rule execution results tab
:class: screenshot
:::

You can hover over each column heading to display a tooltip about that column’s data. Click a column heading to sort the table by that column.

Use these controls to filter what’s included in the logs table:

* The **Run type** drop-down filters the table by rule execution type:

    * **Scheduled**: Automatic, scheduled rule executions.
    * **Manual**: Rule executions that were [started manually](../../../solutions/security/detect-and-alert/manage-detection-rules.md#manually-run-rules).

* The **Status** drop-down filters the table by rule execution status:

    * **Succeeded**: The rule completed its defined search. This doesn’t necessarily mean it generated an alert, just that it ran without error.
    * **Failed**: The rule encountered an error that prevented it from running. For example, a {{ml}} rule whose corresponding {{ml}} job wasn’t running.
    * **Warning**: Nothing prevented the rule from running, but it might have returned unexpected results. For example, a custom query rule tried to search an index pattern that couldn’t be found in {{es}}.

* The date and time picker sets the time range of rule executions included in the table. This is separate from the global date and time picker at the top of the rule details page.
* The **Source event time range** button toggles the display of data pertaining to the time range of manual runs.
* The **Show metrics columns** toggle includes more or less data in the table, pertaining to the timing of each rule execution.
* The **Actions** column allows you to show alerts generated from a given rule execution. Click the filter icon (![Filter](../../../images/serverless-filterInCircle.svg "")) to create a global search filter based on the rule execution’s ID value. This replaces any previously applied filters, changes the global date and time range to 24 hours before and after the rule execution, and displays a confirmation notification. You can revert this action by clicking **Restore previous filters** in the notification.


### Manual runs table [manual-runs-table]

[beta]

Each manual run can produce multiple rule executions, depending on the time range of the run and the rule’s execution schedule. These details are shown in the Manual runs table.

To access the Manual runs table, navigate to the detection rules page, click the rule’s name to open its details, then scroll down and select the **Execution results** tab. Scroll down again to find the Manual runs table.

To stop an active run, go to the appropriate row and click **Stop run** in the **Actions** column. Completed rule executions for each manual run are logged in the Execution log table.

:::{image} ../../../images/serverless--detections-manual-rule-run-table.png
:alt: Manual rule runs table on the rule execution results tab
:class: screenshot
:::

The Manual runs table displays important details such as:

* The status of each manual run:

    * **Pending**: The rule is not yet running.
    * **Running**: The rule is executing during the time range you specified. Some rules, such as indicator match rules, can take longer to run.
    * **Error**: The rule’s configuration is preventing it from running correctly. For example, the rule’s conditions cannot be validated.

* When a manual run started and the time in which it will run
* The number of rule executions that are failing, pending, running, and completed for a manual run
* The total number of rule executions that are occurring for a manual run


## Troubleshoot missing alerts [troubleshoot-signals]

When a rule fails to run close to its scheduled time, some alerts may be missing. There are a number of ways to try to resolve this issue:

* [Troubleshoot gaps](../../../troubleshoot/security/detection-rules.md#troubleshoot-gaps)
* [Troubleshoot ingestion pipeline delay](../../../troubleshoot/security/detection-rules.md#troubleshoot-ingestion-pipeline-delay)

You can also use Task Manager in {{kib}} to troubleshoot background tasks and processes that may be related to missing alerts:

* [Task Manager health monitoring](../../../deploy-manage/monitor/kibana-task-manager-health-monitoring.md)
* [Task Manager troubleshooting](../../../troubleshoot/kibana/task-manager.md)


### Troubleshoot maximum alerts warning [troubleshoot-max-alerts]

When a rule reaches the maximum number of alerts it can generate during a single rule execution, the following warning appears on the rule’s details page and in the rule execution log: `This rule reached the maximum alert limit for the rule execution. Some alerts were not created.`

If you receive this warning, go to the rule’s **Alerts** tab and check for anything unexpected. Unexpected alerts might be created from data source issues or queries that are too broadly scoped. To further reduce alert volume, you can also add [rule exceptions](../../../solutions/security/detect-and-alert/add-manage-exceptions.md) or [suppress alerts](../../../solutions/security/detect-and-alert/suppress-detection-alerts.md).


### Troubleshoot gaps [troubleshoot-gaps]

If you see values in the Gaps column in the Rule Monitoring table or on the Rule details page for a small number of rules, you can increase those rules' Additional look-back time (**Rules** → **Detection rules (SIEM)*** → the rule’s ***All actions*** menu (*…​*) → ***Edit rule settings*** → ***Schedule** → **Additional look-back time**).

It’s recommended to set the `Additional look-back time` to at least 1 minute. This ensures there are no missing alerts when a rule doesn’t run exactly at its scheduled time.

{{elastic-sec}} prevents duplication. Any duplicate alerts that are discovered during the `Additional look-back time` are *not* created.

::::{note}
If the rule that experiences gaps is an indicator match rule, see [how to tune indicator match rules](../../../solutions/security/detect-and-alert/tune-detection-rules.md#tune-indicator-rules). Also please note that {{elastic-sec}} provides [limited support for indicator match rules](../../../solutions/security/detect-and-alert.md#support-indicator-rules).

::::


If you see gaps for numerous rules:

* If you restarted {{kib}} when many rules were activated, try deactivating them and then reactivating them in small batches at staggered intervals. This ensures {{kib}} does not attempt to run all the rules at the same time.
* Consider adding another {{kib}} instance to your environment.


### Troubleshoot ingestion pipeline delay [troubleshoot-ingestion-pipeline-delay]

Even if your rule runs at its scheduled time, there might still be missing alerts if your ingestion pipeline delay is greater than your rule interval + additional look-back time. Prebuilt rules have a minimum interval + additional look-back time of 6 minutes. To avoid missed alerts for prebuilt rules, use caution to ensure that ingestion pipeline delays remain below 6 minutes.

In addition, use caution when creating custom rule schedules to ensure that the specified interval + additional look-back time is greater than your deployment’s ingestion pipeline delay.

You can reduce the number of missed alerts due to ingestion pipeline delay by specifying the `Timestamp override` field value to `event.ingested` in [advanced settings](../../../solutions/security/detect-and-alert/create-detection-rule.md#rule-ui-advanced-params) during rule creation or editing. The detection engine uses the value from the `event.ingested` field as the timestamp when executing the rule.

For example, say an event occurred at 10:00 but wasn’t ingested into {{es}} until 10:10 due to an ingestion pipeline delay. If you created a rule to detect that event with an interval + additional look-back time of 6 minutes, and the rule executes at 10:12, it would still detect the event because the `event.ingested` timestamp was from 10:10, only 2 minutes before the rule executed and well within the rule’s 6-minute interval + additional look-back time.

:::{image} ../../../images/serverless--detections-timestamp-override.png
:alt:  detections timestamp override
:class: screenshot
:::
