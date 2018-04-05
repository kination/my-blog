---
layout: post
title:  "Actor model, and Akka"
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

...
Important point of Actor model is, each of them does not share memory or state. Each of them are only doing their job without interfering other, and this makes the difference with using thread for concurrency implementation.




