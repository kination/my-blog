---
layout: post
title:  Several ways to handle multiple tasks in Scala
date:   2020-08-27
description: 
tags:
- scala
- task
- queue
- listener
- producer
- consumer
permalink: handling-tasks-on-scala
---


## Requirements
Let's think there are some application running based on Spark, and need to get DataFrame which has specific name in `username`, like: 

```scala
val df = spark.read.parquet("s3://data/user.parquet")
...
def getUserByName(name: String) = 
  df.filter(df("username") == name)
  
...
```

Actually, spark is very fast, so it will be just okay to use it directly in most of cases. But let's assume that this application are facing on huge traffic, so need to separate the runner by thread.

One thing we can think is to use queue, which keeps inputs to be handled until application is ready. Yes, we can think of some Queue systems like Kafka, RabbitMQ, and more. But here, I will think of ways to handled by application itself, by making own queue to receive inputs.

If this needs to be handled on production, I'll not recommend this(please, don't take hard way on work). This is just primitive fashion way, to think about how logic goes on.


## Separate thread, as observer
It should be most simple way, comparing with others.


```scala
private val listenerExecutor = new ScheduledThreadPoolExecutor(1)
private val taskQueue: ConcurrentLinkedQueue[String] = new ConcurrentLinkedQueue[String]()

val tasks: Runnable = () => {
  if (!taskQueue.isEmpty()) {
    val persistPath: String = taskQueue.poll()

    if (persistPath.equals("end")) {
      
    }
    else {

    }
  }
}

listenerExecutor.scheduleAtFixedRate(tasks, 0L, 200, TimeUnit.MILLISECONDS)

taskQueue.offer("")


```


## Extend queue to setup listener

```scala
abstract class Listener { def runner }

case class TaskQueue[E]() extends ConcurrentLinkedQueue[E] {

  private var listener: Listener = null

  def addListener(listener: Listener): Unit = this.listener = listener

  override def iterator(): Iterator[E] = super.iterator()

  override def size(): Int = super.size()

  override def poll(): E = super.poll()

  override def peek(): Nothing = ???

  override def offer(e: E): Boolean = {
    super.offer(e)
    listener.runner
    true
  }
}
```


```scala
val tq: TaskQueue[Int] = new TaskQueue[Int]

tq.addListener(new Listener {
  override def trigger: Unit = println("triggered!: " + tq.poll())
})

```


## Pub/Sub pattern




## 

