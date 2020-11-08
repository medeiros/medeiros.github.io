---
layout: post
title: "Configure Hadoop in CentOS7 for Development Purposes"
description: >
  This post explains how to configure Hadoop from Zero to Hero in the simplest
  possible way, for development purposes.
categories: distributedarchitecture, bigdata
tags: [hadoop, hive, hbase, hdfs]
comments: true
image: /assets/img/blog/general/hadoop-logo.jpg
---
> This page presents a simple way to install Hadoop, Hive, HBase and
HDFS in a CentOS box using Single Node Cluster without Security. This is
particular useful to give developers a way to run, test and validate their
code (that depends on Hadoop) locally, without the need for a fully-configured
Hadoop cluster (which may be hard to get).
{:.lead}

- Table of Contents
{:toc}

## Configure and Run: Hadoop/HDFS

The following steps configure Hadoop in a CentOS7 environment. They were
mostly extracted from this [excellent post](https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-apache-hadoop-ubuntu-14-10-centos-7-single-node-cluster.html), tested and validated in practice.

This post assumes that the Single Node Cluster has the internal ip
`192.168.56.109`. Given that, the following commands must be run as `root`
(except, of course, when `su` is used to impersonate a `hadoop` user).

### Install Java

```bash
yum -y install java-1.8.0-openjdk wget

java -version
```

Output:

```
openjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)
```

### Create Hadoop User & Enable Passwordless Authentication

```bash
# creating a hadoop user
useradd -m -d /home/hadoop -s /bin/bash hadoop
passwd hadoop

# configure a passwordless ssh to the local system
su - hadoop
$ ssh-keygen
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys

# first time: type Yes to add RSA keys to hnown hosts
$ ssh 127.0.0.1
```

### Install and Configure Apache Hadoop

The configuration is this section prepares Hadoop, HDFS and Hive to work
properly.

The commands of this section must be run with `hadoop` user.

#### Download Hadoop

```bash
$ wget https://www-us.apache.org/dist/hadoop/common/stable/hadoop-3.2.0.tar.gz
$ tar -zxvf hadoop-3.2.0.tar.gz
$ mv hadoop-3.2.0 hadoop
```

#### Configure Env Vars

```bash
$ vim ~/.bashrc
# java realpath can be found with: $ ls -l /etc/alternatives/jre_1.8.0
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_6.x86_64/jre   
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

# apply env vars to the shell
$ source ~/.bashrc
```

#### Modify configuration files

In `$HADOOP_HOME/etc/hadoop/hadoop-env.sh` file:
```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_6.x86_64/jre
```

In `$HADOOP_HOME/etc/hadoop/core-site.xml` file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://192.168.56.109:9000</value>
  </property>

  <!-- for hive beeline -->
  <property>
    <name>hadoop.proxyuser.hadoop.groups</name>
    <value>*</value>
  </property>
  <property>
    <name>hadoop.proxyuser.hadoop.hosts</name>
    <value>*</value>
  </property>
</configuration>
```

In `$HADOOP_HOME/etc/hadoop/hdfs-site.xml` file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.name.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.data.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
  </property>
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>
</configuration>
```


In `$HADOOP_HOME/etc/hadoop/mapred-site.xml` file:
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=/home/hadoop/hadoop</value>
  </property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=/home/hadoop/hadoop</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=/home/hadoop/hadoop</value>
  </property>
</configuration>
```

In `$HADOOP_HOME/etc/hadoop/yarn-site.xml` file:
```xml
<?xml version="1.0"?>
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

#### Create NameNode and DataNode directories

Create those in the hadoop user’s home directory and with `hadoop` user:

```bash
mkdir -p ~/hadoopdata/hdfs/{namenode,datanode}
```

#### Format NameNode

```bash
hdfs namenode -format
```

#### Allow Hadoop through Firewall

Add the following entries to the `/etc/services` file (as root):

```properties
zookeeper   2181/tcp    # HBase is reached by clients via Zookeeper
hbase       16020/tcp
hadoop      9000/tcp   
hadoopnode  38033/tcp
hadoopnode2 9866/tcp    # data node TCP Port
hive        10000/tcp
```

And then run the following commands to open firewall for those ports and
for the default ports:

```bash
# default ports for http access
firewall-cmd --permanent --add-port=9870/tcp  
firewall-cmd --permanent --add-port=8088/tcp

firewall-cmd --permanent --add-port=2181/tcp
firewall-cmd --permanent --add-port=16020/tcp
firewall-cmd --permanent --add-port=9000/tcp
firewall-cmd --permanent --add-port=38033/tcp
firewall-cmd --permanent --add-port=9866/tcp
firewall-cmd --permanent --add-port=10000/tcp

firewall-cmd --reload
```

### Starting Hadoop

One can start all Hadoop services:

```bash
su - hadoop
$ ~/.hadoop/sbin/start-all.sh
```

Or start services individually:

#### Starting and Accessing Namenode (dfs)

```bash
su - hadoop
$ ~/.hadoop/sbin/start-dfs.sh
```

- Namenode Web UI: [http://192.168.56.109:9870](http://192.168.56.109:9870)

#### Starting and Accessing Resource Manager and NodeManagers (yarn)

```bash
su - hadoop
$ ~/.hadoop/sbin/start-yarn.sh
```

- Resource Manager Web UI: [http://192.168.56.109:8088](http://192.168.56.109:8088)

#### Test HDFS: Directory creation

```bash
su - hadoop
$ hdfs dfs -mkdir /tempdir
$ hdfs dfs -ls /tempdir
```

#### Testing Connectivity from client machine

The following commands will test if the required TCP ports (to client's
external access) are open and listening for connections:

```
$ nc -vz 192.168.56.109 9000
$ nc -vz 192.168.56.109 9866
```

## Configure and Run: Hive

Install Hive with the `hadoop` user, in the same parent directory of Hadoop,
as below:

### Download

Hadoop binary files can be found at [https://downloads.apache.org/hive/](https://downloads.apache.org/hive/).

```bash
su - hadoop
$ cd ~
$ wget https://downloads.apache.org/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
$ tar xvf apache-hive-3.1.2-bin.tar.gz
mv apache-hive-3.1.2 hive
```

### Replace Guava Version

In the current Hive version (3.1.2), there is a problem with Guava
compatibility. The solution, as described in [this Hive Jira issue](https://issues.apache.org/jira/browse/HIVE-22915),
is to use replace Guava version by Hadoop's version:

```bash
$ rm hive/lib/guava-19.0.jar
$ cp hadoop/share/hadoop/hdfs/lib/guava-27.0-jre.jar hive/lib
```

### Configure Env Vars for Hive

```bash
$ vim ~/.bashrc
export HIVE_HOME=/home/hadoop/hive
export PATH=$PATH:$HIVE_HOME/bin

# apply env vars to the shell
$ source ~/.bashrc
```

### Start Derby Schematool

```bash
$HIVE_HOME/bin/schematool -dbType derby -initSchema
```

### Start Hive

```bash
$ ./hive/bin/hive --service hiveserver2 \
    --hiveconf hive.server2.thrift.port=10000 \
    --hiveconf hive.root.logger=INFO,console &
```

### Access Hive via CLI and Create Some Data

```bash
$ beeline -u jdbc:hive2://192.168.56.109:10000/default -n hadoop
0: jdbc:hive2://192.168.56.109:10000> CREATE TABLE
  IF NOT EXISTS clients (code String)
  COMMENT 'Client Data'
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t'
  LINES TERMINATED BY '\n'
  STORED AS TEXTFILE;

0: jdbc:hive2://192.168.56.109:10000> INSERT INTO clients (code) VALUES ('123');

0: jdbc:hive2://192.168.56.109:10000> SELECT * FROM clients;
```

### Accessing Remotely

Since the TCP port 10000 was already opened and configuration files were
adjusted in previous (Hadoop) steps, there is no need to configure anything
else. Just use some Hive JDBC Driver to connect to
`jdbc:hive2://192.168.56.109:10000/default` with no password.

## Configure and Run: HBase

The steps to configure HBase in CentOS7 can be found [at this post, in this very same blog](/blog/bigdata,/linux/2020-10-19-Create-Virtual-Machine-on-Windows-with-HBase/#downloading-and-starting-hbase).


## References

- [Hadoop: How To Install Apache Hadoop on CentOS 7, Ubuntu 18.04 & Debian 9](https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-apache-hadoop-ubuntu-14-10-centos-7-single-node-cluster.html)
- [Hive: java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument](https://issues.apache.org/jira/browse/HIVE-22915)
- [HIVE: GettingStarted: Installation and Configuration](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-InstallationandConfiguration)
- [Connect To Hive](https://docs.bitnami.com/aws/apps/hadoop/administration/test-hive/)
- [Starting hiveserver2](https://stackoverflow.com/questions/31173113/starting-hiveserver2)
- [Cannot connect to hive using beeline, user root cannot impersonate anonymous](https://stackoverflow.com/questions/43180305/cannot-connect-to-hive-using-beeline-user-root-cannot-impersonate-anonymous)
