NOTE: There is a known issue with Java 1.8 update 242. The detailed information can be found here:

~~~
HDP Services not starting with JDK 8u242 (Keberized cluster)
https://jira.cloudera.com/browse/TSB-394
~~~

### With help from Kevin Risden who is the original author of the article:
----------------------------------------------------------------------

~~~
Apache Ambari Infra Solr and Apache Ranger - Fixing OOM
https://risdenk.github.io/2017/12/18/ambari-infra-solr-ranger.html
~~~

o Replacing the "managed-schmea" for "ranger_audits":
---------------------------------------------------
```
#ssh into Ambari Infra Solr host
#sudo -u infra-solr -i
```

#### If using Kerberos:
```
$ kinit -kt /etc/security/keytabs/ambari-infra-solr.service.keytab $(whoami)/$(hostname -f)
```

#### Set environment for zkcli
```
$ source /etc/ambari-infra-solr/conf/infra-solr-env.sh
$ export SOLR_ZK_CREDS_AND_ACLS="${SOLR_AUTHENTICATION_OPTS}"
```

#### Download pre-edited
```
$ wget -O managed-schema https://gist.githubusercontent.com/risdenk/8cc8f722e200468f9aa536cee7979d06/raw/aa61053847b84e40c3bae8adf806e68b5a1408d3/managed-schema.xml
```

#### Download from zookeeper and edit:
```
$ /usr/lib/ambari-infra-solr/server/scripts/cloud-scripts/zkcli.sh --zkhost "${ZK_HOST}" -cmd getfile /infra-solr/configs/ranger_audits/managed-schema /tmp/managed-schema.xml.orig
```  

#### Upload configuration back to Zookeeper:
```
$ /usr/lib/ambari-infra-solr/server/scripts/cloud-scripts/zkcli.sh --zkhost "${ZK_HOST}" -cmd putfile /infra-solr/configs/ranger_audits/managed-schema managed-schema

$ /usr/hdp/current/zookeeper-client/bin/zkCli.sh -server <zookeeper-node>:2181 set /infra-solr/configs/audit_logs/managed-schema "`cat /usr/lib/ambari-logsearch-portal/conf/solr_configsets/audit_logs/conf/managed-schema`"
```

> Delete and recreate the ranger_audits collection.

#### If using Kerberos, add "-u : --negotiate" to the curl commands below
```
$curl -i "http://$(hostname -f):8886/solr/admin/collections?action=DELETE&name=ranger_audits"
$curl -i "http://$(hostname -f):8886/solr/admin/collections?action=CREATE&name=ranger_audits&numShards=5&maxShardsPerNode=10"
```
