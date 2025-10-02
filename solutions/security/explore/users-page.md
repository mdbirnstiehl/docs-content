---
mapped_pages:
  - https://www.elastic.co/guide/en/security/current/users-page.html
  - https://www.elastic.co/guide/en/serverless/current/security-users-page.html
applies_to:
  stack: all
  serverless:
    security: all
products:
  - id: security
  - id: cloud-serverless
---

# Users page

The Users page provides a comprehensive overview of user data to help you understand authentication and user behavior within your environment. Key performance indicator (KPI) charts, data tables, and interactive widgets let you view specific data and drill down for deeper insights.

:::{image} /solutions/images/security-users-page.png
:alt: User's page
:screenshot:
:::

The Users page has the following sections:


## User KPI (key performance indicator) charts [_user_kpi_key_performance_indicator_charts]

KPI charts show the total number of users and successful and failed user authentications within the time range specified in the date picker. Data in the KPI charts is visualized through linear and bar graphs.

::::{tip}
Hover inside a KPI chart to display the actions menu (**…**), where you can perform these actions: inspect, open in Lens, and add to a new or existing case.
::::



## Data tables [_data_tables]

Beneath the KPI charts are data tables, which are useful for viewing and investigating specific types of data. Select the relevant tab to view the following details:

* **Events**: Ingested events that contain the `user.name` field. You can stack by the `event.action`, `event.dataset`, or `event.module` field. To display alerts received from external monitoring tools, scroll down to the Events table and select **Show only external alerts** on the right.
* **All users**: A chronological list of unique user names, when they were last active, and the associated domains.
* **Authentications**: A chronological list of user authentication events and associated details, such as the number of successes and failures, and the host name of the last successful destination.
* **Anomalies**: Unusual activity discovered by [{{ml}} jobs](/solutions/security/advanced-entity-analytics/anomaly-detection.md) that contain user data.
* **User risk**: The latest recorded user risk score for each user, and its user risk classification. In {{stack}}, this feature requires a [Platinum subscription](https://www.elastic.co/pricing) or higher. In serverless, this feature requires the Security Analytics Complete [project feature tier](/deploy-manage/deploy/elastic-cloud/project-settings.md). Click **Enable** on the **User risk** tab to get started. To learn more, refer to our [entity risk scoring documentation](/solutions/security/advanced-entity-analytics/entity-risk-scoring.md).

The Events table includes inline actions and several customization options. To learn more about what you can do with the data in these tables, refer to [*Manage detection alerts*](/solutions/security/detect-and-alert/manage-detection-alerts.md).


## User details page [user-details-page]

A user’s details page displays all relevant information for the selected user. To view a user’s details page, click its **User name** link from the **All users** table.

The user details page includes the following sections:

* **Asset Criticality**: This section displays the user’s current [asset criticality level](/solutions/security/advanced-entity-analytics/asset-criticality.md).
* **Summary**: Details such as the user ID, when the user was first and last seen, the associated IP address(es), and operating system. If the user risk score feature is enabled, this section also displays user risk score data.
* **Alert metrics**: The total number of alerts by severity, rule, and status (`Open`, `Acknowledged`, or `Closed`).
* **Data tables**: The same data tables as on the main Users page, except with values for the selected user instead of for all users.

:::{image} /solutions/images/security-user-details-pg.png
:alt: User details page
:screenshot:
:::

