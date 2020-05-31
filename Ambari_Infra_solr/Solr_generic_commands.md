### Generic commands in Ambari-Infra-Solr and Apache Solr* 
---

* Note: Please change the port to 8886 when referring to Ambari Infra Solr and 8983 when referring to Apache Solr.

- To check if ambari-infra-solr server instance is running on the node:

	 ```ps -elf | grep -i infra-solr```
	 
 	 ```netstat -plant | grep -i 8886```

- If the cluster is Kerberized:
  - Check for valid kerberos tickets:
     
     ```klist -A```
  
  - Obtain a kerberos ticket if not present:
    
    ```$ kinit -kt /etc/security/keytabs/ambari-infra-solr.service.keytab $(klist -kt /etc/security/keytabs/ambari-infra-solr.service.keytab |sed -n "4p"|cut -d ' ' -f7) ```


#### 1) LIST SOLR COLLECTIONS:
-------------------------

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=list&indent=true"
```

#### 2) CREATE A COLLECTION:
-----------------------

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8983/solr/admin/collections?action=CREATE&name=<collection_name>" 
```

Optional Values which are necessary when creating a collection manually using APIs:

```
&numShards=<number>
&collection.configName=nameofconfiguration
&maxShardsPerNode=<number>
&replicationFactor=<number>
```

#### 3) DELETE A COLLECTION & CONFIG set:
------------------------------------

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=DELETE&name=ranger_audits" 
```

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/configs?action=DELETE&name=ranger_audits" 
```

#### 4) CHECK SOLR cloud CLUSTER STATUS:
-----------------------------------

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=clusterstatus&wt=json&indent=true"
```

The below will beautify the Json format:

```
$ curl --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=clusterstatus&wt=json&indent=true" | python -m json.tool 
```

#### 5) To issue a manual commit to SOLR:
-------------------------------------

```
$ curl -ikv --negotiate -u: "http://$(hostname -f):8983/solr/<collection_name>/update?commit=true"
```

-> This is going to do a hard commit and create a new segment with all the documents in it on the local disk. 


#### 6) To query a collection from command line:
-------------------------------------------

```
$ curl -ikv --negotiate -u: "http://$(hostname -f):8886/solr/ranger_audits/select?q=*%3A*&wt=json&rows=10&indent=true"
```

- Optional:

Can use the query to sort using ascending (asc) or descending (desc) order.

For example: To sort the output according to the "_version_" feild:

```
$ curl -ikv --negotiate -u: http://c2218-node2.labs.support.hortonworks.com:8886/solr/ranger_audits/select?q=*:*&rows=10&sort=_version_%20asc&wt=json
```

```
$ curl -ikv --negotiate -u: http://c2218-node2.labs.support.hortonworks.com:8886/solr/ranger_audits/select?q=*:*&rows=10&sort=_version_%20desc&wt=json
```

- Filter query (fq) example:

```
$ curl -ikv --negotiate -u http://$(hostname -f):8886/solr/<collection/shard name>/select?q=*:*&fq=repo:Apollo_kms&fq=result:0&wt=json&indent=true
```

- [Solr Admin UI (query) page](https://lucene.apache.org/solr/guide/6_6/query-screen.html#query-screen)



#### 7) To back up a SOLR collection in Solr Cloud mode:
---------------------------------------------------

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=BACKUP&name=<myBackupName>&collection=<myCollectionName>&location=</path/to/my/shared/drive>
```

#### 8) To restore a SOLR collection in Solr cloud mode:
---------------------------------------------------

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=RESTORE&name=<myBackupName>&location=</path/to/my/shared/drive>&collection=<myRestoredCollectionName>
```

#### 9) To delete the old entires from the collections log:
------------------------------------------------------

```
$ curl -ikv --negotiate -u: "http://$(hostname -f):8886/solr/ranger_audits/update?commit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>evtTime:[* TO NOW-15DAYS]</query></delete>"
```

- This is going to delete data before 15 days from the current date.


#### 10) To add replica for the collections:
---------------------------------------

```
curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=ADDREPLICA&collection=<collection_name>&shard=shard1&node=<hostname of the server>:8886_solr"
```

- Example:

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=ADDREPLICA&collection=vertex_index&shard=shard1&node=c2218-node2.labs.suqadron.hortonworks.com:8886_solr"
```

***NOTE: More details can be found at:
[Lucene Documentation](https://lucene.apache.org/solr/guide/6_6/collections-api.html)


#### 11) Split Shard on ranger_audits collection:
--------------------------------------------

```
$ curl "http://$(hostname -f):8886/solr/admin/collections?action=SPLITSHARD&collection=ranger_audits&shard=shard1&indent=true"
```

#### For Kerberos:
---

```
curl -ikv --negotiate -u: "http://$(hostname -f):8886/solr/admin/collections?action=SPLITSHARD&collection=ranger_audits&shard=shard1&indent=true"
```

***Can use an "async" call to put the thread in the BG.


#### Backup the OLD configs for Ambari Infra Solr for ranger_audits collection:
----------------------------------------------------------------------------

-> This can be done for any config set that is available in ZK. The config sets are present under `/infra-solr/configs`.

```
#/usr/lib/ambari-infra-solr/server/scripts/cloud-scripts/zkcli.sh -zkhost c165-node2.local.domain:2181/infra-solr -cmd downconfig -confname ranger_audits -confdir /tmp/OLDRANGER/ 
```

#### Solr check index tool:
------------------------

- This is helpful when you have to perform a index check for any corruption.

```
$ $JAVA_HOME -cp /opt/lucidworks-hdpsearch/solr/server/solr-webapp/webapp/WEB-INF/lib/lucene-core-5.2.1.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex /path/to/the/index/directory/data/ -verbose
```

-> This will give Verbose you information on the corruption of the index.

For example: 

```
$ /usr/jdk64/jdk1.8.0_92/bin/java -cp /opt/lucidworks-hdpsearch/solr/server/solr-webapp/webapp/WEB-INF/lib/lucene-core-5.2.1.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex /opt/lucidworks-hdpsearch/solr/server/solr/DOCSS_UAT2_shard2_replica1/data/ -verbose 2> &1 | tee check_index_verbose_out.txt 
```

-> The below is going to help you remove the corrupt indexes.

```
$ $JAVA_HOME -cp /opt/lucidworks-hdpsearch/solr/server/solr-webapp/webapp/WEB-INF/lib/lucene-core-5.2.1.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex /path/to/the/index/directory/data/ -exorcise 2 > &1 | tee exorcise_out.txt
```

-> This will remove the corrupted index file. Please make sure that the "exorcise" command is run with "infra-solr" (service) user (there might be some data loss as the corrupt segments are purged). The command is going to delete a corrupt index file and create a new one merging the un-merged indexes.

For example: 

```
$ /usr/jdk64/jdk1.8.0_92/bin/java -cp /opt/lucidworks-hdpsearch/solr/server/solr-webapp/webapp/WEB-INF/lib/lucene-core-5.2.1.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex /opt/lucidworks-hdpsearch/solr/server/solr/DOCSS_UAT2_shard2_replica1/data/ -exorcise
```

#### Helpful articles:
-------------------

[Understanding Ambari Infra](https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/using-ambari-core-services/content/amb_understanding_ambari_infra.html)

[Ambari HDF Upgrade](https://docs.hortonworks.com/HDPDocuments/HDF3/HDF-3.4.0/ambari-managed-hdf-upgrade-ppc/content/hdf-upgrade-ambari-infra.html)


#### Updating the security.json in Kerberized clusters for Apache Solr:
--------------------------------------------------------------------

```
$ /opt/lucidworks-hdpsearch/solr/server/scripts/cloud-scripts/zkcli.sh -zkhost c3218-node2.hwx.internal2:2181,c3218-node3.hwx.internal2:2181,c3218-node4.hwx.internal2:2181 -cmd put /solr/security.json '{"authentication":{"class": "org.apache.solr.security.KerberosPlugin"}}'  
```

---
- [How-to-setup-and-secure-a-SolrCloud-cluster-with-Kerberos-and-Ranger](https://support.hortonworks.com/s/article/How-to-setup-and-secure-a-SolrCloud-cluster-with-Kerberos-and-Ranger)


- [Modifying-Ranger-Audit-Solr-Config](https://community.cloudera.com/t5/Community-Articles/Modifying-Ranger-Audit-Solr-Config/ta-p/244899)


- Documentation for [Solr Admin UI](https://lucene.apache.org/solr/guide/6_6/using-the-solr-administration-user-interface.html):


- [Oliver Szabo's github link with Asciinemas](https://github.com/apache/ambari-infra/tree/master/ambari-infra-solr-client)
---

