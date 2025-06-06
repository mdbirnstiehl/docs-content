---
navigation_title: Graph
mapped_pages:
  - https://www.elastic.co/guide/en/kibana/current/graph-troubleshooting.html
applies_to:
  stack: all
  serverless: all
products:
  - id: kibana
---



# Troubleshoot graph analytics features [graph-troubleshooting]



## Why are results missing? [_why_are_results_missing]

The default settings in Graph API requests are configured to tune out noisy results by using the following strategies:

* Only looking at samples of the most-relevant documents for a query
* Only considering terms that have a significant statistical correlation with the sample
* Only considering terms to be paired that have at least 3 documents asserting that connection

These are useful defaults for getting the "big picture" signals from noisy data, but they can miss details from individual documents. If you need to perform a detailed forensic analysis, you can adjust the following settings to ensure a graph exploration produces all of the relevant data:

* Increase the `sample_size` to a larger number of documents to analyze more data on each shard.
* Set the `use_significance` setting to `false` to retrieve terms regardless of any statistical correlation with the sample.
* Set the `min_doc_count` for your vertices to 1 to ensure only one document is required to assert a relationship.


## What can I do to improve performance? [_what_can_i_do_to_improve_performance]

With the default setting of `use_significance` set to `true`, the Graph API performs a background frequency check of the terms it discovers as part of exploration. Each unique term has to have its frequency looked up in the index, which costs at least one disk seek. Disk seeks are expensive. If you don’t need to perform this noise-filtering, setting `use_significance` to `false` eliminates all of these expensive checks (at the expense of not performing any quality-filtering on the terms).

If your data is noisy and you need to filter based on significance, you can reduce the number of frequency checks by:

* Reducing the `sample_size`. Considering fewer documents can actually be better when the quality of matches is quite variable.
* Avoiding noisy documents that have a large number of terms. You can do this by either allowing ranking to naturally favor shorter documents in the top-results sample (see [enabling norms](elasticsearch://reference/elasticsearch/mapping-reference/norms.md)) or by explicitly excluding large documents with your seed and guiding queries.
* Increasing the frequency threshold. Many many terms occur very infrequently so even increasing the frequency threshold by one can massively reduce the number of candidate terms whose background frequencies are checked.

Keep in mind that all of these options reduce the scope of information analyzed and can increase the potential to miss what could be interesting details. However, the information that’s lost tends to be associated with lower-quality documents with lower-frequency terms, which can be an acceptable trade-off.


## Limited support for multiple indices [_limited_support_for_multiple_indices]

The graph API can explore multiple indices, types, or aliases in a single API request, but the assumption is that each "hop" it performs is querying the same set of indices. Currently, it is not possible to take a term found in a field from one index and use that value to explore connections in *a different field* held in another type or index.

A good example of where this might be useful is if an IP address is found in the `remote_host` field of an index called "weblogs20160101", you might want to follow that up by looking for the same address in the `ip_address` field of an index called "knownthreats".

Supporting this behavior would require extra mappings to indicate that the weblogs' `remote_host` field contained values that had currency and meaning in the `ip_address` field of the threats index.

Since we do not currently support this translation, you would have to perform multiple calls to take the values from the weblogs index response and build them into a separate request to the threats index.

