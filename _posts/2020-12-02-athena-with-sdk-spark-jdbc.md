---
layout: post
title: Athena with Scala and Spark
date:  2020-12-02
description: 
tags:
- scala
- spark
- aws
- athena
- sdk
- dataframe
- jdbc
permalink: athena-with-sdk-spark-jdbc

---

Amazon Athena is a service to analyze data stored in S3 with interactive SQL query. You don't need additional setup, and only need to pay for the queies you've run.

It is usually being used by data scientists, or business developer who needs to get insights from big data(probably stored inside S3, or else).

Also for data engineers, it can be needed to get specific data to construct data process logic. And in this case, they may need to create application to automate querying job. Of course, Amazon is offering SDK for the developers.

This is a note of process to receive/treat data executed by Athena query, inside data pipeline.


## Using official SDK(2.x)
Let's see SDK for Athena first. Newest version for AWS is `2.x`.

```scala
libraryDependencies ++= Seq(
  ...
  "software.amazon.awssdk" % "athena" % "2.15.79"
)
```

First it requires to build client:
```scala
import software.amazon.awssdk.auth.credentials.InstanceProfileCredentialsProvider
import software.amazon.awssdk.regions.Region
import software.amazon.awssdk.services.athena.AthenaClient

val client: AthenaClient = AthenaClient
  .builder
  .region(Region.US_WEST_1)
  .credentialsProvider(InstanceProfileCredentialsProvider.create())
  .build
```

Now query can be submitted with client, but there are several configuration to be done:
```scala
import software.amazon.awssdk.services.athena.model.{
  QueryExecutionContext, 
  ResultConfiguration, 
  StartQueryExecutionRequest
}
// ...
val QUERY_STRING = "SELECT * FROM user ORDER BY created_at DESC LIMIT 10"
val OUTPUT_PATH = "s3://output/athena"

val queryExecutionContext = QueryExecutionContext.builder.database("athena_project").build
val resultConfiguration = ResultConfiguration.builder.outputLocation(OUTPUT_PATH).build

val startQueryExecutionRequest = StartQueryExecutionRequest
  .builder
  .queryString(QUERY_STRING)
  .queryExecutionContext(queryExecutionContext)
  .resultConfiguration(resultConfiguration)
  .build

```

This is to submit 'newest 10 row inside `user` dataset of database `athena_project`.

Athena SDK requires user to define 'output location' to store result data in S3. This can be useless if you just want to check the result, but it is a requirements, and it will return error when not defined with `ResultConfiguration.builder.outputLocation`.

Query execution can take time depend on database size or complication of query, so need to add wait loop logic.

```scala
// get id from 'startQueryExecutionRequest' above
val queryExecutionId = startQueryExecutionResponse.queryExecutionId()

val getQueryExecutionRequest = GetQueryExecutionRequest.builder.queryExecutionId(queryExecutionId).build
var getQueryExecutionResponse = null
var isQueryStillRunning = true

while (isQueryStillRunning) {
  getQueryExecutionResponse = athenaClient.getQueryExecution(getQueryExecutionRequest)
  val queryState = getQueryExecutionResponse.queryExecution.status.state

  queryState match {
    case QueryExecutionState.FAILED => throw new RuntimeException("The Amazon Athena query failed to run with error message: " + getQueryExecutionResponse.queryExecution.status.stateChangeReason)
    case QueryExecutionState.CANCELLED => throw new RuntimeException("The Amazon Athena query was cancelled.")
    case QueryExecutionState.SUCCEEDED => isQueryStillRunning = false
    case _ => Thread.sleep(1000)
  }

  logger.info("The current status is: " + queryState)
}
```

It will execute query execution state every 1 second until state is success or failed(cancel), and move on to next step if query execution has done successfully.

If process is going well until here, you can find the result by `GetQueryResultsRequest`, using query execution ID.

```scala
val getQueryResultsRequest = GetQueryResultsRequest.builder.queryExecutionId(queryExecutionId).build
val getQueryResultsResults = athenaClient.getQueryResultsPaginator(getQueryResultsRequest)
```


Now you can see the result through this.

But if this process is inside data pipeline with big data processing engine(such as `Spark`), you might need to load the result inside spark dataframe. In this case, you can read the result using output CSV file(which defined above).

```scala
spark.read.option("header", "true").csv(OUTPUT_PATH)
```

But this `writing to file + read into dataframe` seems causing unnecessary process, because actually it don't need to write file if it can be read directly to dataframe.

In this case, you can use JDBC driver for this approach.


## With JDBC driver

You should download JDBC driver from Athena official website.

https://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html

One bad story is, driver has not been uploaded into maven repository about 3 years, so several features will not work. You should put jdbc into `/lib` directory inside project.

(For example `Catalog` option will not work in maven repo version)

```scala
val driver = "com.simba.athena.jdbc.Driver"
Class.forName(driver)

val connector: String = Seq(
  s"jdbc:awsathena://MetadataRetrievalMethod=ProxyAPI",
  s"AwsRegion=us-west-1",
  s"S3OutputLocation=$OUTPUT_PATH",
  s"AwsCredentialsProviderClass=${credential-provider}",
  s"Catalog=${your-catalog}",
  s"Schema=${your-schema}"
).mkString(";")

spark.read
  .option("driver", driver)
  .option("url", connector)
  .option("query", QUERY_STRING)
  .format("jdbc")
  .load()
```

Detail for connector url can be found [here](https://www.simba.com/products/Athena/doc/JDBC_InstallGuide/content/jdbc/ath/using/connectionurl.htm)

If you need to setup `credential-provider`, you could refer [this](https://www.simba.com/products/Athena/doc/JDBC_InstallGuide/content/jdbc/ath/options/intro-auth.htm).



## Reference
* <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/examples-athena.html>
* <https://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html>
* <https://www.simba.com/products/Athena/doc/JDBC_InstallGuide/content/jdbc/using/intro.htm>


