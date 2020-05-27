#### 1 Details:
--------

> Default output behavior:

> By default, Solr outputs its log to console by stderr, so a way to redirect Solr's log is by running it with the command: java -jar start.jar 2> output.log. This way you won't have any log rolling. This is only an easy way to start working while developing, but it's not recommended for production environments.

* Logging Configuration:
------------------------

> Solr uses for logging the API SLF4J, which means that many different logging tools can be easily attached. By default, it comes with java.util.logging, but it can be changed to, for example log4J by changing the proper jars. See http://wiki.apache.org/solr/SolrLogging

* What's included in the logs?
------------------------------

> Of course, the logs will show different things depending on the configuration. In this doc, I'll only consider and mention some things that I consider can help  operations and are shown by default (by using INFO level)

* Bootstrap
-----------

> Immediately after starting Solr some lines will show information related to the configuration that can be very helpful, specially on the development stage of the projects for troubleshooting

* Solr Home
-----------

> Solr Home is the directory where the configuration of Solr's different cores are located. The solr.xml file will be located in the "solr home" directory, linking to where each core configuration is located. Typically, the cores are located inside the "solr home", under a directory with the same name as the core. One of the first things that the logger will show is the information about the "Solr Home" directory. For example:

```
--Snippet-
Jan 23, 2012 11:14:20 AM org.apache.solr.core.SolrResourceLoader locateSolrHome
INFO: JNDI not configured for solr (NoInitialContextEx)
Jan 23, 2012 11:14:20 AM org.apache.solr.core.SolrResourceLoader locateSolrHome
INFO: solr home defaulted to 'solr/' (could not find system property or JNDI)
Jan 23, 2012 11:14:20 AM org.apache.solr.core.SolrResourceLoader <init>
INFO: Solr home set to 'solr/'
...
INFO: looking for solr.xml: /home/user/trunk/solr/example/solr/solr.xml
--Snippet--
```

> After the solr.xml file is located, Solr will display information about the directory to use for each of the cores.

*** For information about how to set the Solr Home see http://wiki.apache.org/solr/SolrInstall

* Used Jars:
------------

> During bootstrap, Solr will display all the external jars added to the classpath. Those jars can be configured (added or removed) from the schema.xml file. If a tool that requires an external jar (for example, language identification) is failing due to ClassNotFound or related exception, a typical problem is that the jar is not being correctly added to the classpath. These lines look like:

```
--Snippet--
Jan 23, 2012 11:14:20 AM org.apache.solr.core.SolrResourceLoader replaceClassLoader
INFO: Adding 'file:/home/user/trunk/solr/example/solr/lib/apache-solr-uima-4.0-SNAPSHOT.jar' to classloader
Solr Configuration
After the used Jars, Solr will display information about the configuration to use for each core, including the field types, the default field, uniqueKey, default operator, and all solrconfig.xml information (request handlers, response writers, etc).

INFO: Using Lucene MatchVersion: LUCENE_40
Jan 23, 2012 11:15:58 AM org.apache.solr.core.SolrConfig <init>
INFO: Loaded SolrConfig: solrconfig.xml
Jan 23, 2012 11:15:58 AM org.apache.solr.schema.IndexSchema readSchema
INFO: Reading Solr Schema
Jan 23, 2012 11:15:59 AM org.apache.solr.schema.IndexSchema readSchema
INFO: Schema name=example
Jan 23, 2012 11:15:59 AM org.apache.solr.util.plugin.AbstractPluginLoader load
INFO: created string: org.apache.solr.schema.StrField
Jan 23, 2012 11:15:59 AM org.apache.solr.util.plugin.AbstractPluginLoader load
INFO: created boolean: org.apache.solr.schema.BoolField
...
Jan 23, 2012 11:15:59 AM org.apache.solr.schema.IndexSchema readSchema
INFO: default search field is text
Jan 23, 2012 11:15:59 AM org.apache.solr.schema.IndexSchema readSchema
INFO: query parser default operator is OR
Jan 23, 2012 11:15:59 AM org.apache.solr.schema.IndexSchema readSchema
INFO: unique key field: id
...
Jan 23, 2012 11:15:59 AM org.apache.solr.core.SolrCore initWriters
INFO: created json: solr.JSONResponseWriter
Jan 23, 2012 11:15:59 AM org.apache.solr.core.SolrCore initWriters
...
Jan 23, 2012 11:16:00 AM org.apache.solr.core.RequestHandlers initHandlersFromConfig
INFO: created search: solr.SearchHandler
...
2012-01-23 11:16:01.167:INFO::Opened /home/user/trunk/solr/example/logs/2012_01_23.request.log
2012-01-23 11:16:01.170:INFO::Started SocketConnector@0.0.0.0:8983
--Snippet--
```

> This is all interesting information that can be extracted from the logs, specially for troubleshooting during development. See how the last two lines show where the "request" logs are generated (this is not the exact same log this document is describing but one where Jetty outputs all the requests to the server) plus the port where Solr is running. 

* Search:
---------

> A typical search in Solr will be logged also with INFO level like the following example:

```
[...]
Jan 23, 2012 3:51:39 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/select params={facet=true&facet.field=cat&facet.field=popularity&fq=cat:software&q=name:solr} hits=2 status=0 QTime=4 
[...]
```

> The log line will show the path were the request was issued (which shows the target request handler) plus all the parameters used explicitly in the query. Parameters that are configured in the solrconfig file are not displayed in this line. After the parameters, the log line shows: the number of hits for the query, the status (0 meaning OK) and the time spent in this request in milliseconds.

* Distributed Search:
---------------------

> When using distributed search, the log output is slightly different than on single instances requests. The reason why the log output is different is because evidently the distributed search works in a different way than  single instance search. When a Solr instance receives a distributed request (lets say, the server A), it will distribute the request to all of the Solr instances specified in the "shards" parameter (for example, servers B and C). With the request, A will send all the original parameters except for the "shards" parameter, and will include a parameter called "isShard", so that B and C know that this request is actually part of a distributed search. B and C will respond to A with the ids and the score of each of the documents matching the query (in case of sorting by score, otherwise the sorting fields are included in the response, for this, the "fl" parameter will also change). With this information, A sort by score, and request B and/or C the documents with a given set of IDs (the ones that made the top N list). Once B and C respond with those documents, A will respond to the user query with the list of top rated documents for the search. 

> Continuing with the above example, the server A will receive a request like:

```
http://A:port/solr/select?q=name:solr&facet=true&facet.field=cat&facet.field=popularity&fq=cat:software&shards=B:port/solr,C:port/solr
```

> A will distribute the request to B and C and they will log something like this:

```
--Snippet--
Jan 23, 2012 3:57:17 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/select params={facet=true&f.popularity.facet.limit=160&wt=javabin&rows=10&version=2&f.cat.facet.limit=160&NOW=1327345037405&shard.url=[B or C]:port/solr&fl=id,score&start=0&q=name:solr&facet.field=cat&facet.field=popularity&isShard=true&fsv=true&fq=cat:software} hits=2 status=0 QTime=14 
--Snippet--
```

> This looks very similar to a  regular request, but it has some parameters that were added automatically. The above line also logs the number of hits for this shard, the status and the query time for this shard. Note that these are partial QTime and partial hits, just for this shard.
> A will get the response from B and C and will sort the results by score. Once it does it, it will request specific documents to them, that's why you'll see them log a line like:

```
--Snippet--
INFO: [] webapp=/solr path=/select params={facet=false&shard.url=[B or C]:port/solr&NOW=1327345037405&q=name:solr&ids=SOLR1001,SOLR1000&facet.field=cat&facet.field=popularity&isShard=true&wt=javabin&fq=cat:software&rows=10&version=2} status=0 QTime=1 
--Snippet--
```

> Finally, A will respond to the distributed search and log:

```
--Snippet--
Jan 23, 2012 3:57:17 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/select params={facet=true&shards=B:port/solr,C:port/solr&facet.field=cat&facet.field=popularity&fq=cat:software&q=name:solr} status=0 QTime=338 
--Snippet--
```

> This line will not show the total number of matches for the search, but it will show the status and the total query time for the request.

* Update:
---------

> Add/Update. This is a typical log section for "adds":

```
--Snippet--
INFO: {add=[0, 1, 2, 3, 4, 5, 6, 7, ... (100 adds)]} 0 281
Jan 23, 2012 1:10:03 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/update params={wt=javabin&version=2} status=0 QTime=281 
--Snippet--
```

> The first line shows the IDs of the documents that were added in the request. If the number of documents added in the request is big, not all of the IDs will be displayed and instead, just the first ones plus the total number will be shown. It also shows the status (the first number, being 0 = OK) and the request time in milliseconds.The last line of the snippet shows the path of the request plus the parameters used for the request and (again) the status and time spent in ms.

* Commit:
---------

> A commit operation will be displayed like:

```
--Snippet--
Jan 23, 2012 1:10:03 PM org.apache.solr.update.DirectUpdateHandler2 commit
INFO: start commit(optimize=false,waitSearcher=true,expungeDeletes=false,softCommit=false)
--Snippet--
```

> Shows when the commit is issued and the parameters used for it.

```
--Snippet--
Jan 23, 2012 1:10:03 PM org.apache.solr.core.SolrDeletionPolicy onCommit

INFO: SolrDeletionPolicy.onCommit: commits:num=2

 commit{dir=/home/user/Documents/solr/solr-trunk/trunk/solr/example/solr/data/index,segFN=segments_2,version=1327327721224,generation=2,filenames=[_0_0.frq, _0_0.prx, _0.fnm, _0_nrm.cfe, _0_0.tip, segments_2, _0.per, _0_0.tim, _0.fdx, _0_nrm.cfs, _0.fdt]

 commit{dir=/home/user/Documents/solr/solr-trunk/trunk/solr/example/solr/data/index,segFN=segments_3,version=1327327721226,generation=3,filenames=[_1_nrm.cfs, _0_0.frq, _0.fnm, _1_0.frq, _1.per, _1_0.tim, _0_0.tip, _0_0.tim, _0_nrm.cfs, _1.fnm, _1_0.prx, _1_nrm.cfe, _1.fdx, _0_0.prx, _1.fdt, _0_nrm.cfe, _0.fdx, _0.per, _1_0.tip, segments_3, _0.fdt]
--Snippet--
```

> This information belongs to the "Deletion Policy". The Deletion policy determines when a a commit point is going to be deleted (the default is to keep just one commit, this can be configured through solrconfig.xml). The two lines show the two existing commits (the old one and the one just created) with the files they contain. Currently Solr can't make much use of older commit points.

```
--Snippet--
INFO: newest commit = 1327327721226
This line shows the version of the last commit. This is going to be used  by the replication handler (along with the index generation).

Jan 23, 2012 1:10:03 PM org.apache.solr.search.SolrIndexSearcher <init>

INFO: Opening Searcher@6f3b625b main
The name of the Searcher instance that was created after the commit. Remember that a new SolrIndexSearcher is created after each commit. 
...
Jan 23, 2012 1:10:03 PM org.apache.solr.search.SolrIndexSearcher warm

INFO: autowarming Searcher@6f3b625b main{DirectoryReader(segments_3:1327327721226 _0(4.0):C4 _1(4.0):C100)} from Searcher@395fd251 main{DirectoryReader(segments_2:1327327721224 _0(4.0):C4)}

 fieldValueCache{lookups=0,hits=0,hitratio=0.00,inserts=0,evictions=0,size=0,warmupTime=0,cumulative_lookups=0,cumulative_hits=0,cumulative_hitratio=0.00,cumulative_inserts=0,cumulative_evictions=0}

Jan 23, 2012 1:10:03 PM org.apache.solr.search.SolrIndexSearcher warm

INFO: autowarming result for Searcher@6f3b625b main{DirectoryReader(segments_3:1327327721226 _0(4.0):C4 _1(4.0):C100)}

 fieldValueCache{lookups=0,hits=0,hitratio=0.00,inserts=0,evictions=0,size=0,warmupTime=0,cumulative_lookups=0,cumulative_hits=0,cumulative_hitratio=0.00,cumulative_inserts=0,cumulative_evictions=0}
--Snippet--
```

> In Solr, caches are owned by the Index Searcher. Each Index Searcher contains a set of caches, and when a new one is created (after a commit operation for example), it will automatically warm its caches by getting cache items from the existing Index Searcher (the one currently being used to respond queries). That's the information that is being logged in the above lines. The second line displays the new searcher (that is being warmed) and the old (current) searcher. The third line, information about the actual cache of the old searcher.
> The fifth and the sixth lines show information about the new index searcher and the new cache after warming it. 
> This is repeated for each of the different Solr caches (fieldValueCache, filterCache, documentCache and queryResultCache). 

```
--Snippet--
Jan 23, 2012 1:10:03 PM org.apache.solr.core.SolrCore registerSearcher

INFO: [] Registered new searcher Searcher@6f3b625b main{DirectoryReader(segments_3:1327327721226 _0(4.0):C4 _1(4.0):C100)}
This means that the new searcher has been registered. All the new search requests will be responded by the newly created index searcher (which means that the new added documents are going to be available)

Jan 23, 2012 1:10:03 PM org.apache.solr.search.SolrIndexSearcher close

INFO: Closing Searcher@395fd251 main

 fieldValueCache{lookups=0,hits=0,hitratio=0.00,inserts=0,evictions=0,size=0,warmupTime=0,cumulative_lookups=0,cumulative_hits=0,cumulative_hitratio=0.00,cumulative_inserts=0,cumulative_evictions=0}

 filterCache{lookups=0,hits=0,hitratio=0.00,inserts=0,evictions=0,size=0,warmupTime=0,cumulative_lookups=0,cumulative_hits=0,cumulative_hitratio=0.00,cumulative_inserts=0,cumulative_evictions=0}

 queryResultCache{lookups=0,hits=0,hitratio=0.00,inserts=0,evictions=0,size=0,warmupTime=0,cumulative_lookups=1,cumulative_hits=0,cumulative_hitratio=0.00,cumulative_inserts=1,cumulative_evictions=0}

 documentCache{lookups=0,hits=0,hitratio=0.00,inserts=0,evictions=0,size=0,warmupTime=0,cumulative_lookups=0,cumulative_hits=0,cumulative_hitratio=0.00,cumulative_inserts=0,cumulative_evictions=0}

Jan 23, 2012 1:10:03 PM org.apache.solr.update.processor.LogUpdateProcessor finish
This means that the old searcher was closed, all the requests that the old searcher was processing were responded. It also displays the final status of the searcher's cache. This information is sometimes valuable for sizing caches.

INFO: {commit=} 0 657
--Snippet--
```

> This means that the commit operation was finised in 657 ms (status 0 means OK)

* Replication:
--------------

> With no changes in the master
> The slave will periodically request to the master the version and generation of its index. For each of those request, the master will output: 

```
--Snippet--
Jan 23, 2012 4:46:40 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/replication params={command=indexversion&wt=javabin} status=0 QTime=0 
and if there hasn't been changes (master's version and generation is the same as slave's), the slave will log:

Jan 23, 2012 4:46:40 PM org.apache.solr.handler.SnapPuller fetchLatestIndex
INFO: Slave in sync with master.
With changes in the master
When replication is enabled and there are changes in the master, Solr will log all the steps that it will reproduce to copy the master's index to the slave and make it available for searches. Those steps are described here: http://wiki.apache.org/solr/SolrReplication#How_does_it_work.3F

INFO: Master's version: 1326979928332, generation: 6
Jan 23, 2012 5:21:50 PM org.apache.solr.handler.SnapPuller fetchLatestIndex
INFO: Slave's version: 1326979928328, generation: 5
This shows the version and generation of the indexes. In this case, the master's generation is grater, this means that the index will be copied to the slave.
The slave will then ask the master for its list of files. The request will look like this in the master:

Jan 23, 2012 5:21:50 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/replication params= {command=filelist&wt=javabin&indexversion=1326979928332} status=0 QTime=2
And after the response, the slave will output that number and see which files are present in its index and which aren't. The slave will output lines like:

INFO: Starting replication process
Jan 23, 2012 5:21:50 PM org.apache.solr.handler.SnapPuller fetchLatestIndex
INFO: Number of files in latest index in master: 34
Jan 23, 2012 5:21:50 PM org.apache.solr.handler.SnapPuller downloadIndexFiles
INFO: Skipping download for /home/user/Documents/solr/solr-trunk/trunk/solr/example_slave/solr/data/index.20120119103550/_2_0.frq
Jan 23, 2012 5:21:50 PM org.apache.solr.handler.SnapPuller downloadIndexFiles
INFO: Skipping download for /home/user/Documents/solr/solr-trunk/trunk/solr/example_slave/solr/data/index.20120119103550/_1_0.frq
Jan 23, 2012 5:21:50 PM org.apache.solr.handler.SnapPuller downloadIndexFiles
....
Jan 23, 2012 5:21:50 PM org.apache.solr.handler.SnapPuller fetchLatestIndex
INFO: Total time taken for download : 0 secs
In this case, the master has a total of 34 files, but some of them don't need to be downloaded, as they are already present in the slave (from a previous replication), for example _2_0.frq and _1_0.frq.
After this, the slave will request for those files that need to be downloaded. This will be one request per file and in the master it will look like: 

Jan 23, 2012 5:21:50 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/replication params={command=filecontent&checksum=true&indexversion=1326979928332&wt=filestream&file=_4_0.frq} status=0 QTime=0
Jan 23, 2012 5:21:50 PM org.apache.solr.core.SolrCore execute
INFO: [] webapp=/solr path=/replication params={command=filecontent&checksum=true&indexversion=1326979928332&wt=filestream&file=_4_nrm.cfe} status=0 QTime=0
...
--Snippet--
```

> After the new files are downloaded, the slave will proceed to commit it's own index (to make the new index available for searches), so the log will display lines exactly like the ones mentioned in the "commit" section.
