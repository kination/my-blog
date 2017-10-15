---
layout: post
title:  "Zookeeper overview"
date:   2017-10-03
tags:
- zookeeper
- open source
permalink: overview-zookeeper
description: What for, and how to use Apache zookeeper
---

One of my interests is distribute computing for data research, so it made me looking on open source projects for this feature. One of important point in distribution is managing status of each server, and communicating between them. I found out lots of them are using `zookeeper` for this feature, and this is why I started to looking on it.


### What is zookeeper

Zookeeper is a service for coordinating distributed system. It includes feature for configuration management, synchronization, group services, etc. and these are offered with simple interface to make easy usage for developers. 

![Screenshot](/assets/post_img/zookeeper_overview/zkservice.jpg)

This is simple sketch of Zookeeper managed system.
Zookeeper service is made up with multiple server for managing each componenets in distribution system. One of them becomes a `leader`, and others are called `follower`. As you see in image, followers service clients and all updates go through order of `follower -> leader -> other followers`. 


### Zookeeper data model

Data in zookeeper is for store coordination data of service, like status information, configuration, location information, and more. It uses a data model styled after the familiar directory tree structure of file systems. 

![Screenshot](/assets/post_img/zookeeper_overview/zknamespace.jpg)

Each of data node are being called `znode`.



### Start zookeeper

Start with install zookeeper. You can download from [release page](http://zookeeper.apache.org/releases.html) or use `brew` if using Mac. Use `zkServer` command for zookeeper.

{% highlight shell %}
{% raw %}
$ brew install zookeeper
...
// after install
$ zkServer 
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
{% endraw %}
{% endhighlight %}




### Reference
* http://zookeeper.apache.org

