~~~
Example Active Directory Configuration
https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/configuring-proxy-knox/content/example_active_directory_configuration.html
~~~

~~~
Apache Knox Gateway 0.8.x User’s Guide
http://knox.apache.org/books/knox-0-8-0/user-guide.html#Apache+Knox+Gateway+0.8.x+User's+Guide
~~~

~~~
Contents (HDP 2.6.5.0 using Active Directory)
https://github.com/HortonworksUniversity/Security_Labs/blob/master/HDP-2.6-AD.md#contents--hdp-2650-using-active-directory
~~~

NOTE: 

1. If Ranger plugin for KNOX is enabled, make sure that it is present in the list of users in the Policy for Knox. Otherwise it will throw HTTP 403 autorization error in the browser.

2. Also, please check the whitelisting of the URLs in the knoxsso topology section before trying to accessing UIs through PROXY.
USE: https://regex101.com/ for evaluation of regex for URLs.

3. KNOX PROXY and KNOXSSO are two different concepts. Using KNOX PROXY you are not exposing the internal cluster URLs to the external world.

4. KNOXSSO is used for setting up single sign on so that, you will not need to sign on by entering the credentials everytime for every service in the cluster.


#### * default.xml
-------------

```
<topology>
	<gateway>
		<provider>
			<role>authentication</role>
			<name>ShiroProvider</name>
			<enabled>true</enabled>
			<param>
				<name>sessionTimeout</name>
				<value>30</value>
			</param>
			<param>
				<name>main.ldapRealm</name>
				<value>org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm</value>
			</param>
			<!-- Section not needed -->
			<param>
				<name>main.ldapRealm.userDnTemplate</name>
				<value>uid={0},ou=people,dc=hadoop,dc=apache,dc=org</value>
			</param>
			<!-- Section not needed -->
			<param>
				<name>main.ldapRealm.contextFactory.url</name>
				<value>ldap://<host>:389</value>
			</param>
			<param>
				<name>main.ldapRealm.contextFactory.systemUsername</name>
				<value>cn=test1,ou=hortonworks,dc=support,dc=com</value>
			</param>
			<param>
				<name>main.ldapRealm.contextFactory.systemPassword</name>
				<value>hadoop12345!</value>
			</param>
			<!-- START: changes needed for user sync-->
			<param>
				<name>main.ldapRealm.searchBase</name>
				<value>ou=users,ou=hortonworks,dc=support,dc=com</value>
			</param>
			<param>
				<name>main.ldapRealm.userSearchAttributeName</name>
				<value>sAMAccountName</value>
			</param>
			<param>
				<name>main.ldapRealm.userObjectClass</name>
				<value>person</value>
			</param>
			<!-- END: changes needed for user sync-->
			<!-- START: changes needed for group sync-->
			<param>
				<name>main.ldapRealm.authorizationEnabled</name>
				<value>true</value>
			</param>
			<param>
				<name>main.ldapRealm.groupSearchBase</name>
				<value>ou=groups,ou=hortonworks,dc=support,dc=com</value>
			</param>
			<param>
				<name>main.ldapRealm.groupObjectClass</name>
				<value>group</value>
			</param>
			<param>
				<name>main.ldapRealm.groupIdAttribute</name>
				<value>cn</value>
				<!-- END: changes needed for group sync-->
			</param>
			<param>
				<name>main.ldapRealm.contextFactory.authenticationMechanism</name>
				<value>simple</value>
			</param>
			<param>
				<name>urls./**</name>
				<value>authcBasic</value>
			</param>
		</provider>
		<provider>
			<role>identity-assertion</role>
			<name>Default</name>
			<enabled>true</enabled>
		</provider>
		<provider>
			<role>authorization</role>
			<name>AclsAuthz</name>
			<enabled>true</enabled>
		</provider>
		<provider>
			<role>ha</role>
			<name>HaProvider</name>
			<enabled>true</enabled>
			<param>
				<name>WEBHDFS</name>
				<value>maxFailoverAttempts=3;failoverSleep=1000;maxRetryAttempts=300;retrySleep=1000;enabled=true
				</value>
			</param>
			<param>
				<name>HIVE</name>
				<value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true;zookeeperEnsemble=nodeXX.coelab.cloudera.com:2181,nodeXX.coelab.cloudera.com:2181,nodeXX.coelab.cloudera.com:2181;zookeeperNamespace=hiveserver2</value>
			</param>
			<param>
				<name>OOZIE</name>
				<value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true</value>
			</param>
			<param>
				<name>HDFSUI</name>
				<value>maxFailoverAttempts=3;failoverSleep=1000;maxRetryAttempts=300;retrySleep=1000;enabled=true</value>
			</param>
			<param>
				<name>HBASE</name>
				<value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true</value>
			</param>
			<param>
				<name>WEBHCAT</name>
				<value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true</value>
			</param>
		</provider>
	</gateway>
	<service>
		<role>NAMENODE</role>
		<url>hdfs://nodeXX.coelab.cloudera.com:8020</url>
		<url>hdfs://nodeXX.coelab.cloudera.com:8020</url>
	</service>
	<service>
		<role>JOBTRACKER</role>
		<url>rpc://nodeXX.coelab.cloudera.com:8050</url>
	</service>
	<service>
		<role>WEBHDFS</role>
		<url>http://nodeXX.coelab.cloudera.com:50070/webhdfs</url>
		<url>http://nodeXX.coelab.cloudera.com:50070/webhdfs</url>
	</service>
	<service>
		<role>HDFSUI</role>
		<url>http://nodeXX.coelab.cloudera.com:50070</url>
		<url>http://nodeXX.coelab.cloudera.com:50070</url>
	</service>
	<service>
		<role>YARNUI</role>
		<url>http://nodeXX.coelab.cloudera.com:8088/ui2</url>
		<url>http://nodeXX.coelab.cloudera.com:8088/ui2</url>
	</service>
	<service>
		<role>OOZIE</role>
		<url>http://nodeXX.coelab.cloudera.com:11000/oozie</url>
	</service>
	<service>
		<role>OOZIEUI</role>
		<url>http://nodeXX.coelab.cloudera.com:11000/oozie</url>
	</service>
	<service>
		<role>HIVE</role>
	</service>
	<service>
		<role>SERVICE-TEST</role>
	</service>
</topology>
```
