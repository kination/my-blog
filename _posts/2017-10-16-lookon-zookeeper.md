---
layout: post
title:  "Zookeeper overview - 1"
date:   2017-10-03
tags:
- zookeeper
- open source
permalink: overview-zookeeper-1
description: What for, and how to use Apache zookeeper
---

One of my interests is distribute computing for data research, so it made me looking on open source projects for this feature. One of important point in distribution is managing status of each server, and communicating between them. I found out lots of them(such as `storm`, `kafka`, and much more) are using `zookeeper` for this feature, and this is why I started to looking on it.


### What is zookeeper

Zookeeper is a service for coordinating distributed system. It includes feature for configuration management, synchronization, group services, etc. and these are offered with simple interface to make easy usage for developers. 

![Screenshot](/assets/post_img/zookeeper_overview/zkservice.jpg)

This is simple sketch of Zookeeper managed system. In this diagram, `server` means the machine which zookeeper actually runs, and `client` is the target which are being managed.
Zookeeper service is made up with multiple server for managing each componenets in distribution system. One of them becomes a `leader`, and others are called `follower`. As you see in image, followers service clients and all updates go through order of `follower -> leader -> other followers`. 


### Zookeeper data model

Data in zookeeper is for store coordination data of service, like status information, configuration, location information, and more. It uses a data model styled after the familiar directory tree structure of file systems. 

![Screenshot](/assets/post_img/zookeeper_overview/zknamespace.jpg)

These data nodes are being called `znode`. Each of them are identified with name and seperated by sequence of path. Every znode maintains a stat structure. A stat simply provides the metadata, which has version number, action control list (ACL), timestamp, and data length information.

There are 2 types of node, `Persistence znode` and `Ephemeral znode`. Ephemeral znode are kept alive only while client session is connected, and deleted automatically when connection is lost. On the contrary, persistence znode are persistent regardless of client state unless it has been specified changing state.


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

Configuration for zookeeper service are being set in `zoo.cfg`. If you want to connect multiple server, add server list here as `server.[seq number]=address`. Instead just for test with single laptop, you don't need to add this.

{% highlight shell %}
{% raw %}
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/usr/local/var/run/zookeeper/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
...
# add server list for connection
server.1=192.168.0.1:2888:3888
server.2=192.168.0.2:2888:3888
server.3=192.168.0.3:2888:3888
{% endraw %}
{% endhighlight %}

Now open 2 terminal window and start service.

[1]
{% highlight shell %}
{% raw %}
$ zkServer start-foreground
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
{% endraw %}
{% endhighlight %}

[2]
{% highlight shell %}
{% raw %}
$ zkCli -server 127.0.0.1:2181
Connecting to 127.0.0.1:2181
Welcome to ZooKeeper!
JLine support is enabled
[zk: 127.0.0.1:2181(CONNECTING) 0] 
WATCHER::
{% endraw %}
{% endhighlight %}

This is totally simple!. It tells that it has been connected to zkServer. Now let's see the data model.

{% highlight shell %}
{% raw %}
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: 127.0.0.1:2181(CONNECTED) 2] ls /zookeeper
[quota]
[zk: 127.0.0.1:2181(CONNECTED) 3] ls /zookeeper/quota
[]
{% endraw %}
{% endhighlight %}

There are only one root directory 'zookeeper' and there are one child 'quota'. You can make new ones with `create <path> <data>` command. With `-s` flag, it creates sequential node. To see information of node, use `get <path>`.

{% highlight shell %}
{% raw %}
[zk: 127.0.0.1:2181(CONNECTED) 2] create /new-node node1
Created /new-node
[zk: 127.0.0.1:2181(CONNECTED) 3] get /new-node         
node1
cZxid = 0x1de
ctime = Tue Oct 17 00:40:36 JST 2017
mZxid = 0x1de
mtime = Tue Oct 17 00:40:36 JST 2017
pZxid = 0x1de
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
[zk: 127.0.0.1:2181(CONNECTED) 4] create -s /second-node node2
Created /second-node0000000003
[zk: 127.0.0.1:2181(CONNECTED) 5] get /second-node0000000003
node2
cZxid = 0x1df
ctime = Tue Oct 17 00:43:00 JST 2017
mZxid = 0x1df
mtime = Tue Oct 17 00:43:00 JST 2017
pZxid = 0x1df
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
{% endraw %}
{% endhighlight %}


### Next goal

This is just a starting point of going on zookeeper. As I mentioned, the reason I started to looking on it is to see how this coordination system are being used in lots of distribution systems.


### Reference
* http://zookeeper.apache.org
* https://www.tutorialspoint.com/zookeeper

