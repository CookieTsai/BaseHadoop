## Quick-Start Cloudera CDH5.3.X Apache HBase 

* Download Apache HBase **hbase-0.98.6-cdh5.3.2.tar.gz** 
	
	[**下載網址**](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_vd_cdh_package_tarball.html)
	
* Unzip hbase-0.98.6-cdh5.3.2.tar.gz

	```
	HostName:Downloads UserName$ tar zxvf hbase-0.98.6-cdh5.3.2.tar.gz
	```

* Set JAVA_HOME in Terminal **on MacBook**

	```
	HostName:~ UserName$ JAVA_HOME=`/usr/libexec/java_home`  
	HostName:~ UserName$ export=JAVA_HOME
	```

* Set HBASE_HOME in Terminal

	```
	HostName:~ UserName$ HBASE_HOME={Your HBase Home}
	HostName:~ UserName$ export HBASE_HOME
	```

* Edit **hbase-site.xml**

	``` hbase-site.xml
	<?xml version="1.0"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	
	<configuration>
		<property>
			<name>hbase.rootdir</name>
		    <value>file:///tmp/hbase</value>
		</property>
		<property>
		    <name>hbase.zookeeper.property.dataDir</name>
		    <value>/tmp/zookeeper</value>
		</property>
	</configuration>

	```

* Start The HBase

	```
	HostName:~ UserName$ $HBASE_HOME/bin/start-hbase.sh
	```