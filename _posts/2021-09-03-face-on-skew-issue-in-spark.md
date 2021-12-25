---
layout: post
title:  Optimizing spark pipeline from data skew
date:   2021-09-03
description: 
tags:
- scala
- spark
- parallel
- distribution
- skew
permalink: optimizing-spark-pipeline-from-data-skew
---

'Data Skewness' is a one of issue you can face oftenly while treating spark which is caused based on parallel computing. 

Following image simply described the situation of 'skewed'.

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

### Add key to improve join
If we can expect the status of data, we can select the way to add column for `join`, which has equity can make distribution more equity.

Based on this table, we can including engine_size in join condition to achieve the desired result -> every original record from t1 will be joined to every record from t2 with the same make, model, and engine size +/- 0.1. To acheive this we can make modified column, using `explode`.

Spark function `explode`, is to split the array value by separate rows. For example,
```scala
scala> df.show(false)
+-------+---------------------+
|name   |knownLanguages       |
+-------+---------------------+
|James  |["Java","Scala"]     |
|Michael|["Spark","Java",null]|
|Robert |["CSharp",""]        |
+-------+---------------------+
```

assume there are dataframe like following. And by explode, it becomes:
```scala
scala> df.select($"name",explode($"knownLanguages"))
  .show(false)
+-------+------+
|name   |col   |
+-------+------+
|James  |Java  |
|James  |Scala |
|Michael|Spark |
|Michael|Java  |
|Michael|null  |
|Robert |CSharp|
|Robert |      |
+-------+------+
```

So, by making new column `engine_size`, and split the row by filter above:
```scala
t1.withColumn("engine_size", explode(array($"engine_size" - BigDecimal("0.1"), $"engine_size", $"engine_size" + BigDecimal("0.1"))))
  .join(t2, Seq("make", "model", "engine_size"))
  .groupBy("registration")
  .agg(avg("sale_price").as("average_price"))
```

repartitioning will be held equally.
![skew:https://coxautomotivedatasolutions.github.io](https://user-images.githubusercontent.com/1720209/147382429-575fddc6-1779-4ce3-a65e-51e4b5a6db6c.jpeg)


### Salting
Or if it is difficult to indicate the equity status of table data, you can make some random data to table, and add in condition to make partitioning evenly. Shortly, it means to add noise data.

Assume there are some logic like this:
```scala
df.groupBy("address", "type")
```

make random value, and setup in `groupBy`.
```scala
val salt = random(0, 100) - 1)
df.withColumn("salt", lit(salt))
  .groupBy("address", "type", "salt")
  .drop("salt")
```

To make more efficiently, random value should be in range `0 ~ partition count - 1`.


### Broadcase join
In other way, you can use function `broadcast` to spread join event to be done distributedly.

```scala
scala> orgTable.show(false)
+-------+
|id     |
+-------+
|1      |
|2      |
|3      |
|...    |
|...    |
|9999   |
|10000  |
+-------+

scala> newTable.show(false)
+-----+-------+
|id   |medal  |
+-----+-------+
|1    |gold   |
|2    |silver |
|3    |bronze |
+-----+-------+
```

With 2 table above, you can join these with `id`.

```scala
val joined = orgTable.join(newTable, "id")
```

Actually there are no problem on result, but there are in performance.

```scala
joined.explain()

== Physical Plan ==
*(5) Project [id#259L, medal#264]
+- *(5) SortMergeJoin [id#259L], [cast(id#263 as bigint)], Inner
   :- *(2) Sort [id#259L ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#259L, 200)
   :     +- *(1) Range (1, 10000, step=1, splits=6)
   +- *(4) Sort [cast(id#263 as bigint) ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(cast(id#263 as bigint), 200)
         +- *(3) Filter isnotnull(id#263)
            +- Scan ExistingRDD[id#263,order#264]
```
In physical plan, you can find 'SortMergeJoin' on `id`, which includes the logic following:
1. Sorting `orgTable`
2. Suffle `orgTable`
and 'Suffling' is very expensive operation for performance.


Simply, just wrap small table with `broadcast`:
```scala
val newJoined = orgTable.join(broadcast(newTable), "id")
```

you can find improved plan.
```scala
newJoined.explain()

== Physical Plan ==
*(2) Project [id#294L, order#299]
+- *(2) BroadcastHashJoin [id#294L], [cast(id#298 as bigint)], Inner, BuildRight
   :- *(2) Range (1, 10000, step=1, splits=6)
   +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint)))
      +- *(1) Filter isnotnull(id#298)
         +- Scan ExistingRDD[id#298,order#299]
```

There are no shuffling in `orgTable`, and `BroadcastExchange` is held for `newTable`, and it will only take small cost hence this is small table.


## Reference
* <http://spark.apache.org/>
* <https://coxautomotivedatasolutions.github.io/datadriven/spark/data%20skew/joins/data_skew/>
* <https://itnext.io/handling-data-skew-in-apache-spark-9f56343e58e8>
* <https://blog.rockthejvm.com/spark-broadcast-joins/>
* <https://sparkbyexamples.com/>
