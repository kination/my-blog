---
layout: post
title: Full web service deployment(S3 + ElasticBeanstalk) with AWS
date: 2021-06-03
description: 
tags:
- aws
- frontend
- backend
- react
- nodejs
- s3
- elasticbeanstalk
permalink: full-web-service-deployment-aws
---



## Prepare command line for deployment


## Client-side deployment
First let's look on the process for deploying client-side(=front-end).

I've generated my client with React in most common way, using `create-react-app`. If you have not setup custom configuration, it will generate bundled static files inside `/build`(or maybe `/dist`) directory by `npm run build`. Deploying web client means uploading these bundled results into cloud storage, and connect to host.

### How about Netlify?
![image](/assets/post_img/full-web-service-deployment-aws/deploy-project-with-netlify.png)


First plan for client deployment was using Netlify. I've tried this in previous workplace, and it 

But if you're planning to connect 


### Deploy static file to S3
First thing to do, is setup permission for user to access S3 bucket. Let's add user, and check 'Programmatic Access', so we can access through CLI.

![image](/assets/post_img/full-web-service-deployment-aws/s3-add-user-1.png)

Here, give `S3FullAccess` permission, so user can upload static files to S3.

![image](/assets/post_img/full-web-service-deployment-aws/s3-add-user-2.png)

After you've created it, you will have some 'Access Key ID', and 'Secret access key'.
Setup this into your local AWS configuration through CLI:

```
$ aws configure --profile `IAM-user`
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
...
```

Or you can write directly inside `~/.aws/credentials`
```
...
[IAM-user]
aws_access_key_id = <your-access-key-id>
aws_secret_access_key = <your-secret-access-key-id>
```

Now create a bucket in S3 where static files will be kept. When creating it, you need to uncheck `Block Public Access settings for this bucket`, so user can access your pages.

![image](/assets/post_img/full-web-service-deployment-aws/s3-bucket-unblock-access.png)

After created, update bucket policy in `Permissions` options of bucket, so you could upload static files through CLI. Create through `Policy generator` inside policy editor, and result should be something like this.
```
{
    "Version": "2012-10-17",
    "Id": "<some-id>",
    "Statement": [
        {
            "Sid": "<some-statement-id>",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<your-bucket-name>/*"
        }
    ]
}
```

You can think value of "Version" is bit strange, cause it is already 2021. But this is version of `AWS policy grammer`, not related with permission itself. So keep this value.


## Server-side deployment with ElasticBeanstalk


```sh
$ eb init --profile remoconn-eb-api

Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) cn-northwest-1 : China (Ningxia)
14) us-east-2 : US East (Ohio)
15) ca-central-1 : Canada (Central)
16) eu-west-2 : EU (London)
17) eu-west-3 : EU (Paris)
18) eu-north-1 : EU (Stockholm)
19) eu-south-1 : EU (Milano)
20) ap-east-1 : Asia Pacific (Hong Kong)
21) me-south-1 : Middle East (Bahrain)
22) af-south-1 : Africa (Cape Town)
(default is 3): 9


Enter Application Name
(default is "server"): remoconn-api-server
Application remoconn-api-server has been created.

It appears you are using Node.js. Is this correct?
(Y/n): Y
Select a platform branch.
1) Node.js 14 running on 64bit Amazon Linux 2
2) Node.js 12 running on 64bit Amazon Linux 2
3) Node.js running on 64bit Amazon Linux
4) Node.js 10 running on 64bit Amazon Linux 2 (Deprecated)
(default is 1): 1

Cannot setup CodeCommit because there is no Source Control setup, continuing with initialization
Do you want to set up SSH for your instances?
(Y/n): n
```



```sh
eb create
Enter Environment Name
(default is remoconn-api-server-dev):
Enter DNS CNAME prefix
(default is remoconn-api-server-dev):

Select a load balancer type
1) classic
2) application
3) network
(default is 2):


Would you like to enable Spot Fleet requests for this environment? (y/N): N
Creating application version archive "app-210605_202130".
Uploading: [##################################################] 100% Done...
Environment details for: remoconn-api-server-dev
  Application name: remoconn-api-server
  Region: ap-northeast-1
  Deployed Version: app-210605_202130
  Environment ID: e-hpmici2nrq
  Platform: arn:aws:elasticbeanstalk:ap-northeast-1::platform/Node.js 14 running on 64bit Amazon Linux 2/5.4.0
  Tier: WebServer-Standard-1.0
  CNAME: remoconn-api-server-dev.ap-northeast-1.elasticbeanstalk.com
  Updated: 2021-06-05 11:22:27.385000+00:00
Printing Status:
2021-06-05 11:22:26    INFO    createEnvironment is starting.
2021-06-05 11:22:27    INFO    Using elasticbeanstalk-ap-northeast-1-460093525937 as Amazon S3 storage bucket for environment data.
 -- Events -- (safe to Ctrl+C)

```


```sh
$ eb status
Environment details for: remoconn-api-server-dev
  Application name: remoconn-api-server
  Region: ap-northeast-1
  Deployed Version: app-210605_202130
  Environment ID: e-hpmici2nrq
  Platform: arn:aws:elasticbeanstalk:ap-northeast-1::platform/Node.js 14 running on 64bit Amazon Linux 2/5.4.0
  Tier: WebServer-Standard-1.0
  CNAME: remoconn-api-server-dev.ap-northeast-1.elasticbeanstalk.com
  Updated: 2021-06-05 11:36:45.508000+00:00
  Status: Updating
  Health: Red
```

// DB setting

```sh
----------------------------------------
/var/log/nginx/error.log
----------------------------------------
2021/06/05 12:56:07 [error] 8546#8546: *59 connect() failed (111: Connection refused) while connecting to upstream, client: 172.31.25.18, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8080/", host: "172.31.40.212"
2021/06/05 12:56:17 [error] 8546#8546: *61 connect() failed (111: Connection refused) while connecting to upstream, client: 172.31.34.220, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8080/", host: "172.31.40.212"
2021/06/05 12:56:22 [error] 8546#8546: *63 connect() failed (111: Connection refused) while connecting to upstream, client: 172.31.25.18, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8080/", host: "172.31.40.212"
2021/06/05 12:56:32 [error] 8546#8546: *65 connect() failed (111: Connection refused) while connecting to upstream, client: 172.31.34.220, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8080/", host: "172.31.40.212"
2021/06/05 12:56:37 [error] 8546#8546: *67 connect() failed (111: Connection refused) while connecting to upstream, client: 172.31.25.18, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8080/", host: "172.31.40.212"
...
```

