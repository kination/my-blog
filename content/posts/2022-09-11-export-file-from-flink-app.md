---
layout: post
title:  Export file(raw, compressed) from Flink application
date:   2022-09-11
description: 
tags:
- bigdata
- flink
- sink
- raw
- gzip
permalink: export-file-from-flink-app
---

`Sink` is a 'loading' part of ETL(Extract, Transformation, Loading) inside Flink. 
It is last process of data pipeline, to store data inside datalake after it has been extract from source, and transformed into specific format.

This is example of how you can sink from Flink `DataStream`:
```java
// ...

private static DataStream<String> createSourceFromStaticConfig(StreamExecutionEnvironment env) {
	Properties inputProperties = new Properties();
	inputProperties.setProperty(ConsumerConfigConstants.AWS_REGION, region);
	inputProperties.setProperty(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");

	return env.addSource(new FlinkKinesisConsumer<>(inputStreamName, new SimpleStringSchema(), inputProperties));
}

public static void main(String[] args) throws Exception {
	// set up the streaming execution environment
	final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

	DataStream<String> input = createSourceFromStaticConfig(env);

	input.addSink(mySinkFunction("s3://data-storage/")); // <- this is for `SinkFunction<T>`
	input.sinkTo(mySinkFunction("s3://data-storage/"));  // <- this is for `FileSink`
}
```

#### FileSink, and StreamingFileSink
If you see the documents [here](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/connectors/datastream/overview/), you can find out there are `StreamingFileSink` and `FileSink`. 

Internally, `StreamingFileSink` is a predecessor of `FileSink`. And in the document it has written that `FileSink` supports `BATCH and STREAMING` both, while `StreamingFileSink` is only for streaming.

And finally from Flink `1.17`, `StreamingFileSink` has been deprecated, so it would be good to go on with `FileSink` from now.

`addSink` function requires `SinkFunction<T>` parameter, and this is for the case when you're trying to use `StreamingFileSink`. Or instead for `FileSink`, call `sinkTo` to add on sink logic.


In this post, I'll just talk about raw file/compressed file sink, which I've worked recently. For big data format file, such as `parquet`, `orc` ..., refer the guide of connectors in [documents](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/connectors/datastream/overview/) .


## Sink out raw text data
First, you need additional dependencies for `FileSink`:
```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-files</artifactId>
    <version>${flink.version}</version>
</dependency> 
```

and here is some configurations needs for file sink(some are optional).

```java
public static void main(String[] args) throws Exception {
	// set up the streaming execution environment
	final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

	DataStream<String> input = createSourceFromStaticConfig(env);

	input.sinkTo(mySinkFunction("s3://data-storage/"));
}

private static FileSink<String> mySinkFunction(String bucket) throws IOException {
    OutputFileConfig outputConfig = OutputFileConfig
            .builder()
            .withPartPrefix("my-data")
            .withPartSuffix(".txt")
            .build();

    return FileSink
            .forRowFormat(new Path(outputPath), new SimpleStringEncoder<String>("UTF-8"))
            .withBucketAssigner(new DateTimeBucketAssigner("yyyy-MM-dd_HH-mm"))
            .withRollingPolicy(
		        DefaultRollingPolicy.builder()
		            .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
		            .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
		            .withMaxPartSize(1024 * 1024 * 1024)
		            .build()
		    )
            .withOutputFileConfig(outputConfig)
            .build();
}

```

In `forRowFormat`, you should define output path, and `Encoder` to serialize individual row data, to output stream. If your data is just raw text data, it will be just fine to use default `SimpleStringEncoder` with `UTF-8` encoded.

`withBucketAssigner` is to define the directory name data to be stored, in specific rule. `DateTimeBucketAssigner` is to generate directory in date-time format. In above, it will put each data inside directory like `/2022-09-10_18-46`

`withRollingPolicy` is to decide the rule, how/when the stream data will be roll-out as output file. In the rule above, single `.txt` file will append the data in stream in following status

- when data has been collected at least 15 minutes
- there are no new elements for 5 minutes
- file size has been reached to 1GB

And with output configuration, file name will be defined as `my-data-????.txt` by defining prefix/suffix.


So finally, output will be located as `s3://data-storage/2022-09-10_18-46/my-data-????.txt`


Some other case, there can be case if user needs to make all elements placed in separate files. File release are related with rolling policy. So in my case, I've make custom policy which extends default `CheckpointRollingPolicy`

```java
private static FileSink<String> mySinkFunction(String bucket) throws IOException {
    OutputFileConfig outputConfig = OutputFileConfig
            .builder()
            .withPartPrefix("my-data")
            .withPartSuffix(".txt")
            .build();

    return FileSink
            .forRowFormat(new Path(outputPath), new SimpleStringEncoder<String>("UTF-8"))
            .withBucketAssigner(new DateTimeBucketAssigner("yyyy-MM-dd_HH-mm"))
            .withRollingPolicy(new CustomCheckpointRollingPolicy())
            .withOutputFileConfig(outputConfig)
            .build();
}
...
static final class CustomCheckpointRollingPolicy<IN, BucketID> extends CheckpointRollingPolicy<IN, BucketID> {
    private static final long serialVersionUID = 1L;

    CustomCheckpointRollingPolicy() {}

    public boolean shouldRollOnEvent(PartFileInfo<BucketID> partFileState, IN element) {
        return true;
    }

    public boolean shouldRollOnProcessingTime(PartFileInfo<BucketID> partFileState, long currentTime) {
        return false;
    }

}

```

It has been defined `shouldRollOnEvent` to always return true, so it will roll out for every elemnts. In this case, every 'string' data from `DataStream` will be generated as separated text file.


## Compress('.gzip') output sink file
Now let's think about when you want to sink data as compressed file(like `gzip`). In most of case you would want in this way, to reduce file size, network traffic. As you know, these are all related with cost.

For compressed format, you need to use bulk format, with `BulkWriter` which can be defined with base `BulkWriter.Factory`. This is not just for compressed file such as `zip, gzip`, but also for big data formats like `parquet, avro, orc`.

[Bulk encoded formats](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/connectors/datastream/file_sink/#bulk-encoded-formats)

You can find several built-in writer here. And for `gzip`, it needs to use `CompressWriterFactory`.

```java
private static FileSink<String> mySinkFunction(String bucket) throws IOException {
    OutputFileConfig outputConfig = OutputFileConfig
            .builder()
            .withPartPrefix("my-data")
            .withPartSuffix(".gzip")
            .build();

    return FileSink
            .forBulkFormat(
            	new Path(outputPath),
            	CompressWriters.forExtractor(new DefaultExtractor()).withHadoopCompression("GzipCodec")
            )
            .withBucketAssigner(new DateTimeBucketAssigner("yyyy-MM-dd_HH-mm"))
            .withRollingPolicy(new CustomCheckpointRollingPolicy())
            .withOutputFileConfig(outputConfig)
            .build();
}
```

`CompressWriters` are builder for creating `CompressWriterFactory` instance, and `DefaultExtractor` is to turn record into byte array for writing data. This transformed byte array data can be compressed with following hadoop compression codec, by `withHadoopCompression`.

- DEFLATE: org.apache.hadoop.io.compress.DefaultCodec
- gzip: org.apache.hadoop.io.compress.GzipCodec
- bzip2: org.apache.hadoop.io.compress.BZip2Codec
- LZO: com.hadoop.compression.lzo.LzopCodec

Okay, now it's done in code. But it could cause exception as following,

```
java.util.concurrent.CompletionException: 
java.lang.NoClassDefFoundError: org/apache/hadoop/conf/Configuration\n\tat java.base
java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:314)
java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:319)
java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1702)
java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
java.base/java.lang.Thread.run(Thread.java:829)
Caused by: java.lang.NoClassDefFoundError: org/apache/hadoop/conf/Configuration
org.apache.flink.formats.compress.CompressWriterFactory.withHadoopCompression(CompressWriterFactory.java:76)
```

and this means it cannot find class definition for hadoop file system. In this case you should add following package:
```xml
<groupId>org.apache.flink</groupId>
<artifactId>flink-compress</artifactId>
<version>${flink.version}</version>
...
<groupId>org.apache.flink</groupId>
<artifactId>flink-s3-fs-hadoop</artifactId>
<version>${flink.version}</version>
```


## Reference
- <https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/connectors/datastream/overview/>
- <https://stackoverflow.com/questions/>
- <https://docs.aws.amazon.com/kinesisanalytics/latest/java/how-sinks.html>
- <https://www.oreilly.com/library/view/hadoop-the-definitive/9780596521974/ch04.html>


