---
layout: post
title:  "Actor model, and first work with Akka"
date:   2018-03-28
description: 
tags:
- actor
- akka
- java
- scala
permalink: actor-model-and-akka
---

First time I heard of Akka, was the blog post written about case of Twitter, that they implemented their service based on this to handle massive twit data in real-time. I've had some experiences developing web server, and currently having interests of concurrency issue, and it makes me looking on this.

## Actor Model?
As you could see in other documents, theory of Actor has been more than 40 years. But just before 15 years ago, we usually did not need to think about concurrency or multi-threading deeply, because there were no smartphones which causes heavy requests. Now, massive internet services like Facebook and Twitter needs to handle millions of requests per minute, and so they are finding/developing platform to satisfy this.

Actor is an object, composed with behavior, state, mailbox. Behavior is the job which Actor need to do, state is the current state of Actor, and mailbox is to send/receive the message from other Actors. 

![Screenshot](/assets/post_img/actor-model-and-akka/actor-diagram.png)

Yes, this has some similarity with `Thread`. In programming we could use these to run process asyncrounously. The point which makes it different is, each of them cannot access to other actor's memory or state. Each of them are only doing their job without sharing other's area and every communication is being done by mailbox.
As there is no sharing resources, there is no state like `lock` in `Thread` because it does not need to wait for waiting memory from others. It just sends the message without blocking so you can get benefits of multi-thread programming without worring about the problems you can face with 'sharing' issue.


## Akka
`Akka` is a library which helps to implement actor model to your system. It is OSS(Open Source Software), and currently managed by `Typesafe`. Most of open source actor model implementations based on Java/Scala, are based on this, and I'll try to use it here. Core Akka is implemented in Scala, but they also offer Java SDK for development.

Before creating `Hello world` stuff, let’s check the relationships between actors you create in your code and what happens below the modules.

An actor in Akka always belongs to a parent. If you generate new actor in your code, it will be created as child of `/user` actor, which are being created by system.
When you setup the Akka library and start the code, it will create three actors including `/user`. The names of these built-in actors contain guardian because they supervise every child actor in their path.

![Screenshot](/assets/post_img/actor-model-and-akka/actor-top-tree.png)

- The one in top is root guardian. It is the parent of the all actors in system, and will be removed after system is destroyed.
- `/user` is parent actor for all user created actors. Every actor you create will have the path '/user/' prepended to it.
- `/system` the system guardian.



## Akka implementations



## 


## Reference

https://doc.akka.io




