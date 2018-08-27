## Using solr at scale


#### Why

If searches are taking too long or the index is approaching the physical limitations of its machine, you should consider distributing the index across two or more Solr servers.

To distribute an index, you divide the index into partitions called shards, each of which runs on a separate machine. Solr then partitions searches into sub-searches, which run on the individual shards, reporting results collectively.

The architectural details underlying index sharding are invisible to end users, who simply experience faster performance on queries against very large indexes.

#### How

Apache Solr includes the ability to set up a cluster of Solr servers that combines fault tolerance and high availability. Called SolrCloud, these capabilities provide distributed indexing and search capabilities, supporting the following features:

- Central configuration for the entire cluster
- Automatic load balancing and fail-over for queries
- ZooKeeper integration for cluster coordination and configuration.

SolrCloud is flexible distributed search and indexing, without a master node to allocate nodes, shards and replicas. Instead, Solr uses ZooKeeper to manage these locations, depending on configuration files and schemas. Queries and updates can be sent to any server. Solr will use the information in the ZooKeeper database to figure out which servers need to handle the request.

#### Demo

- Use the [node1](../example/solr-cloud-demo/node1/solr.xml) and [node2](../example/solr-cloud-demo/node2/solr.xml) directory to start up 2 solr instances.
- Use this command to use those nodes and start solr `bin/solr start -s /path/to/node1 -p 9001` and similarly for `node2` with port as `9002`.
- If you use `curl http://localhost:9001/solr/core1/select?q=*:*&wt=xml&indent=true`, you can see one document with name `Dell Widescreen ...` and if you do same for 2nd node i.e. at port 9002, you can see the same document.
- If you specify shards while searching, like this `curl http://localhost:9001/solr/core1/select?q=*:*&indent=true&shards=localhost:9001/solr/core1,localhost:9002/solr/core1&fl=id,name`, you can see the one and only same document . Since those are 2 shards, if considered bothe documents as same and shown only one.

#### Limitation

- If Solr discovers duplicate document IDs, Solr selects the first document and discards subsequent ones.
- The index for distributed searching may become momentarily out of sync if a commit happens between the first and second phase of the distributed search. This might cause a situation where a document that once matched a query and was subsequently changed may no longer match the query but will still be retrieved. This situation is expected to be quite rare, however, and is only possible for a single query request.
- The number of shards is limited by number of characters allowed for GET method’s URI.
- commit commands and deleteByQuery commands are sent to every server in shards.


#### Reference

- https://lucene.apache.org/solr/guide/6_6/distributed-search-with-index-sharding.html