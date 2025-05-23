---
navigation_title: Use Elastic prebuilt rules
mapped_pages:
  - https://www.elastic.co/guide/en/security/current/prebuilt-rules-management.html
  - https://www.elastic.co/guide/en/serverless/current/security-prebuilt-rules-management.html
applies_to:
  stack: all
  serverless:
    security: all
products:
  - id: security
  - id: cloud-serverless
---

# Install and manage Elastic prebuilt rules [security-prebuilt-rules-management]

Follow these guidelines to start using the {{security-app}}'s [prebuilt rules](detection-rules://index.md), keep them updated, and make sure they have the data needed to run successfully.

* [Install and enable Elastic prebuilt rules](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#load-prebuilt-rules)
* [Prebuilt rule tags](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#prebuilt-rule-tags)
* [Select and duplicate all prebuilt rules](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#select-all-prebuilt-rules)
* [Update Elastic prebuilt rules](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#update-prebuilt-rules)
* [Confirm rule prerequisites](/solutions/security/detect-and-alert/manage-detection-rules.md#rule-prerequisites)

::::{note}
* Most prebuilt rules don’t start running by default. You can use the **Install and enable** option to start running rules as you install them, or first install the rules, then enable them manually. After installation, only a few prebuilt rules will be enabled by default, such as the Endpoint Security rule.

* Without an [Enterprise subscription](https://www.elastic.co/pricing) subscription on {{stack}} or a [Complete project tier](../../../deploy-manage/deploy/elastic-cloud/project-settings.md) subscription on {{serverless-short}}, you can't modify most settings on Elastic prebuilt rules. You must first duplicate them, then make your changes to the duplicated rules. Refer to [Select and duplicate all prebuilt rules](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#select-all-prebuilt-rules) to learn more.
* On {{stack}}, automatic updates of Elastic prebuilt rules are supported for the current {{elastic-sec}} version and the latest three previous minor releases. For example, if you’re on {{elastic-sec}} 9.0, you’ll be able to use the Rules UI to update your prebuilt rules until {{elastic-sec}} 9.4 is released. After that point, you can still manually download and install updated prebuilt rules, but you must upgrade to the latest {{elastic-sec}} version to receive automatic updates.

::::



## Install and enable Elastic prebuilt rules [load-prebuilt-rules]

1. Find **Detection rules (SIEM)** in the navigation menu or by using the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md), then go to the Rules table.

    The badge next to **Add Elastic rules** shows the number of prebuilt rules available for installation.

    :::{image} /solutions/images/security-prebuilt-rules-add-badge.png
    :alt: The Add Elastic Rules page
    :screenshot:
    :::

2. Click **Add Elastic rules**.

    ::::{tip}
    To examine the details of a rule before you install it, select the rule name. This opens the rule details flyout.
    ::::

3. Do one of the following:

    * Install all available rules: Click **Install all** at the top of the page. (This doesn’t enable the rules; you still need to do that manually.)
    * Install a single rule: In the rules table, either click **Install** to install a rule without enabling it, or click ![Vertical boxes button](/solutions/images/security-boxesVertical.svg "") → **Install and enable** to start running the rule once it’s installed.
    * Install multiple rules: Select the rules, and then at the top of the page either click **Install *x* selected rule(s)** to install without enabling the rules, or click ![Vertical boxes button](/solutions/images/serverless-boxesVertical.svg "") → **Install and enable** to install and start running the rules.

    ::::{tip}
    Use the search bar and **Tags** filter to find the rules you want to install. For example, filter by `OS: Windows` if your environment only includes Windows endpoints. For more on tag categories, refer to [Prebuilt rule tags](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#prebuilt-rule-tags).
    ::::


    :::{image} /solutions/images/security-prebuilt-rules-add.png
    :alt: The Add Elastic Rules page
    :screenshot:
    :::

4. For any rules you haven’t already enabled, go back to the **Rules** page, search or filter for the rules you want to run, and do either of the following:

    * Enable a single rule: Turn on the rule’s **Enabled** switch.
    * Enable multiple rules: Select the rules, then click **Bulk actions** → **Enable**.


Once you enable a rule, it starts running on its configured schedule. To confirm that it’s running successfully, check its **Last response** status in the rules table, or open the rule’s details page and check the [**Execution results**](/solutions/security/detect-and-alert/monitor-rule-executions.md#rule-execution-logs) tab.

If you have an [Enterprise subscription](https://www.elastic.co/pricing) subscription on {{stack}} or a [Complete project tier](../../../deploy-manage/deploy/elastic-cloud/project-settings.md) subscription on {{serverless-short}}, you can also [edit the prebuilt rules](/solutions/security/detect-and-alert/manage-detection-rules.md#edit-rules-settings) that you've installed.

## Prebuilt rule tags [prebuilt-rule-tags]

Each prebuilt rule includes several tags identifying the rule’s purpose, detection method, associated resources, and other information to help categorize your rules. These tags are category-value pairs; for example, `OS: Windows` indicates rules designed for Windows endpoints. Categories include:

* `Data Source`: The application, cloud provider, data shipper, or Elastic integration providing data for the rule.
* `Domain`: A general category of data source types (such as cloud, endpoint, or network).
* `OS`: The host operating system, which could be considered another data source type.
* `Resources`: Additional rule resources such as investigation guides.
* `Rule Type`: Identifies if the rule depends on specialized resources (such as machine learning jobs or threat intelligence indicators), or if it’s a higher-order rule built from other rules' alerts.
* `Tactic`: MITRE ATT&CK tactics that the rule addresses.
* `Threat`: Specific threats the rule detects (such as Cobalt Strike or BPFDoor).
* `Use Case`: The type of activity the rule detects and its purpose. Use cases include:

    * `Active Directory Monitoring`: Detects changes related to Active Directory.
    * `Asset Visibility`: Detects changes to specified asset types.
    * `Configuration Audit`: Detects undesirable configuration changes.
    * `Guided Onboarding`: Example rule, used for {{elastic-sec}}'s guided onboarding tour.
    * `Identity and Access Audit`: Detects activity related to identity and access management (IAM).
    * `Log Auditing`: Detects activity on log configurations or storage.
    * `Network Security Monitoring`: Detects network security configuration activity.
    * `Threat Detection`: Detects threats.
    * `Vulnerability`: Detects exploitation of specific vulnerabilities.



## Select and duplicate prebuilt rules [select-all-prebuilt-rules]

Without an [Enterprise subscription](https://www.elastic.co/pricing) subscription on {{stack}} or a [Complete project tier](../../../deploy-manage/deploy/elastic-cloud/project-settings.md) subscription on {{serverless-short}}, you can't modify most settings on Elastic prebuilt rules. You can only edit [rule actions](/solutions/security/detect-and-alert/create-detection-rule.md#rule-schedule) and [add exceptions](/solutions/security/detect-and-alert/add-manage-exceptions.md). If you want to modify other settings on a prebuilt rule, you must first duplicate it, then make your changes to the duplicated rule. Note that your customized rule is entirely separate from the original prebuilt rule, and will not get updates from Elastic if the prebuilt rule is updated.

1. Find **Detection rules (SIEM)** in the navigation menu or by using the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
2. In the **Rules** table, select the **Elastic rules** filter.
3. Select one or more rules, or click **Select all *x* rules** above the Rules table.
4. Click **Bulk actions** → **Duplicate**.
5. (Optional) Select whether to duplicate the rules' exceptions, then click **Duplicate**.

You can then modify the duplicated rules and, if required, delete the prebuilt ones.


## Update Elastic prebuilt rules [update-prebuilt-rules]

::::{important}

The following steps are only applicable if you have a [Platinum](https://www.elastic.co/pricing) subscription  or lower on  {{stack}} or an [Essentials project tier](../../../deploy-manage/deploy/elastic-cloud/project-settings.md) subscription on {{serverless-short}}.

If you have an Enterprise subscription on {{stack}} or a Complete project tier subscription on {{serverless-short}}, follow the guidelines in [Update modified and unmodified Elastic prebuilt rules](/solutions/security/detect-and-alert/prebuilt-rules-update-modified-unmodified.md) instead.
::::

Elastic regularly updates prebuilt rules to optimize their performance and ensure they detect the latest threats and techniques. When updated versions are available for your installed prebuilt rules, the **Rule Updates** tab appears on the **Rules** page, allowing you to update your installed rules with the latest versions.

1. Find **Detection rules (SIEM)** in the navigation menu or by using the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
2. In the **Rules** table, select the **Rule Updates** tab.

    ::::{note}
    The **Rule Updates** tab doesn’t appear if all your installed prebuilt rules are up to date.
    ::::


    :::{image} /solutions/images/security-prebuilt-rules-update.png
    :alt: The Rule Updates tab on the Rules page
    :screenshot:
    :::

3. (Optional) To examine the details of a rule’s latest version before you update it, select the rule name. This opens the rule details flyout.

    Select the **Elastic update overview** tab to view rule changes field by field, or the **JSON view** tab to view changes for the entire rule in JSON format. Both tabs display side-by-side comparisons of the **Current rule** (what you currently have installed) and the **Elastic update** version (what you can choose to install). Deleted characters are highlighted in red; added characters are highlighted in green.

    To accept the changes and install the updated version, select **Update rule**.

    :::{image} /solutions/images/security-prebuilt-rules-update-diff-basic.png
    :alt: Prebuilt rule comparison
    :screenshot:
    :::

4. Do one of the following to update prebuilt rules on the **Rules** page:

    * Update all available rules: Click **Update all**.
    * Update a single rule: Click **Update rule** for that rule.
    * Update multiple rules: Select the rules and click **Update *x* selected rule(s)**.

        ::::{tip}
        Use the search bar and **Tags** filter to find the rules you want to update. For example, filter by `OS: Windows` if your environment only includes Windows endpoints. For more on tag categories, refer to [Prebuilt rule tags](/solutions/security/detect-and-alert/install-manage-elastic-prebuilt-rules.md#prebuilt-rule-tags).
        ::::