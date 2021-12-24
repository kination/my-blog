---
layout: post
title:  
date:   2021-09-03
description: 
tags:
- scala
- spark
- skew
- spark3
permalink: 
---

`Data Skewness` is a one of issue you can face oftenly while treating spark which is caused based on parallel computing. 

Following image simply described the situation of `skewed`

![1*SICi8EJBHIpWzeQvBb1Jog](https://user-images.githubusercontent.com/1720209/147326734-510090e3-f2e0-42d6-9042-e43b9cdb0c41.png)


## Why it happens
Spark is distributed computing system, and it distributes the data separately inside cluster, to make data processing. So if data has been spread unevenly, processing will be concentrated in few specific machine which receive heavy data load. 

It makes cluster unefficiently, and causes bad performance of system.


One main reason of this cause, is by calling `join`, `groupBy`, which makes data transformation to change data partitioning.

If you see the diagram,
![skew:https://coxautomotivedatasolutions.github.io](https://user-images.githubusercontent.com/1720209/147329532-1957da82-19fa-4347-8f19-8ca6fc930e02.jpg)

this is caused by running logic below:
```scala
t1.join(t2, Seq("make", "model"))
  .filter(abs(t2("engine_size") - t1("engine_size")) <= BigDecimal("0.1"))
  .groupBy("registration")
  .agg(avg("sale_price").as("average_price")
```

On `join` process, data will be repartitioned with join criteria `make`/`model`. But because value of the criteria is skewed, distribution will be held up unequaly, like the diagram.

## Way to solve 
#### 



#### Salting



#### Broadcase join
You can use function `broadcast` to spr


```spark
val orgTable = spark.range(1, 10000)

val rows = sc.parallelize(List(
  Row(1, "gold"),
  Row(2, "silver"),
  Row(3, "bronze")
))

val rowsSchema = StructType(Array(
  StructField("id", IntegerType),
  StructField("medal", StringType)
))

val lookupTable: DataFrame = spark.createDataFrame(rows, rowsSchema)
```


```scala
val joined = orgTable.join(lookupTable, "id")
```

```scala
joined.explain()
```


## New features for skew in spark 3





## Reference
* <http://spark.apache.org/>
* <https://coxautomotivedatasolutions.github.io/datadriven/spark/data%20skew/joins/data_skew/>
* <https://itnext.io/handling-data-skew-in-apache-spark-9f56343e58e8>
* <https://blog.rockthejvm.com/spark-broadcast-joins/>
* <https://blog.cloudera.com/how-does-apache-spark-3-0-increase-the-performance-of-your-sql-workloads/>
* https://www.clairvoyant.ai/blog/optimizing-the-skew-in-spark
* https://gunju-ko.github.io/spark/2020/11/14/HandlingDataSkew.html
* https://pizzathief.oopy.io/dealing-with-spark-data-skew
