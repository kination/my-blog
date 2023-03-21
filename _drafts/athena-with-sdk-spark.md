---
layout: post
title: Athena with Scala and Spark
date:  2020-10-21 
description: 
tags:
- scala
- spark
- aws
- athena
- sdk
- dataframe
permalink: 

---


## Athena
Amazon Athena is a service to analyze data stored in S3 with interactive SQL query. You don't need additional setup, and only need to pay for the queies you've run.

It is usually being used by data scientists, or business developer who needs to get insights from big data.

Also for data engineers, it can be needed to get specific data to construct data process logic. And in this case, they will create application to automate this kind of job. Of course, Amazon is offering SDK for the developers.


## Using official SDK(1.x)
First, this example is based on 1.x version. Still 2.x version seems in early stage, so I've worked on this. If you're interest on ...

```scala
libraryDependencies ++= Seq(
  ...
  "com.amazonaws" % "aws-java-sdk-athena" % "1.11.832",
)
```

```scala
val athenaClient = AmazonAthenaClientBuilder
      .standard()
      .withRegion(Regions.US_WEST_1)
      .withCredentials(InstanceProfileCredentialsProvider.getInstance())
      .build()
```

```

```




## With JDBC driver




## Reference
* <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/examples-athena.html>
* <https://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html>
* <https://www.simba.com/products/Athena/doc/JDBC_InstallGuide/content/jdbc/using/intro.htm>


