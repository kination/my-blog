---
layout: post
title: Start-on Hadoop, with Hive
date: 2018-05-18
description: 
tags:
- apache
- hadoop
- hive
permalink: hadoop-hive-setup
---

It has been several years since I started about knowing data research. And during now, I had some chance to study/use machine learning algorithm with great ML/DL modules, but didn't think deep about storing or handling big data, because the data I treated was not so big. So for now, I'm trying to go on setting up hadoop, and some of useful modules in hadoop echo system to getting know about this.

![Screenshot](/assets/post_img/hadoop-hive-setup/apache-hadoop.png)

Apache Hadoop is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. For storing quickly growing data, the scalability of Hadoop was fit to big data managing. And with this advantage, lots of data engineering project has been created based on Hadoop, and they formed as hadoop ecosystem.

![Screenshot](/assets/post_img/hadoop-hive-setup/hadoop-ecosystem.png)

I will keep going on these projects, and setting up Hadoop is the first step for this.


## Setup Hadoop

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

This shows the url to access hdfs system. Usual default is `localhost:9000`.

etc/hadoop/hdfs-site.xml:
{% highlight xml %}
{% raw %}
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>file:///Users/jung/utils/hdfs-manage/name</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>file:///Users/jung/utils/hdfs-manage/data</value>
    </property>
</configuration>
{% endraw %}
{% endhighlight %}

`dfs.replication` is a property which defines the number of replication block. Because hdfs system is designed as fault-tolerant, there are replicated blocks for keeping data available. HDFS system will keep number of '`dfs.replication` - 1' backup blocks. Because this is just a testing, no replication is needed, so 1 will be enough.
`dfs.name.dir` and `dfs.data.dir` is to define the local path to store hdfs data. This is important because if you don't setup this, every data in hdfs will be vanished when you stop hdfs.
Add `file://` in front of local path to make it find from your computer.

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
![Screenshot](/assets/post_img/hadoop-hive-setup/apache-hive.png)
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

And you need to give information where you will keep data for hive in `conf/hive-site.xml`. You can copy file `hive-default.xml.template`(it is in `conf/`), change name to `hive-site.xml` and edit it. Or just make a new one.

{% highlight xml %}
{% raw %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
...
<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
</property>
...
</configuration>
{% endraw %}
{% endhighlight %}

Create directory for warehouse in hdfs.

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

This defines the location of metastore warehouse. Metastore is for keeping informations related to databases, tables, and their relations in metadata form.

Now you could start hive by `hive` command. But you could face on error something like this when trying to run any query, such as `show databases;`.
```
...
Exception in thread "main" java.lang.RuntimeException: Hive metastore database is not initialized. Please use schematool (e.g. ./schematool -initSchema -dbType ...) to create the schema. If needed, don't forget to include the option to auto-create the underlying database in your JDBC connection string
...
```

Okay, it said metastore is not initialized. 
Metadata is stored in an embedded database whose disk storage location is determined by the Hive configuration variable named javax.jdo.option.ConnectionURL. Add properties for this.

{% highlight xml %}
{% raw %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
...
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:metastore_db;create=true</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>org.apache.derby.jdbc.EmbeddedDriver</value>
</property>
...
</configuration>
{% endraw %}
{% endhighlight %}

This means I'll store metastore data with `Apache Derby`. You can also use mysql, but using Derby would be more simple. Recent version of hive includes derby module, so you don't need to install it. If you want to use seperate derby application, set in to `ConnectionURL`. 

Now create initiate database.
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

If it works, you will see `metastore_db` directory created.


## Import CSV, and generate database
Installation is done, so setup file directory to create data table. Create a dummy directory for test, and put CSV file for test. The file is the training CSV file from titanic competition in [Kaggle](https://www.kaggle.com).
{% highlight shell %}
{% raw %}
$ bin/hadoop fs -mkdir /user/hive/test
$ hdfs dfs -put /user/hive/test/titanic /path/to/train.csv
{% endraw %}
{% endhighlight %}

Now start hive, and import with sql query. I'll just create table with first 4 columns in file(there are too many cols here...query becomes too long).

{% highlight sql %}
{% raw %}
hive> CREATE TABLE IF NOT EXISTS titanic(PassengerId INT, Survived INT, Pclass INT, Name STRING) 
    > ROW FORMAT DELIMITED 
    > FIELDS TERMINATED BY ',' 
    > STORED AS TEXTFILE 
    > LOCATION '/user/hive/test/';
OK
Time taken: 0.053 seconds
hive (default)> select * from titanic limit 5;
titanic.passengerid	titanic.survived	titanic.pclass	titanic.name
NULL	NULL	NULL	Name
1	0	3	"Braund	 Mr. Owen Harris"
2	1	1	"Cumings	 Mrs. John Bradley (Florence Briggs Thayer)"
3	1	3	"Heikkinen	 Miss. Laina"
4	1	1	"Futrelle	 Mrs. Jacques Heath (Lily May Peel)"
Time taken: 0.078 seconds, Fetched: 5 row(s)
{% endraw %}
{% endhighlight %}

These are pretty odd because column title are included...but anyway we could check how to add-on csv to hive database.


## For now...
The case I wrote will be pretty different with the product in field of work. It would use streaming service, or else for storing the data instead of porting CSV. There will be lots of ways to tune-up for upgrading performances of Hive, or some of will use other `SQL-on-Hadoop` project(like `Presto`, `Drill`, etc.).
I'll keep working on, and share here as much as possible.


## Reference
* https://hadoop.apache.org/docs/stable/
* https://cwiki.apache.org/confluence/display/Hive/GettingStarted
* https://www.tutorialspoint.com/hive/
* https://princetonits.com/blog/technology/
* https://www.dotnettricks.com/learn/hadoop/
