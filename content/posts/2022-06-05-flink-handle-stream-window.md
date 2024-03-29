---
layout: post
title:  How streams are being handled in Flink, with window
date:   2022-06-05
description: 
tags:
- bigdata
- flink
- window
- stream
- java
permalink: flink-handle-stream-window
---

Data stream is a flow of data, which are coming from multiple sources continuously. It can be event log or database log captured by CDC internally, or sensor data from IoT devices.

These kind of streams are usually very high-frequent in production level, and in this case we will think of connecting sources into stream processing framework to handle it, such as Spark or Flink.

Because I've started using Flink recently, so try to look on how this is being treated.

![](/assets/post_img/flink-handle-stream-window/flink-application-sources-sinks.png)


## What is Flink
Flink is distributed data processing framework for stateful computations over unbounded and bounded data streams. In the point of 'distribute data processing framework' it looks similar with Apache Spark. Actually both has lots in common, and are being widely used in data engineering field. If you're looking on job description for data engineering position, you could see at least one of these(or both) in requirements.

One of big difference is that Flink is implemented as to be true streaming engine, while Spark is handling streaming data with 'micro-batch' engine. Though it looks like streaming, there are difference under the hood.

![](/assets/post_img/flink-handle-stream-window/streaming-bounded-unbounded.png)

To treat bounded/unbounded streaming(which means whether the end of streaming is defined or not), there are lots of logics implemented in Flink, to let developers handles streaming data safe and stable.


## How Flink handles stream data - basics
After receiving stream, we probably want to transform the data to specific format, and send outside to the target. In Flink, it offers very expressive window semantics, with `window` function.

'Window' are core of processing streams. It defines how to splits infinite stream into finite size, so it can apply computation.

Check the general structure below.
```java
...
DataStream<T> inputStream = ...;

// 1. Keyed
inputStream
       .keyBy(...)               //  keyed versus non-keyed windows
       .window(...)              //  required: "assigner"
      // ... and more operator

// 2. Non-Keyed
inputStream
       .window(...)              //  required: "assigner"
       // ...  and more operator
```

Generated stream variable `inputStream`, is stream of `T` instances. And by `window assigner`, stream will be splited into defined size.

After splitted, it can be computed with variable functions offered by Flink. Also there are several more operators(`trigger`, `evictor`, ...) to control data flow which is optional. But here, I've just noted about how data inside stream are being assigned into separated window.


## Keyed / Non-Keyed
On the code, you can see difference between 2 structure, that one of them has `keyBy` function. 

'Keyed' stream, will allow stream computation to be performed in parallel, and each logical keyed data will be processed separately, with data which have different key. More simply, you can think key as some kind of 'partition' of data processing.

```java
class User {
      String name;
      int id;
      int groupId;
}

DataStream<User> userStream = ...;

userStream
      .keyBy(User::groupId)
      .window(...);
```

So in this code, stream will be internally partitioned by `groupId`.


## Window Assigners
'Window Assigners' are the features which goes inside `window(...)` operator. It is to define how elements in stream will be 'assigned' in windows.

![](/assets/post_img/flink-handle-stream-window/window-assigners.svg)

### Tumbling windows
'Tumbling windows' assigners is to assign input elements to window of specified window size. In this, size means time range(event time, or processing time).

```java
DataStream<T> inputStream = ...;

// define window size as 5 second range of event time
inputStream
      .keyBy(...)
      .window(TumblingEventTimeWindows.of(Time.seconds(5)));

// define window size as 1 minute range of processing time
inputStream
      .keyBy(...)
      .window(TumblingProcessingTimeWindows.of(Time.minutes(1)));


// can input offset parameter as option.
// 1 day range(event time) of window, with -9 hour
inputStream
      .keyBy(...)
      .window(TumblingEventTimeWindows.of(Time.days(1), Time.hours(-9)));     
```


### Sliding windows
'Sliding windows' also assign inputs to range-defined window, but it needs to define how frequently start new window. So as you can see in diagram above, there can be overlapped part between windows(if frequency is smaller than window size). So single input can be assigned to multiple windows.

```java
DataStream<T> inputStream = ...;

// window size is 10 second of event time, and create new one every 5 second
inputStream
      .keyBy(...)
      .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)));

// window size is 1 minute of processing time, and create new one every 50 second
inputStream
      .keyBy(...)
      .window(SlidingProcessingTimeWindows.of(Time.minutes(1), Time.seconds(50)));
```


### Session windows
'Session windows' groups inputs by sessions of activity. It opens window when input has been received, and close if there's no input in defined time range. So there are no fixed start/end time.

```java
DataStream<T> inputStream = ...;

// window is kept opened until there are no inputs on 10 minutes
inputStream
      .keyBy(...)
      .window(EventTimeSessionWindows.withGap(Time.minutes(10)));

// event-time session windows with dynamic gap
inputStream
      .keyBy(...)
      .window(EventTimeSessionWindows.withDynamicGap((element) -> {
        // determine and return session gap
    }));
```

### Global windows
'Global windows' assigns all elements with the same key to the same single global window. This is only useful when specifiying custom trigger. Otherwise, no computation will be performed.

```java
DataStream<T> inputStream = ...;

inputStream
      .keyBy(...)
      .window(GlobalWindows.create());
```


## Reference
- https://nightlies.apache.org/flink/flink-docs-master/

