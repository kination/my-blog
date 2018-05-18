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
{% highlight shell %}
{% raw %}
$ cd /usr/local
$ wget http://mirror.apache-kr.org/hadoop/common/hadoop-2.8.4//hadoop-2.8.4.tar.gz
$ tar xvf hadoop-2.8.4.tar.gz
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


Now start with command in `sbin` directory. We will try HDFS only, not YARN here.
{% highlight shell %}
{% raw %}
$ ./sbin/start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop-2.8.4/logs/hadoop-inylove82-namenode-cs-6000-devshell-vm-45621d49-d12f-4108-a6f7-0d82efca5420
.out
localhost: starting datanode, logging to /usr/local/hadoop-2.8.4/logs/hadoop-inylove82-datanode-cs-6000-devshell-vm-45621d49-d12f-4108-a6f7-0d82efca5420
.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.8.4/logs/hadoop-inylove82-secondarynamenode-cs-6000-devshell-vm-45621d49-d12f-4108-a
6f7-0d82efca5420.out
{% endraw %}
{% endhighlight %}

If you try to run this command via `sudo`, (in my case)it causes permission issue, so don't use it.

{% highlight shell %}
{% raw %}

{% endraw %}
{% endhighlight %}


## Setup Hive



{% highlight shell %}
{% raw %}
$ cd /usr/local
$ wget wget http://mirror.apache-kr.org/hive/hive-2.3.3/apache-hive-2.3.3-bin.tar.gz
$ tar xvf apache-hive-2.3.3-bin.tar.gz
{% endraw %}
{% endhighlight %}


Hive needs to know the path of Hadoop installed, so define it in environment file in `conf/`
{% highlight shell %}
{% raw %}
$ cd conf/
$ mv hive-env.sh.template hive-env.sh
$ vi hive-env.sh
{% endraw %}
{% endhighlight %}

{% highlight shell %}
{% raw %}
...
HADOOP_HOME=/usr/local/hadoop-2.8.4
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



