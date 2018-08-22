## Building blocks of Solr

#### Admin interface

Once solr is running up, you can access the URL http://hostname:8983/solr/ which will show the main dashboard, that is divided into two parts.

A left-side of the screen is a menu under the Solr logo that provides the navigation through the screens of the UI. The first set of links are for system-level information and configuration and provide access to Logging, Collection/Core Administration, and Java Properties, among other things. At the end of this information is at least one pulldown listing Solr cores configured for this instance. On SolrCloud nodes, an additional pulldown list shows all collections in this cluster. Clicking on a collection or core name shows secondary menus of information for the specified collection or core, such as a Schema Browser, Config Files, Plugins & Statistics, and an ability to perform Queries on indexed data.

#### Request handler

It processes requests coming to Solr. These might be query requests or index update requests. You will likely need several of these defined, depending on how you want Solr to handle the various requests you will make.

It is present after the <query> section of solrconfig.xml.

Every request handler is defined with a name and a class. The name of the request handler is referenced with the request to Solr, typically as a path. For example, if Solr is installed at ` http://localhost:8983/solr/ `and you have a collection named “gettingstarted”, you can make a request using URLs like this - `http://localhost:8983/solr/gettingstarted/select?q=solr`. This query will be processed by the request handler with the name /select. We’ve only used the "q" parameter here, which includes our query term, a simple keyword of "solr". If the request handler has more parameters defined, those will be used with any query we send to this request handler unless they are over-ridden by the client (or user) in the query itself.

There are several kind of request handler such as "SearchHandlers", "UpdateRequestHandlers" ,"ShardHandlers" etc...

#### Index management

This is happens via index writer and index searcher. Job of index writer is to write index, merge index, optimize whenever required. Index searcher responsible for handling query input and finding result.

#### Search Component

It is a feature of search, such as highlighting or faceting. The search component is defined in solrconfig.xml separate from the request handlers, and then registered with a request handler as needed. It defines the logic that is used by the SearchHandler to perform queries for users.

#### Response writer

It generates the formatted response of a search. Solr supports a variety of Response Writers to ensure that query responses can be parsed by the appropriate language or application. The wt parameter selects the Response Writer to be used. There are severl type of writer such as JSON, XML, CSV, xlsx etc..

#### Data Import Handler

Many search applications store the content to be indexed in a structured data store, such as a relational database. The Data Import Handler (DIH) provides a mechanism for importing content from a data store and indexing it. DIH can index content from HTTP based data sources such as RSS and ATOM feeds, e-mail repositories, and structured XML where an XPath processor is used to generate fields.

