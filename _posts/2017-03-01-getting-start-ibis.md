---
layout: post
title: "Looking on Ibis, python data analysis framework for Hadoop components"
date: 2017-03-14 20:35:48
image: '/assets/img/'
description: 'What is, and how to go on with Ibis'
main-class: 'jekyll'
tags:
- python
- open source
- data
- ibis
categories:
---

Ibis is a platform(or toolbox) implemented by Cloudera, which helps connecting remote storage and local python codes. The newest version is 0.8, and currently supports Hadoop components(HDFS, Impala, Kudu) and SQL DBs(SQLite, PostgreSQL). 
Main purpose of this is to simplify analytical workflows with remote data. For this goal, there are lots of useful APIs for data analysis. If you are friendly with python Pandas, you could find familiar expression with it.

## Expression

Before working with remote stuffs, you can test with local SQLite db to get the hang of it.
First, download data file `Crunchbase` from [here](http://blog.ibis-project.org/pages/data.html), and make python code.
{% highlight python %}
{% raw %}
import ibis


connect = ibis.sqlite.connect('crunchbase.db')
tables = connect.list_tables()
{% endraw %}
{% endhighlight %}

You can open SQLite DB to use on Ibis simply by `sqlite.connect`. Call `list_tables()` method to list up the tables which are in DB file. There would be `['acquisitions', 'companies', 'investments', 'rounds']` in sample.

Let's see what is in table `companies`.
{% highlight python %}
{% raw %}
companies = con.table('companies')
companies.info()
{% endraw %}
{% endhighlight %}

You will see something like this.
{% highlight shell %}
{% raw %}
$ Table rows: 54292

Column             Type    Non-null #
------             ----    ----------
permalink          string  51166     
name               string  51166     
homepage_url       string  47583     
category_list      string  47826     
market             string  46416     
funding_total_usd  float   42237
...
{% endraw %}
{% endhighlight %}

If you want to see what is in `funding_total_usd` column, go on like this.

{% highlight python %}
{% raw %}
companies = con.table('companies')
companies.funding_total_usd.value_counts()
{% endraw %}
{% endhighlight %}

{% highlight shell %}
{% raw %}
      funding_total_usd  count
0                   NaN  12055
1                   1.0      1
2                   2.0      1
3                   5.0      1
...
9998         10024049.0      1
9999         10025000.0      3

[10000 rows x 2 columns]
{% endraw %}
{% endhighlight %}

If you just want to see mean value, it also can be known simply.

{% highlight python %}
{% raw %}
companies = con.table('companies')
mean_value = companies.funding_total_usd.mean()   #16198405.416388474
{% endraw %}
{% endhighlight %}

I'll try `bucket` this time, which compute a discrete binning with given numeric array.
{% highlight python %}
{% raw %}
companies = con.table('companies')
funding_buckets = [0, 10, 1000, 1000000, 10000000, 50000000]

bucket = (companies
          .funding_total_usd
          .bucket(funding_buckets, include_over=True))
bucket.value_counts()
{% endraw %}
{% endhighlight %}

It will return each sum value of each bucket.
{% highlight shell %}
{% raw %}
   unnamed  count
0      NaN  12055
1      0.0      4
2      1.0     42
3      2.0  15919
4      3.0  15754
5      4.0   7921
6      5.0   2597
{% endraw %}
{% endhighlight %}

Going on more, you can use `group_by` and `aggregate` to compute group summaries. `group_by` creates an intermediate grouped table expression, and `aggregate` aggregates table with a given set of reductions, grouping expressions, and post-aggregation filters.
{% highlight python %}
{% raw %}
metrics = (companies.group_by(bucket.name('bucket'))
           .aggregate(count=companies.count(),
                      total_funding=companies.funding_total_usd.sum()))

print(metrics)
{% endraw %}
{% endhighlight %}

Out:
{% highlight shell %}
{% raw %}
   bucket  count  total_funding
0     NaN  12055            NaN
1     0.0      4   1.700000e+01
2     1.0     42   1.518000e+04
3     2.0  15919   4.505161e+09
4     3.0  15754   5.712283e+10
5     4.0   7921   1.765166e+11
6     5.0   2597   4.460274e+11
{% endraw %}
{% endhighlight %}

This is just a part of Ibis framework. You could find more features which will make you more productive in [here](http://docs.ibis-project.org/api.html).

## Prepare VB access

For testing remote access, Ibis prepared demo [VirtualBox](https://www.virtualbox.org) image to test. You can download ova file [here](http://archive.cloudera.com/cloudera-ibis/ibis-demo.ova). It contains Impala DB and demo table to test.

![Screenshot](/assets/post_img/start_ibis/impala_in_vb.png)

Now you need to make it connective with local environment. If you are first in VB, you need to setup for this. Let's check network configuration in VB image.

{% highlight shell %}
{% raw %}
$ ifconfig eth0 | grep Mask
  inet addr:10.0.2.15  Bcase:10.0.2.255  Mask:255.255.255.0
{% endraw %}
{% endhighlight %}

Go to VB setting -> network tab and check IPv4 Address.

![Screenshot](/assets/post_img/start_ibis/vb_setting_network.png)

Now right click in `ibis-demo` image and press `setting`. Go to network -> Adapter 1 and press `Port Forwarding`

![Screenshot](/assets/post_img/start_ibis/vb_ova_setting_1.png)

Press plus button in right of the list, and create new one. Input Host IP as IPv4 Address of VB, and Guest IP with inet address of VB image. Give port number with 22.

![Screenshot](/assets/post_img/start_ibis/vb_ova_setting_2.png)

Check with `ping` to look if it connected well.

{% highlight shell %}
{% raw %}
$ ping 192.168.56.1
PING 192.168.56.1 (192.168.56.1): 56 data bytes
64 bytes from 192.168.56.1: icmp_seq=0 ttl=64 time=0.070 ms
64 bytes from 192.168.56.1: icmp_seq=1 ttl=64 time=0.101 ms
64 bytes from 192.168.56.1: icmp_seq=2 ttl=64 time=0.081 ms
64 bytes from 192.168.56.1: icmp_seq=3 ttl=64 time=0.130 ms
^C
--- 192.168.56.1 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
{% endraw %}
{% endhighlight %}

It's done.

## Remote access

Now let's make a code for remote access.

{% highlight python %}
{% raw %}
import ibis
import os


webhdfs_host = '192.168.56.1'
webhdfs_port = 22

ibis.options.interactive = True

{% endraw %}
{% endhighlight %}

You can setup global configuration with ibis.option. To make expressions to be executed immediately when printed to the console, setup `ibis.option.interactive` as True.
{% highlight python %}
{% raw %}
hdfs_port = os.environ.get('IBIS_WEBHDFS_PORT', 50070)
hdfs = ibis.hdfs_connect(host=webhdfs_host, port=hdfs_port)
con = ibis.impala.connect(host=webhdfs_host, database='ibis_testing',
                          hdfs_client=hdfs)

{% endraw %}
{% endhighlight %}

Now you can open Impala DB in remote VB. Try to get table data from here.
{% highlight python %}
{% raw %}

tables = con.list_tables()
print(tables)
table = con.table('functional_alltypes')
print(table)
{% endraw %}
{% endhighlight %}

{% highlight shell %}
{% raw %}
['functional_alltypes', 'tpch_ctas_cancel', 'tpch_customer', 'tpch_lineitem', 'tpch_nation', 'tpch_orders', 'tpch_part', 'tpch_partsupp', 'tpch_region', 'tpch_region_avro', 'tpch_supplier']
        id bool_col  tinyint_col  smallint_col  int_col  bigint_col  \
0     5460     True            0             0        0           0   
1     5461    False            1             1        1          10   
2     5462     True            2             2        2          20   
...
{% endraw %}
{% endhighlight %}

Now you and I are at the starting point of this platform. It has not been spread out wide yet, but it seems availability for data engineer as it can make research directly to remote database with local codes, and offers intuitive modules for research. It seems it worth to look on more deeply for now. Hope you could find more possibility in here.

