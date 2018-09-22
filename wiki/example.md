## Usage and example

This part is divided to following sections.

#### Getting started

- Solr depends on java. Here I will be using Solr 6.5, it depends on Java 8 and above. 
- Download solr from here - http://archive.apache.org/dist/lucene/solr/6.5.1/
- Since solr contains self contained jetty server, you can directly use it. There is option to host in separate server.
- Open terminal and go to solr directory. Then use './bin/solr start' to start solr. By default it runs on 8983 port. Now you can browse to `"http://localhost:8983/solr"` to use solr.
- To stop solr use `./bin/solr stop`.

#### Solr Core 

- A Solr Core is a running instance of a Lucene index that contains all the Solr configuration files required to use it. We need to create a Solr Core to perform operations like indexing and analyzing.
- A Solr application may contain one or multiple cores. If necessary, two cores in a Solr application can communicate with each other.
- To create a core in solr use this command - './bin/solr create -c solr_demo'. Here solr_demo is the name of core.

#### Adding data to solr core 

- There are several ways to add documents to solr. As earlier explained, in solr each record is known as document and each attribute of record knows as solr field. Following ways to add document to solr.
	 - via post utility . It is present in /bin/post. E.g. command to index CSV file is "./bin/post -c {core_name} {path_to_file}"
	 - via curl
	 - via uppdate handler in solr admin. Navigate to solr admin, select the core, then click on documents. Here you can post JSON/XML/CSV etc. kind of data.
	 - via data importhandler. You can specify data import handler, it will connect to DB to fetch data. The source can be any thing, like file system, RSS etc..
	 - There are java libraries available ( e.g. solrj) , where you can programmatically push data to solr.
- For example, I will be using sample data which consists of following attributes. It is csv file - [all-course.csv](example/Sample-Data-File/all-course.csv)
	- course id, course title, durationinsecond, releasedate, description, assessment, iscourseretired, course-author, tag.


#### Prerequisite for Example

- To start will the example you can have to first create a solr using `./bin/solr create -c courses`
- Once you create solr core, you can see that there is a directory created under `solr_root/server/solr/courses`.
- If you go to conf directory, there are several files out of which 2 are most important one - managed_schema, solrconfig.xml.
- managed_schema contains schema definition of the core. It defines what are fields , its type, attributes. And the other one ,specifies most parameters affecting Solr itself.
- Open managed_schema file and put following content after `<schema name="example-data-driven-schema" version="1.6">`.

```
<field name="courseid" type="string" indexed="true" stored="true" required="true" multiValued="false" />     
<field name="coursetitle" type="string" indexed="true" stored="true" multiValued="false"/>  
<field name="durationinseconds" type="int" indexed="true" stored="true" />  
<field name="releasedate" type="date" indexed="true" stored="true"/>
<field name="description" type="text_general" indexed="true" stored="true"/>  
<field name="assessmentstatus" type="string" indexed="true" stored="true"/> 
<field name="iscourseretired" type="string" indexed="true" stored="true"/>  
<field name="tag" type="string" multiValued="true" indexed="true" stored="true"/> 
<field name="course-author" type="string" multiValued="true" indexed="true" stored="true" />
```

- Things to note. indexed and stored attribute.
	- `indexed=true` makes a field searchable (and sortable and facetable). For eg, if you have a field named test1 with indexed=true, then you can search it like q=test1:foo, where foo is the value you are searching for. If indexed=false for field test1 then that query will return no results, even if you have a document in Solr with test1's value being foo.
	- `stored=true` means you can retrieve the field when you search. If you want to explicitly retrieve the value of a field in your query, you will use the fl param in your query like fl=test1 (Default is fl=* meaning retrieve all stored fields). Only if stored=true for test1, the value will be returned. Else it will not be returned.

- Then you have to post the CSV file using - `./bin/post -c courses ~/Documents/github/solr-tutorial/data/all_courses.csv`.

#### Querying

- We will be using solr admin for querying. In real world, application consumes solr data via some API and show it inside application. 
- Navigate to solr admin panel. Select "courses" core. Then click on "query" on left side just below the core selection. Then just hit `Execute query`. There you can see the results. It will list out all datas present in CSV file.
- Lets say we can you search for `web-services`, if you put that to 2nd text box ( i.e. q), and hit the "Execute query" you can see that all result has `web-services` present somewhere in any of attribute.
- Solr assigns some core to document based on relevancy. It uses vector space model to for score calculation, you can use this [link](https://lucene.apache.org/core/2_9_4/scoring.html) to get more info about this.
- In admin pane, you can see there a lot of configurable things. We will go through them here.
- Now, we have started with `q`, which takes query parameter. And we have searched for `web-services`.
- If we want to search for `web-services` only in description field, the value of `q` will look like this `description:"web-services"`. If you execute this query you can see that only results are show where description has `web-services`. With this you can see there are total `6` records present.
- Then we have `fq` which stands for filter query. If we want to filter the courses which has duration for 7000 to 8000 second, we can use it like `durationinseconds:[7000 TO 8000]`. With this you can see only 1 course present. Note: I still have `q=description:"web-services"`. This is mostly used along with `q`, this can utilize filter cache to give better perfomance.
- We have parameter for `sort`. Here you can specify by which parameter you want to sort. Say we want to sort all courses by `durationinseconds`, then you can use `durationinseconds asc` in that field to sort it by ascending.
- start, rows:. By default, sorl result shows 10 result. This parameter enables pagination. You can specify the page as `start` and rows as `number of records` to fetch.
- fl: This enables you select whatever field you need to show in result. Say we want to show only coursetitle, description you can put the value as `coursetitle description` and on executing query you can see only those attribute values. If you want see the score for the result, you can include score as `coursetitle description score` and see score for individual documents.
- Raw query: It enables to pass advanced parameter which is not present in admin console.
- wt: This is writer option. Currently we are using JSON, you can change that to XML or required type of data.
- indent: Tell whether to indent.
- debugQuery: Lets you anayze the result, to see which part took more time. Helps you to identify the score for each document.

Rest option we will cover in advanced feature.


#### Advanced features

##### (e)dismax: 

The DisMax query parser is designed to process simple phrases (without complex syntax) entered by users and to search for individual terms across several fields using different weighting (boosts) based on the significance of each field. Additional options enable users to influence the score based on rules specific to each use case (independent of user input).
 	- qf: specifies the fields in the index on which to perform the query. If absent, defaults to.
 	- mm: Minimum "Should" Match: specifies a minimum number of clauses that must match in a query.
 	- pf: Phrase Fields: boosts the score of documents in cases where all of the terms in the q parameter appear in close proximity.
 	- ps: Phrase Slop: specifies the number of positions two terms can be apart in order to match the specified phrase.
 	- qs: Query Phrase Slop: specifies the number of positions two terms can be apart in order to match the specified phrase. Used specifically with the qf parameter.
 	- tie: Tie Breaker: specifies a float value (which should be something much less than 1) to use as tiebreaker in DisMax queries. Default: 0.0
 	- bq: Boost Query: specifies a factor by which a term or phrase should be "boosted" in importance when considering a match.
 	- bf: Boost Functions: specifies functions to be applied to boosts. (See for details about function queries.)
##### Highlighting

 	- if you want to highlight the search result, you can use this option. I have used it like this
 >>>
 		hl.fl = description
 		hl.simple.pre = <found>
 		hl.simple.post = </found>
 		
 	- This means, it will look for desciption field to highlight the result and if that matches, it will wrap with `found` tag around.
 	- Here is the sample section of the result I received.

	 	
>>> 
"highlighting":{
	    "29b71398-3f08-42e5-a3b8-a93409016191":{
	      "description":["Improve exposure, <found>build</found> <found>community</found>, and increase downloads. Learn how to market your app and manage"]},
	    "e2c46c4d-d97b-4a20-b03d-23586a5f45ac":{},
	    "c12c8088-b976-4064-8280-3180cedd2c1d":{},
	    "5ff30aa3-eb85-47ab-9857-11521951a17d":{
	      "description":["Learn how to debug <found>Web</found> applications using automated memory dumps, and how to detect and fix <found>web</found>"]},
	    "6f868d20-e0c0-4134-83d8-f1e627ee7c31":{
	      "description":["Learn how to debug <found>Web</found> applications using automated memory dumps, and how to detect and fix <found>web</found>"]},
	    " ...
	   
##### Faceting: 
     It is the arrangement of search results into categories based on indexed terms.
	- Searchers are presented with the indexed terms, along with numerical counts of how many matching documents were found were each term. Faceting makes it easy for users to explore search results, narrowing in on exactly the results they are looking for.
	- In our example, say we want to have faceting for duration in second and we want to show it in 1000 increment. E.g. say we have 10 courses, out of which 3 are of 1000- 2000 second, 4 are of  6000- 7000 second and rest are above 9000 second. Then search result will show following results.
	
>>>
	 1000  (0)
	 2000  (3)
	 3000  (0)
	 4000  (0)
	 5000  (0)
	 6000  (0)
	 7000  (4)
	 8000  (0)
	 9000  (0)
	 10000 (3)
 
	- Here the value of faceting query we needed - `facet.range=durationinseconds&f.durationinseconds.facet.range.start=2000&f.durationinseconds.facet.range.end=20000&f.durationinseconds.facet.range.gap=1000`. I will turn of facet and use above query in raw parameter as it can't be accomodated in facet query field itself.
	- Here is the result I will get.
>>>
    "facet_counts":{
    "facet_queries":{},
    "facet_fields":{},
    "facet_ranges":{
      "durationinseconds":{
        "counts":[
          "2000",6,
          "3000",30,
          "4000",23,
          "5000",32,
          "6000",36,
          "7000",32,
          "8000",50,
          "9000",44,
          "10000",45,
          "11000",41,
          "12000",35,
          "13000",34,
          "14000",25,
          "15000",33,
          "16000",27,
          "17000",17,
          "18000",25,
          "19000",19],
        "gap":1000,
        "start":2000,
        "end":20000}},
    "facet_intervals":{},
    
 ##### Synonyms

 - As the same suggests, you can specify synonym for any of words being used in query or indexed document. For demo purpose, open [synonyms.txt](../example/Solr-Cores/courses/conf/synonyms.txt) and add `businesstalk => BizTalk`.
 - Navigate to admin panel, click on core admin, select courses core, and click on reload. Every time core is changed , it needs to be refreshed like this.
 - Then navigate to admin panel and search for `businesstalk`, there you should be getting document for `BizTalk` as we specified a synonym in the text file.

 ##### Stopwords

 - Here the words specified in stopwords.txt will be ignored while indexing and searching. 
 - If you open config/manage_schema, you can notice following

 ```
 <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100" multiValued="true">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        -->
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>
 ```
 - That means, all text specified in that file will be supplied to `StopFilterFactory` and it will ignore case for those words while filtering.

 ##### Stemming

 - It enables you to reduce words to shorter base.
 - E.g. runs, running, ran all resolve to `run`.
 - It is available via `PorterStemFilterFactory`.

 ##### Lemmatization

 - It is the opposite of stemming.
 - Here you can expand single word to multiple form e.g. run -> runs, running, ran etc.

 ##### Others
 
 - There is facilty to search spatial data where you have to provide lat , long, and radius info, it will get you the data within the range.
 - There is spell check facilty which can suggest you the exact word in document.


##### References

- https://lucene.apache.org/solr/guide/6_6/getting-started.html
- https://lucene.apache.org/solr/guide/6_6/overview-of-searching-in-solr.html

