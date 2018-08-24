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



#### Advanced features
 - Faceting 
 - Highlighting
 - Spell checking 
 - More like this 

