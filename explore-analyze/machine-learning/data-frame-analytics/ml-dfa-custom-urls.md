---
mapped_pages:
  - https://www.elastic.co/guide/en/machine-learning/current/ml-dfa-custom-urls.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: machine-learning
---

# Adding custom URLs to data frame analytics jobs [ml-dfa-custom-urls]

You can optionally attach one or more custom URLs to your {{dfanalytics-jobs}}. These links can direct you to dashboards, the **Discover** app, or external websites. For example, you can define a custom URL that provides a way for users to drill down to the source data from a {{regression}} job. You can create a custom URL during job creation under **Additional settings** in the **Job details** step. Alternatively, you can edit or add new custom URLs in the job list by clicking **Edit** in the **Actions** menu.

:::{image} /explore-analyze/images/machine-learning-ml-dfa-custom-url.png
:alt: Creating a custom URL during job creation
:screenshot:
:::

When you create or edit an {{dfanalytics-job}} in {{kib}}, it simplifies the creation of the custom URLs for {{kib}} dashboards and the **Discover** app and it enables you to test your URLs. For example:

:::{image} /explore-analyze/images/machine-learning-ml-dfa-custom-url-edit.png
:alt: Add a custom URL in {{kib}}
:screenshot:
:::

For each custom URL, you must supply a label. You can also optionally supply a time range. When you link to **Discover** or a {{kib}} dashboard, you’ll have additional options for specifying the pertinent {{data-source}} or dashboard name and query entities.

## String substitution in custom URLs [ml-dfa-url-strings]

You can use dollar sign ($) delimited tokens in a custom URL. These tokens are substituted for the values of the corresponding fields in the result index. For example, a custom URL might resolve to `discover#/?_g=(time:(from:'$earliest$',mode:absolute,to:'$latest$'))&_a=(filters:!(),index:'4b899bcb-fb10-4094-ae70-207d43183ffc',query:(language:kuery,query:'Carrier:"$Carrier$"'))`. In this case, the pertinent value of the `Carrier` field is passed to the target page when you click the link.

::::{tip}
When you create your custom URL in {{kib}}, the **Query entities** option is shown only when there are appropriate fields in the index.
::::

The `$earliest$` and `$latest$` tokens pass the beginning and end of the time span of the data to the target page. The tokens are substituted with date-time strings in ISO-8601 format. For example, the following API updates a job to add a custom URL that uses `$earliest$` and `$latest$` tokens:

```console
POST _ml/data_frame/analytics/flight-delay-regression/_update
{
  "_meta": {
    "custom_urls": [
      {
        "url_name": "flight-delay-regression-results",
        "url_value": "dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d?_g=(filters:!(),time:('$earliest$',mode:absolute,to:'$latest$'))&_a=(filters:!(),query:(language:kuery,query:''))",
        "time_range": "1h",
      }
    ]
  }
}
```

When you click this custom URL, it opens up the **Discover** page and displays source data for the period one hour before and after the date of the default global settings.

::::{tip}

* The custom URL links use pop-ups. You must configure your web browser so that it does not block pop-up windows or create an exception for your {{kib}} URL.
* When creating a link to a {{kib}} dashboard, the URLs for dashboards can be very long. Be careful of typos, end of line characters, and URL encoding. Also ensure you use the appropriate index ID for the target {{kib}} {{data-source}}.
* The dates substituted for `$earliest$` and `$latest$` tokens are in ISO-8601 format and the target system must understand this format.
* If the job performs an analysis against nested JSON fields, the tokens for string substitution can refer to these fields using dot notation. For example, `$cpu.total$`.
* {{es}} source data mappings might make it difficult for the query string to work. Test the custom URL before saving the job configuration to check that it works as expected, particularly when using string substitution.

::::
