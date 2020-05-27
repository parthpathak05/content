#### 1. Use Case: 
-----------

-> In most of the cases, the size of the "ranger_audits" collections is 100s of GBs in that case, using the API and the curl command to delete the collection can take a lot of time. 
-> To by pass the same below is the procedure. 
-> This gets your work done fast, but, it is not recommended. Use the APIs to delete as it is going to take care of removing the right parts and details.

#### 2. Procedure:
------------

1. Put in maintenance mode and stop Ranger Admin.
2. Stop Ambari Infra Solr from Ambari.
3. Locate the data directory for Ambari-Infra-Solr. It can be found at this location in Ambari:

```
** Ambari > Ambari Infra (OR Infra Solr) > Configs > Infra Solr data dir.
```

The alternative way is:

```
# ps -lef | grep -i infra-solr | grep solr.solr.home
```

4. To remove the "ranger_audits" collection (this needs to be done on all the hosts having Infra-Solr master service with the infra-solr service user):

```
$ rm -rf $solr.solr.home/ranger_audits_*
```

5. Login into ZK from any one of the Infra Solr hosts (kinit with infra-solr keytab if kerberos is in action).

```
rmr /infra-solr/collections/ranger_audits
```

6. Start Ambari Infra Solr service from Ambari UI and then run the below curl command with infra-solr service user:

```
$ curl -ikv --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=DELETE&name=ranger_audits" 
```

The above command is going to throw an error but, is going to make sure that there is no stale reference remaining.

7. Start Ranger Admin service and check the Ambari operation logs for the service startup. Ranger Admin service will take care of creating the collections in Solr.

[NOTE: This procedure can be applied to any other collection in the Ambari Infra Solr cloud cluster. For eg. fulltext_index, vertex_index, edge_index, hadoop_logs, history, audit_logs]
