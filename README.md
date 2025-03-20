# Trino
**Deployment &amp; Configurations of Trino**


**Introduction**
Trino, formerly known as Presto SQL, is an open-source distributed SQL query engine designed for running interactive analytic queries against data sources of various sizes. It was initially developed by Facebook and later became a project of the Presto Software Foundation. Trino is optimized for low-latency queries on data stored in various data stores, including Hadoop Distributed File System (HDFS), Amazon S3, Apache Hive, relational databases like MySQL, PostgreSQL, and many others.


**Installation:
**
You must visit its official website to download latest trino release.
https://trino.io/docs/current/installation/deployment.html

Note, Trino latest available versions are as follows: 
https://repo1.maven.org/maven2/io/trino/trino-server/


**Steps:
**

wget https://repo1.maven.org/maven2/io/trino/trino-server/449/trino-server-($Version).tar.gz

tar xzvf trino-server-449.tar.gz

mkdir -p trino-server-449/etc/catalogs


Create a dedicated directory for storing its logs
mkdir -p /var/trino/data


**vim trino-server-449/etc/node.properties
**
node.environment=production

node.id=ffffffff-ffff-ffff-ffff-ffffffffffff

node.data-dir=/var/trino/data

Note: node.id can be any unique identifier.


**vim trino-server-449/etc/jvm.config**

-server

-Xmx16G

-XX:InitialRAMPercentage=80

-XX:MaxRAMPercentage=80

-XX:G1HeapRegionSize=32M

-XX:+ExplicitGCInvokesConcurrent

-XX:+ExitOnOutOfMemoryError

-XX:+HeapDumpOnOutOfMemoryError

-XX:-OmitStackTraceInFastThrow

-XX:ReservedCodeCacheSize=512M

-XX:PerMethodRecompilationCutoff=10000


-XX:PerBytecodeRecompilationCutoff=10000

-Djdk.attach.allowAttachSelf=true

-Djdk.nio.maxCachedBufferSize=2000000

-Dfile.encoding=UTF-8

-XX:+EnableDynamicAgentLoading

-XX:+UnlockDiagnosticVMOptions

-XX:G1NumCollectionsKeepPinned=10000000


Note: Based on your available memory in your node, you can configure Xmx to your maximum, e.g. free -h and check how much free memory is there in your system, and just have 2 GB for buffer so your operating system can work smoothly. Because once trino occupied the memory it wont release unless you stop the trino services. Even there is not query running on trino still its memory would be occupied.



**vim trino-server-449/etc/config.properties
**
The following is a minimal configuration for the coordinator:

coordinator=true

node-scheduler.include-coordinator=false

http-server.http.port=8080

discovery.uri=http://example.net:8080

And this is a minimal configuration for the workers:

coordinator=false

http-server.http.port=8080

discovery.uri=http://example.net:8080


Alternatively, if you are setting up a single machine for testing, that functions as both a coordinator and worker, use this configuration:

coordinator=true

node-scheduler.include-coordinator=true

http-server.http.port=8080

discovery.uri=http://example.net:8080


**vim trino-server-449/etc/log.properties**

io.trino=INFO

Note: It also can be io.trino=DEBUG incase your trino is not starting.

**Catalog properties**

Catalogs are registered by creating a catalog properties file in the etc/catalog directory. For example, create etc/catalog/jmx.properties with the following contents to mount the jmx connector as the jmx 

catalog:

connector.name=jmx


See Connectors for more information about configuring connectors.

https://trino.io/docs/current/connector.html


Note: For connecting with any source like Hive, S3, Mysql, Clickhouse, kafka etc. you need to create catalog file inside etc/catalog/
In case of mysql:


vim etc/catalog/mysql.properties

connector.name=mysql

connection-url=jdbc:mysql://example.net:3306

connection-user=root


connection-password=secret


Note, after creating or changing any catalog, you need to restart trino service.

**Running Trino**

trino-server-449/bin/launcher start

**Check Status**

trino-server-449/bin/launcher -v status

**For Stop Trino**

trino-server-449/bin/launcher stop


Once Trino is started and its status is running, you can check its port is up on server by below command:

netstat -tlnp | grep 8080


you can access in the browser if firewall is disabled or you need to open this port on firewall level.
http://example.net:8080

