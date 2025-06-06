---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/consistent-scoring.html
applies_to:
  stack:
  serverless:
products:
  - id: elasticsearch
---

# Getting consistent scoring [consistent-scoring]

The fact that Elasticsearch operates with shards and replicas adds challenges when it comes to having good scoring.


## Scores are not reproducible [_scores_are_not_reproducible] 

Say the same user runs the same request twice in a row and documents do not come back in the same order both times, this is a pretty bad experience isn’t it? Unfortunately this is something that can happen if you have replicas (`index.number_of_replicas` is greater than 0). The reason is that Elasticsearch selects the shards that the query should go to in a round-robin fashion, so it is quite likely if you run the same query twice in a row that it will go to different copies of the same shard.

Now why is it a problem? Index statistics are an important part of the score. And these index statistics may be different across copies of the same shard due to deleted documents. As you may know when documents are deleted or updated, the old document is not immediately removed from the index, it is just marked as deleted and it will only be removed from disk on the next time that the segment this old document belongs to is merged. However for practical reasons, those deleted documents are taken into account for index statistics. So imagine that the primary shard just finished a large merge that removed lots of deleted documents, then it might have index statistics that are sufficiently different from the replica (which still have plenty of deleted documents) so that scores are different too.

The recommended way to work around this issue is to use a string that identifies the user that is logged in (a user id or session id for instance) as a [preference](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-search). This ensures that all queries of a given user are always going to hit the same shards, so scores remain more consistent across queries.

This work around has another benefit: when two documents have the same score, they will be sorted by their internal Lucene doc id (which is unrelated to the `_id`) by default. However these doc ids could be different across copies of the same shard. So by always hitting the same shard, we would get more consistent ordering of documents that have the same scores.


## Relevancy looks wrong [_relevancy_looks_wrong] 

If you notice that two documents with the same content get different scores or that an exact match is not ranked first, then the issue might be related to sharding. By default, Elasticsearch makes each shard responsible for producing its own scores. However since index statistics are an important contributor to the scores, this only works well if shards have similar index statistics. The assumption is that since documents are routed evenly to shards by default, then index statistics should be very similar and scoring would work as expected. However in the event that you either:

* use routing at index time,
* query multiple *indices*,
* or have too little data in your index

then there are good chances that all shards that are involved in the search request do not have similar index statistics and relevancy could be bad.

If you have a small dataset, the easiest way to work around this issue is to index everything into an index that has a single shard (`index.number_of_shards: 1`), which is the default. Then index statistics will be the same for all documents and scores will be consistent.

Otherwise the recommended way to work around this issue is to use the [`dfs_query_then_fetch`](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-search#operation-search-search_type) search type. This will make Elasticsearch perform an initial round trip to all involved shards, asking them for their index statistics relatively to the query, then the coordinating node will merge those statistics and send the merged statistics alongside the request when asking shards to perform the `query` phase, so that shards can use these global statistics rather than their own statistics in order to do the scoring.

In most cases, this additional round trip should be very cheap. However in the event that your query contains a very large number of fields/terms or fuzzy queries, beware that gathering statistics alone might not be cheap since all terms have to be looked up in the terms dictionaries in order to look up statistics.

