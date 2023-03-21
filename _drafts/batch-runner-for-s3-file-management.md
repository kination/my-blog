---
layout: post
title:  Batch runner for S3 file management
date:   2020-08-11
description: 
tags:
- python
- dataengineer
- shell
- batch
- cron
permalink: batch-runner-for-s3-file-management
---

My career was keep changing during 10 years of working, from system engineer to web engineer. And I've started working as data engineer few weeks ago(but keep studied myself)

....

## Design flow


## Access S3 with boto3
[boto3](https://github.com/boto/boto3) is SDK to make Python developers to write software that makes use of services like Amazon S3 and Amazon EC2. I'm not sure whether this is official or not(why name is boto, instead of something like `aws-sdk-python`?), but it has been introcuded in official AWS guide, so you could ensure using it.

![Screenshot](/assets/post_img/batch-runner-for-s3-file-management/aws-boto-introduce.png)


#### Fetch files

```python
import boto3
from botocore.client import Config

config = Config(
  connect_timeout=10, 
  read_timeout=30, 
  retries={'max_attempts': 1}
)

client = boto3.client('s3', config=config)
paginator = client.get_paginator('list_objects')

pages = list(paginator.paginate(
  Bucket=bucket, 
  Delimiter='/', 
  Prefix=prefix
))

```

This is how to initiate instance of `boto3` client. `Config` is optional, but it will be good to define because it can 





#### Archive files in list


## Shell script to run by cron




## Plus, setup slack hook

