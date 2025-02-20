---
mapped_urls:
  - https://www.elastic.co/guide/en/security/current/third-party-actions.html
  - https://www.elastic.co/guide/en/serverless/current/security-third-party-actions.html
---

# Third-party response actions

% What needs to be done: Align serverless/stateful

% Use migrated content from existing pages that map to this page:

% - [x] ./raw-migrated-files/security-docs/security/third-party-actions.md
% - [ ] ./raw-migrated-files/docs-content/serverless/security-third-party-actions.md

% Internal links rely on the following IDs being on this page (e.g. as a heading ID, paragraph ID, etc):

$$$crowdstrike-response-actions$$$

$$$security-third-party-actions-supported-systems-and-response-actions$$$

You can perform response actions on hosts enrolled in other third-party endpoint protection systems, such as CrowdStrike or SentinelOne. For example, you can direct the other system to isolate a suspicious endpoint from your network, without leaving the {{elastic-sec}} UI.

::::{admonition} Requirements
* Third-party response actions require an [Enterprise subscription](https://www.elastic.co/pricing).
* Each response action type has its own user role privilege requirements. Find an action’s role requirements at [Endpoint response actions](/solutions/security/endpoint-response-actions.md).
* Additional [configuration](/solutions/security/endpoint-response-actions/configure-third-party-response-actions.md) is required to connect {{elastic-sec}} with a third-party system.

::::



## CrowdStrike response actions [crowdstrike-response-actions]

These response actions are supported for CrowdStrike-enrolled hosts:

* **Isolate and release a host** using any of these methods:

    * From a detection alert
    * From the response console

    Refer to the instructions on [isolating](/solutions/security/endpoint-response-actions/isolate-host.md#isolate-a-host) and [releasing](/solutions/security/endpoint-response-actions/isolate-host.md#release-a-host) hosts for more details.

* **Run a script on a host** with the [`runscript` response action](/solutions/security/endpoint-response-actions.md#runscript).
* **View past response action activity** in the [response actions history](/solutions/security/endpoint-response-actions/response-actions-history.md) log.


## Microsoft Defender for Endpoint response actions [defender-response-actions]

These response actions are supported for Microsoft Defender for Endpoint–enrolled hosts:

* **Isolate and release a host** using any of these methods:

    * From a detection alert
    * From the response console

    Refer to the instructions on [isolating](/solutions/security/endpoint-response-actions/isolate-host.md#isolate-a-host) and [releasing](/solutions/security/endpoint-response-actions/isolate-host.md#release-a-host) hosts for more details.



## SentinelOne response actions [sentinelone-response-actions]

These response actions are supported for SentinelOne-enrolled hosts:

* **Isolate and release a host** using any of these methods:

    * From a detection alert
    * From the response console

    Refer to the instructions on [isolating](/solutions/security/endpoint-response-actions/isolate-host.md#isolate-a-host) and [releasing](/solutions/security/endpoint-response-actions/isolate-host.md#release-a-host) hosts for more details.

* **Retrieve a file from a host** with the [`get-file` response action](/solutions/security/endpoint-response-actions.md#get-file).

    ::::{note}
    For SentinelOne-enrolled hosts, you must use the password `Elastic@123` to open the retrieved file.
    ::::

* **Get a list of processes running on a host** with the [`processes` response action](/solutions/security/endpoint-response-actions.md#processes). For SentinelOne-enrolled hosts, this command returns a link for downloading the process list in a file.
* **Terminate a process running on a host** with the [`kill-process` response action](/solutions/security/endpoint-response-actions.md#kill-process).

    ::::{note}
    For SentinelOne-enrolled hosts, you must use the parameter `--processName` to identify the process to terminate. `--pid` and `--entityId` are not supported.

    Example: `kill-process --processName cat --comment "Terminate suspicious process"`

    ::::

* **View past response action activity** in the [response actions history](/solutions/security/endpoint-response-actions/response-actions-history.md) log.
