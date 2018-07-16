---
layout: post
title: hadoop-hive
date: 2018-05-18
description: 
tags:
- apache
- hadoop
- hive
permalink: hadoop-hive-setup
---

It has been several years since I started about knowing data research. And during now, I had some chance to study/use machine learning algorithm with great ML/DL modules, but didn't think deep about storing or handling big data, because the data I treated was not so big. So for now, I'm trying to go on setting up hadoop, and some of useful modules in hadoop echo system to getting know about this.


## Setup Hadoop

{% highlight shell %}
{% raw %}

{% endraw %}
{% endhighlight %}

First, download hadoop from apache. Most of the link I found for hadoop was broken. This link works well for know, but could be lost any time.
Download, decompress, and place the file where you want.
{% highlight shell %}
{% raw %}
$ wget http://mirror.apache-kr.org/hadoop/common/hadoop-2.8.4//hadoop-2.8.4.tar.gz
$ tar xvf hadoop-2.8.4.tar.gz
{% endraw %}
{% endhighlight %}

After unzip, setup directory path in `.bash_profile`
{% highlight shell %}
{% raw %}
...
# Set Hadoop
export HADOOP_HOME=/path/to/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
...
{% endraw %}
{% endhighlight %}


Now you need to setup configuration files in this package. Update lines for file

etc/hadoop/core-site.xml:
{% highlight xml %}
{% raw %}
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
{% endraw %}
{% endhighlight %}

etc/hadoop/hdfs-site.xml:
{% highlight xml %}
{% raw %}
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
{% endraw %}
{% endhighlight %}


Before you start, you could enable to access `ssh localhost` in your console. If it returns like `Permission Denied`, you need to generate passphrase.
{% highlight shell %}
{% raw %}
$ cd ~/
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
{% endraw %}
{% endhighlight %}


Now start with command in `sbin` directory. Using `start-all.sh` to run `Yarn` and `HDFS`(it will recommend to use `start-dfs` and `start-yarn`, but it does not cause a problem).
{% highlight shell %}
{% raw %}
$ ./sbin/start-all.sh
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /Users/kwangin/utils/hadoop/logs/hadoop-kwangin-namenode-kwangin-ui-MacBook-Pro.local.out
localhost: starting datanode, logging to /Users/kwangin/utils/hadoop/logs/hadoop-kwangin-datanode-kwangin-ui-MacBook-Pro.local.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /Users/kwangin/utils/hadoop/logs/hadoop-kwangin-secondarynamenode-kwangin-ui-MacBook-Pro.local.out
18/06/10 20:53:25 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /Users/kwangin/utils/hadoop/logs/yarn-kwangin-resourcemanager-kwangin-ui-MacBook-Pro.local.out
localhost: starting nodemanager, logging to /Users/kwangin/utils/hadoop/logs/yarn-kwangin-nodemanager-kwangin-ui-MacBook-Pro.local.out
{% endraw %}
{% endhighlight %}

Now you can store data in hdfs system.
{% highlight shell %}
{% raw %}
$ hdfs dfs -mkdir your_dir
...
$ hadoop fs -put /root/MyHadoop/file1.txt your_dir
{% endraw %}
{% endhighlight %}


## Setup Hive
To execute SQL applications and queries over distributed data, you should implement it with MapReduce Java API, and It was quite a job. Some of people thought about it in early time, and implemented projects which called `SQL-on-Hadoop`. 
It a class set of analytical application tools that combines SQL-like querying with newer Hadoop data framework elements. By supporting (something like)SQL queries, it had been a great help to lot of data engineer and scientist work in Hadoop clusters. 
In nowadays(2018), there are over 10 projects which offers query for hadoop(though all of them has there own purpose...). In here, I'll look on with project `Hive`.

Apache Hive is a data warehouse software project built on top of Apache Hadoop for providing data summarization, query and analysis. It is one of the oldest, and popular 'SQL-on-Hadoop' project.

Download it first(newest version is 2.3.3):
{% highlight shell %}
{% raw %}
$ cd /usr/local
$ wget wget http://mirror.apache-kr.org/hive/hive-2.3.3/apache-hive-2.3.3-bin.tar.gz
$ tar xvf apache-hive-2.3.3-bin.tar.gz
{% endraw %}
{% endhighlight %}


Hive needs to know the path of Hadoop installed, so define it in environment file in `conf/hive-env.sh`

{% highlight shell %}
{% raw %}
...
HADOOP_HOME=/path/to/hadoop
...
{% endraw %}
{% endhighlight %}

Now you could start hive by `hive` command. But you could face on error something like this when trying to run any query, such as `show databases;`.
```
...
Exception in thread "main" java.lang.RuntimeException: Hive metastore database is not initialized. Please use schematool (e.g. ./schematool -initSchema -dbType ...) to create the schema. If needed, don't forget to include the option to auto-create the underlying database in your JDBC connection string
...
```

For Hive >= 2.0

Metadata is stored in an embedded Derby database whose disk storage location is determined by the Hive configuration variable named javax.jdo.option.ConnectionURL. By default, this location is ./metastore_db (see conf/hive-default.xml). 
You can also setup by using mysql, but using Derby would be more simple.

{% highlight shell %}
{% raw %}
$ cd bin
$ schematool -initSchema -dbType derby
SLF4J: Class path contains multiple SLF4J bindings.
...
Metastore connection URL:	 jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :	 org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:	 APP
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.derby.sql
Initialization script completed
schemaTool completed
...
{% endraw %}
{% endhighlight %}


## Access HDFS with Hive

To use Hive, you should setup file directory to create data table. 
{% highlight shell %}
{% raw %}
$ cd /usr/local/hadoop-2.8.4
$ bin/hadoop fs -mkdir /tmp
$ bin/hadoop fs -mkdir /user
$ bin/hadoop fs -mkdir /user/hive
$ bin/hadoop fs -mkdir /user/hive/warehouse
$ bin/hadoop fs -chmod g+w /tmp
$ bin/hadoop fs -chmod g+w /user/hive/warehouse
{% endraw %}
{% endhighlight %}

If it works well, you could see directory with `hdfs` command.

{% highlight shell %}
{% raw %}
$ bin/hdfs dfs -ls /

{% endraw %}
{% endhighlight %}



## Read


hive-site.xml



