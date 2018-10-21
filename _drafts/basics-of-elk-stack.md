---
layout: post
title:  Basics of ELK stack
date:   2018-10-21
description: 
tags:
- elasticsearch
- kibana
- logstash
permalink: basics-of-elk-stack
---

Nowadays, most of the service requires massive size of database. 
For example, let's think you are making e-commerce service. In early days, it only needed to save data for items(item image, description...) and user(ID, email, address...) which are necessary for commerce. But, to attract more user, it started to offer more personalized information, such as 'recommend items'. To investigate user likes, they need to research more detail user pattern/logs , and database increased explosively.


## Elasticsearch, and ELK stack
Elasticsearch is search/analytics engine based on Lucene. It provides a distributed, multitenant-capable full-text search engine with HTTP REST web interface. In official guide, it saids:
```
Elasticsearch is a highly scalable open-source full-text search and analytics engine. It allows you to store, search, and analyze big volumes of data in near real time...
```
Most important point is, it is very fast, much more than searching from RDBMS using SQL query.
In case of e-commerece service explained above, it can be used to store full catalog, and send auto-complete result to user quickly.


ELK stack means `Elasticsearch`, `Logstash`, and `Kibana`. It first started with Elasticsearch, and others are added later ...


## Start ES

You can download elasticsearch from [official site](https://www.elastic.co/downloads/elasticsearch), or by `brew` if you are MacOS user.
```
$ brew install elasticsearch
```

After install, you could start engine by command `elasticsearch`.

```
$ $ elasticsearch
Java HotSpot(TM) 64-Bit Server VM warning: Cannot open file logs/gc.log due to No such file or directory

[2018-10-21T22:32:45,707][INFO ][o.e.n.Node               ] [] initializing ...
[2018-10-21T22:32:45,800][INFO ][o.e.e.NodeEnvironment    ] [XwChsFl] using [1] data paths
...
...
[2018-10-21T22:32:56,772][INFO ][o.e.n.Node               ] [XwChsFl] started
[2018-10-21T22:32:56,784][INFO ][o.e.g.GatewayService     ] [XwChsFl] recovered [0] indices into cluster_state
[2018-10-21T22:33:26,751][INFO ][o.e.c.r.a.DiskThresholdMonitor] [XwChsFl] low disk watermark [85%] exceeded on [XwChsFluRIGjqONsrmaPuA][XwChsFl][/usr/local/var/elasticsearch/nodes/0] free: 27.5gb[11.5%], replicas will not be assigned to this node
```

As mentioned above, it offers HTTP interface to access data. Default access point is `localhost:9200`, so let's try some simple access to check it runs well.

```
$ curl -X GET localhost:9200
{
  "name" : "XwChsFl",
  "cluster_name" : "elasticsearch_user",
  "cluster_uuid" : "_BvcZeP5RkW9W9Gix726Dw",
  "version" : {
    "number" : "6.4.2",
    "build_flavor" : "oss",
    "build_type" : "tar",
    "build_hash" : "04711c2",
    "build_date" : "2018-09-26T13:34:09.098244Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

Okay, now setup is ready.
