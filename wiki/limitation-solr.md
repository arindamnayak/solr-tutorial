## Solr Performance problems and alternative

#### Solr Performance problems

- Memory usage: A major driving factor for Solr performance is RAM. Solr requires sufficient memory for two separate things: One is the Java heap, the other is "free" memory for the OS disk cache.

Another potential source of problems is a very high query rate. Adding memory can sometimes let Solr handle a higher rate. If more query scalability is required, eventually that won't be enough, and you will need to add multiple replicas of your index on multiple machines (preferably separate physical hardware) to scale further. 

Certain configurations and conditions in Solr will require a lot of heap memory. The following list is incomplete, but in no particular order, these include:

    - A large index.
    - Frequent updates.
    - Super large documents.
    - Extensive use of faceting.
    - Using a lot of different sort parameters.
    - Very large Solr caches
    - A large RAMBufferSizeMB.
    - Use of Lucene's RAMDirectoryFactory.

 One possible rule of thumb: Look at the number of queries per second Solr is seeing. If the number of garbage collections per minute exceeds that value, your heap MIGHT be too small. It also might be perfectly fine ... well-tuned garbage collection might do a large number of very quick collections on a frequent basis.

 Additional rule of thumb: More heap is usually better, but if you make it too big, the amount of time spent doing garbage collection can become extreme. That problem will be discussed below. Also, as discussed above, dropping the size of the OS disk cache can create more problems.

 Here is list showing how to reduce heap requirements, based on the list above for things that require a lot of heap:

    - Take a large index and make it distributed - break your index into multiple shards.
    - Don't store all your fields, especially the really big ones.
    - You can also enable docValues on fields used for sorting/facets and reindex.
    - Reduce the number of different sort parameters. Just like for facets, docValues can have a positive impact on both performance and memory usage for sorting.
    - Reduce the size of your Solr caches.
    - Reduce RAMBufferSizeMB. The default in recent Solr versions is 100.
    - Don't use RAMDirectoryFactory - instead, use the default and install enough system RAM so the OS can cache your entire index.


- Solr Cloud limitation: Regardless of the number of nodes or available resources, SolrCloud begins to have stability problems when the number of collections reaches the low hundreds. With thousands of collections, any little problem or change to the cluster can cause a stability death spiral that may not recover for tens of minutes. Try to keep the number of collections as low as possible. These problems are due to how SolrCloud updates cluster state in ZooKeeper in response to cluster changes.

- High request rate: If your request rate is high, that will impact performance, often very dramatically. If your request rate is above about 30 per second, it's probably time to think about scaling your install by adding additional copies of your index and additional servers. Handling thousands of requests per second will require a LOT of hardware. Handling a high query rate usually involves setting up multiple copies of your index on multiple servers and implementing some kind of load balancing. SolrCloud can automate a lot of this, and it does load balancing internally, but you might still need an external load balancer.

- Other miscellaneous finding:

    - Lack of index time sorting.
    - Lack of multiple document type per schema
    - Lack of scripting language in boosting.

#### References

- https://wiki.apache.org/solr/SolrPerformanceProblems
- https://dzone.com/articles/apache-solr-memory-tuning-for-production

#### Alternative

- Elasticsearch
    
    - About: Elasticsearch is an open source distributed, RESTful search and analytics engine capable of solving a growing number of use cases.
    - Near real time indexing, it is able to index rapidly changing data almost instantly .
    - It is a distributed system by nature and can easily scale horizontally providing the ability to extend resources and balance the loading between the nodes in a cluster. 
    - Elastics earch records any changes made in transactions logs on multiple nodes in the cluster to minimize the chance of data loss.
    - Elastic search is API driven, actions can be performed using a simple Restful API.
    - Multi tenancy: Often, you have multiple customers or users with separate collections of documents, and a user should never be able to search documents that do not belong to him. This often leads to a design where every user has his own index. More often this leads to have too many indexes. One larger Elastic search index is actually be better.
    - Elasticsearch as service : https://www.elastic.co/cloud/elasticsearch-service/pricing

- Sphinx
	
   - About: Sphinx is a full-text search engine (generally standalone) which provides fast, relevant, efficient full-text search functionality to third-party applications. It was especially created to facilitate searches on SQL databases and integrates very well with scripting languages; such as PHP, Python, Perl, Ruby, and Java.
   - indexing speed of up to 10-15 MB/sec per core and HDD.
   - Searching speed of over 500 queries/sec against 1,000,000 document/1.2 GB collection using a 2-core desktop system with 2 GB of RAM.
   - Supports MySQL (MyISAM and InnoDB tables are both supported) and PostgreSQL natively
   - Sphinx has better scalability. It can be scaled vertically (utilizing many CPUs, many HDDs) or horizontally (utilizing many servers), and this comes out of the box with Sphinx.
   - The search service daemon (searchd) is pretty low on memory usage.
   - It is on GPV v2 license, need commercial license for such products.

#### References

   - https://www.elastic.co/
   - http://sphinxsearch.com/about/sphinx/
