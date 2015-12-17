	注意事項：
		1. 所有指令皆使用 root 身份執行，僅供練習使用。
		2. 前置步驟在 master, slaver1, slaver2 都必須做一遍。
		3. PDF 格式部分文字會失真，輸入時請注意符號是否正確。

# Hadoop + HBase + Hive 建置手冊（完全分布式）

## 目錄

[toc]

## 套件清單

| Package          | Package Name                      | Installation Path     | Version     |
| :--------------- | :-------------------------------- | :-------------------- | :---------: |
| Oracle Java      | jdk-7u79-linux-x64.rpm            | /user/java/java       | 7           |
| Apache Hadoop    | hadoop-2.4.1.tar.gz               | /opt/hadoop           | 2.4.1       |
| Apache HBase     | hbase-0.98.13-hadoop2-bin.tar.gz  | /opt/hbase            | 0.98.13     |
| Apache Hive      | apache-hive-1.2.1-bin.tar.gz      | /opt/hive             | 1.2.1       |
| Apache Zookeeper | zookeeper-3.4.6.tar.gz            | /opt/zookeeper        | 3.4.6       |

[TOP](#toc_0)

## 環境配置

| OS            | IP             | Host Name    |
| :-----------: | :------------: | :----------: |
| CentOS 6.7    | 192.168.60.101 | master       |
| CentOS 6.7    | 192.168.60.102 | slaver1      |
| CentOS 6.7    | 192.168.60.103 | slaver2      |

[TOP](#toc_0)

## 前置步驟

### 安裝 JDK

	$ rpm -ivh /tmp/jdk-7u79-linux-x64.rpm
	$ ln -s /usr/java/jdk1.7.0_79 /usr/java/java

### 編輯 profile

	$ vim /etc/profile

> 增加內容
> 
	export JAVA_HOME=/usr/java/java
	export JRE_HOME=$JAVA_HOME/jre
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/rt.jar
	export PATH=$PATH:$JAVA_HOME/bin
	
### 載入 profile

	$ source /etc/profile

### 產生 SSH key

	$ ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ""
	$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	$ ssh localhost exit
	
### SSH asking disable

	$ vim /etc/ssh/ssh_config
	
> 修改內容
> 
	StrictHostKeyChecking no
	
### 重新啟動 SSH

	$ service sshd restart

### Security disable

	$ setenforce 0
	
### Permanent disable

	$ vim /etc/selinux/config
	
> 修改內容
> 
	SELINUX=disabled
	
### 關閉防火牆

	$ service iptables stop
	
### 開機不啟動防火牆
	
	$ chkconfig iptables off
	
[TOP](#toc_0)

## Apache Hadoop
	
### 安裝 Apache Hadoop

#### 解壓縮並建立連結

	$ tar -zxvf /tmp/hadoop-2.4.1.tar.gz
	$ mv hadoop-2.4.1 /opt
	$ ln -s /opt/hadoop-2.4.1 /opt/hadoop
	
#### 建立 Hadoop 暫存目錄

	$ mkdir -p /opt/hadoop/tmp

#### 編輯 hosts

	$ vim /etc/hosts

> 增加內容
> 
	192.168.60.101 master 
	192.168.60.102 slaver1 
	192.168.60.103 slaver2
	
#### 編輯 profile

	$ vim /etc/profile

> 增加內容
> 
	export HADOOP_HOME=/opt/hadoop/
	export HADOOP_MAPRED_HOME=$HADOOP_HOME
	export HADOOP_COMMON_HOME=$HADOOP_HOME
	export HADOOP_HDFS_HOME=$HADOOP_HOME
	export YARN_HOME=$HADOOP_HOME
	export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
	export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
	export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
	export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
	## HADOOP-9450
	export HADOOP_USER_CLASSPATH_FIRST=true
	
#### 載入 profile

	$ source /etc/profile
	
#### 編輯 slaves

	$ vim $HADOOP_HOME/etc/hadoop/slaves

> 覆蓋內容
> 
	slaver1
	slaver2

#### 編輯 core-site.xml

	$ vim $HADOOP_HOME/etc/hadoop/core-site.xml

> 覆蓋內容
> 
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	   <property>
	      <name>fs.defaultFS</name>
	      <value>hdfs://master:9000</value>
	   </property>
	   <property>
	      <name>hadoop.tmp.dir</name>
	      <value>/opt/hadoop/tmp</value>
	   </property>
	</configuration>
	
#### 編輯 hdfs-site.xml

	$ vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml
	
> 覆蓋內容
> 
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	   <property>
	      <name>dfs.replication</name>
	      <value>1</value>
	   </property>
	   <property>
	      <name>dfs.permissions</name>
	      <value>false</value>
	   </property>
	</configuration>
	
#### 編輯 mapred-site.xml

	$ vim $HADOOP_HOME/etc/hadoop/mapred-site.xml
	
> 覆蓋內容
> 
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	   <property>
	      <name>mapreduce.framework.name</name>
	      <value>yarn</value>
	   </property>
	</configuration>

#### 編輯 yarn-site.xml

	$ vim $HADOOP_HOME/etc/hadoop/yarn-site.xml
	
> 覆蓋內容
> 
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	   <property>
	      <name>yarn.resourcemanager.hostname</name>
	      <value>master</value>
	   </property>
	   <property>
	      <name>yarn.nodemanager.aux-services</name>
	      <value>mapreduce_shuffle</value>
	   </property>
	   <property>
	      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
	   </property>
	</configuration>
	
#### 編輯 hadoop-env.sh

	$ vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh
	
> 修改內容
> 
	export JAVA_HOME=/usr/java/java
	
#### Copy to slavers

	$ scp -rp /opt/hadoop slaver1:/opt
	$ scp -rp /opt/hadoop slaver2:/opt
	$ scp –rp /etc/hosts slaver1:/etc
	$ scp –rp /etc/hosts slaver2:/etc
	$ scp -rp /etc/profile root@slaver1:/etc
	$ scp -rp /etc/profile root@slaver2:/etc

#### Hadoop format

	$ hadoop namenode -format
	
#### Reboot all hosts
	
	$ ssh slaver1 reboot
	$ ssh slaver2 reboot
	$ reboot
	
### 啟動 Apache Hadoop

	$ start-dfs.sh
	$ start-yarn.sh

### 測試

	$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-*-tests.jar TestDFSIO -write
	$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-*-tests.jar TestDFSIO -clean
	$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 2 5

[TOP](#toc_0)

## Apache Zookeeper

### 安裝 ZooKeeper

#### 解壓縮並建立連結

	$ tar -zxvf /tmp/zookeeper-3.4.6.tar.gz
	$ mv zookeeper-3.4.6 /opt
	$ ln -s /opt/zookeeper-3.4.6 /opt/zookeeper

#### 編輯 profile

	$ vim /etc/profile

> 增加內容
> 
	export ZOOKEEPER_HOME=/opt/zookeeper
	export PATH=$PATH:$ZOOKEEPER_HOME/bin

#### 載入 profile

	$ source /etc/profile

#### 編輯 zoo.cfg
	
	$ cp $ZOOKEEPER_HOME/conf/zoo_sample.cfg $ZOOKEEPER_HOME/conf/zoo.cfg
	$ vim $ZOOKEEPER_HOME/conf/zoo.cfg
	
> 增修內容
>
	dataDir=/opt/zookeeper
	server.1=master:2888:3888
	
#### 編輯 myid

	$ vim /opt/zookeeper/myid

> 覆蓋內容
>
	1
	
### 啟動 ZooKeeper

	zkServer.sh start
	
[TOP](#toc_0)

## Apache HBase

### 安裝 HBase

#### 解壓縮並建立連結

	$ tar -zxvf /tmp/hbase-0.98.13-hadoop2-bin.tar.gz
	$ mv hbase-0.98.13-hadoop2 /opt
	$ ln -s /opt/hbase-0.98.13-hadoop2 /opt/hbase

#### 編輯 profile

	$ vim /etc/profile

> 增加內容
> 
	export HBASE_HOME=/opt/hbase
	export PATH=$PATH:$HBASE_HOME/bin
	
#### 載入 profile

	$ source /etc/profile

#### 編輯 regionservers

	$ vim $HBASE_HOME/conf/regionservers

> 增加內容
> 
	slaver1
	slaver2
	
#### 編輯 hbase-env.sh

	$ vim $HBASE_HOME/conf/hbase-env.sh
	
> 增加內容
> 
	export JAVA_HOME=/usr/java/java
	export HBASE_MANAGES_ZK=false
	
#### 編輯 hbase-site.xml

	$ vim $HBASE_HOME/conf/hbase-site.xml

> 覆蓋內容
> 
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	   <property>
	      <name>hbase.rootdir</name>
	      <value>hdfs://master:9000/hbase</value>
	   </property>
	   <property>
	      <name>hbase.cluster.distributed</name>
	      <value>true</value>
	   </property>
	   <property>
	      <name>hbase.zookeeper.quorum</name>
	      <value>master</value>
	   </property>
	   <property>
	      <name>hbase.zookeeper.property.dataDir</name>
	      <value>/opt/zookeeper</value>
	   </property>
	</configuration>
	
#### Remove hbase's log4j (bug)

	rm -rf $HBASE_HOME/lib/slf4j-log4j12-1.6.4.jar

#### Copy to slavers

	$ scp -rp /opt/hbase root@slaver1:/opt
	$ scp -rp /opt/hbase root@slaver2:/opt
	$ scp -rp /etc/profile root@slaver1:/etc
	$ scp -rp /etc/profile root@slaver2:/etc
	
### 啟動 HBase

	$ start-hbase.sh
	
[TOP](#toc_0)
	
## Apache Hive

### 安裝 Hive

#### 解壓縮並建立連結

	$ tar -zxvf /tmp/apache-hive-1.2.1-bin.tar.gz
	$ mv apache-hive-1.2.1-bin /opt
	$ ln -s /opt/apache-hive-1.2.1-bin /opt/hive
	
#### 編輯 profile

	$ vim /etc/profile

> 增加內容
> 
	export HIVE_HOME=/opt/hive
	export PATH=$PATH:$HIVE_HOME/bin
	
#### 載入 profile

	$ source /etc/profile
	
#### HDFS 上建立資料夾

	$ hadoop fs -mkdir /tmp
	$ hadoop fs -mkdir -p /user/hive/warehouse
	
#### 更改資料夾權限
	
	$ hadoop fs -chmod -R 777 /tmp
	$ hadoop fs -chmod -R 777 /user/hive/warehouse
	
#### 啟動 hiveserver2

	$ hiveserver2 &

#### 連線方式（新版）
	
	$ beeline -u jdbc:hive2://master:10000
		
#### 連線方式（傳統 可不用啟動 hiveserver2）

	$ hive
	
[TOP](#toc_0)



