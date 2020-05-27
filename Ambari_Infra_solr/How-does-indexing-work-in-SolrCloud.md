When you send a document for indexing this is what happens:
-----------------------------------------------------------
```
1. The sharding algorithm decides which shard a document goes to. 
2. The document then gets forwarded to the shard leader and a version gets assigned to it.
3. The document is then indexed on the leader and is forwarded asynchronously to all replicas
4. At this point if you do a search and if some replicas haven't indexed the document then searches will give different results based on which replica you hit.
5. If you would like to control how many replicas have actually indexed the documents before success has been returned to the client then you can use the 'minRf' parameter in Solr. 'minRf' stands for minimum replication factor.
```
Here is an example update request where you can specify minRf

o Request:
----------
```
curl http://127.0.0.1:8983/solr/gettingstarted/update?min_rf=2  -H 'Content-type:application/json' -d '[{"id" : "1"}]'
```
o Response:
-----------
```
{"responseHeader":{"rf":2,"min_rf":2,"status":0,"QTime":22}}
```

> If you see the response it mentions the number of replicas it indexed to. If the number is less then what you sent it then it's the clients responsibility to re-index that document.

> SOLR-8062 aims to change the above mentioned behaviour and to throw an error instead to make the error more apparent.

> Here is an example when using SolrJ to achieve the same thing.

```
--Snippet--
SolrInputDocument doc = new SolrInputDocument();
doc.addField("id", "1");
UpdateRequest up = new UpdateRequest();
up.setParam(UpdateRequest.MIN_REPFACT, String.valueOf(2));
up.add(doc);
--Snippet--
```
