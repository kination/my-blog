---
layout: post
title: Things of do and don't
date: 2019-02-28
description: Things I don't know in early 2019, as a developer
tags:
- developer
- sw engineer
permalink: things-of-do-dont
---

Few days ago, I read a blog post '[Things I Don’t Know as of 2018](https://overreacted.io/things-i-dont-know-as-of-2018/)' written by Dan, and it has been a great motivation to tidy up what I have done, and what I know.

I subscribe several facebook pages and blog post about software engineering, and getting some information of new tech and trends every day. But that is way different from becoming good at that. For example, I can tell somebody about the usage of `Tensorflow` after I read some posts, but that does not mean I can work as deep learning engineer.

I understand of this, but still being confused about the 'true knowledge'. Now it has been 2019, and seems good time to clean up about my knowledge.

Of course, we know we can do (almost)everything by Google(or GitHub, StackOverflow...) power. We can get knowledge by searching with suitable keywords. So the word `don't` in this post, means `I can do it, but needs help from internet`.

![Screenshot](/assets/post_img/things-of-do-dont/google-main.png)

Some of articles are brought from Dan's.


## Basic, and theories
- Unix command or shell script: I fill comfortable of using basic command such as `ls`, `cd`, `mv`, `ps` to edit/find file in terminal, or check thread status. I also can edit simple code/document using `vi`. But I cannot do more complicate things without google's help.

- Network: I understand TCP/UDP protocol, IP address and 7 layer. Also know about HTTP/HTTP2 too. But that's all. I don't know the detail about low-level network part.

- Database: I know how to use simple SQL query, and work on table design for few times. I also understand about transaction. But don't know about database tuning, and more other things.

- Pattern & Paradigm: I know some design patterns such as singleton, observer, factory, proxy, etc. and simply can implement in this way. But I cannot describe the detail of usage, to optimize the system. I'm also looking on reactive programming, but didn't used on production level. About functional programming, I just can explain what it is with pros and cons...that's all.

- Security: I know basic flow of SSL layer, and how it effects in HTTPS. Also faced on issue about CORS, but don't know the detail. I have made API service using JWT token and OAuth2 demo server, and review about the vulnerability.


## Web/Mobile application
- Front-end(Client): I had developed client-side project with AngularJS(not Angular), and React. I tried using Redux pattern and styled component, and know why this is useful. I once converted JavaScript project to TypeScript, and felt the strength of typed language. I have used gulp and webpack for build process, and understand the usefulness of these.

- Android: I have developed android application for MDM, and NDK library for video streaming. I didn't remember all things, but designed connection between native library and android app. I know about lifecycle of `Activity` and `Fragment`, and making `AsyncTask` such as network connection. I also know about the basic of image handling, and know useful modules such as `Picasso` and `Glide`. But not good on such thing as `Dependency Injection`, or useful network library like `OkHttp3`.

- iOS: I have some experienced on working with Objective-C, but forgot most of them. I don't tried Swift yet.


## Back-end, server
- API server: I have developed API server, and understand how they interact with client side and making connection with database. I understand credential for authentication, and REST(though I never made my APIs RESTful...)

- Message queue, and task system: I know why message queue system is needed, and support making distribution task system based on `celery`(Python). But don't know good about load balancing, or more.

- Data pipeline: I know what ETL(extract+transform+load) is needed for, and designed a demo of data pipeline based on `Kafka`+`Redis`, but don't know more than that.

- Server/Cloud: I have worked on on-premises environment, so understand the role of infrastructure team. I know the needs of duplexing, and how-to debug based on apache/nginx log. About cloud, I have tried App engine/Bigquery/Database in GCP, and run simple project in AWS. But not know deeply about optimization.


## Others
- Container: I know docker, and used for containerizing my project in development environment with `Dockerfile` and `docker-compose`. But I didn't tried k8s or swarm, and deploying container to server yet.

- CI/CD: I know code review flow and continouos integration, and have experience of guiding basic flow of to our team before. I used `Jenkins` on work, and `Travis` for my open source project.

- XML/JSON/GraphQL: I know what they are, and implemented data parser for XML/JSON format. I tried making request in GraphQL format, but don't know more.

- Data engineering and Machine learning: I know tools for using these such as `scikit-learn`, `tensorflow`, and tried several challenges including using `kaggle`. I understand data preprocessing, and how to do it by using `pandas`. But cannot go more than that. I don't know things such as selecting efficient algorithms, or optimizing deep learning network.

This is just tip of the iceberg, but you could know a bit about what kind of developer I am...

## ...and next
Knowing about myself is important. I felt more about this while I'm working as developer, because I always bumped against the wall when I try to explain about what I'm not good about it.

But I think we don't need to fill shame about it, because nobody in the world knows everything you don't know. I truly guarantee that the greatest engineers around you also have more things they don't know than something they knows.

If you don't know, you can learn it. Google, GitHub, StackOverflow, and lots of guide in MOOC site can help you. Anyway, developer is forever-learning job, until quiting. Just learn it if there is something you need to. As Dan said in end of his post,

```
I’m aware of my knowledge gaps (at least, some of them).
I can fill them in later if I become curious or if I need them for a project.

This doesn’t devalue my knowledge and experience.
There’s plenty of things that I can do well.
For example, learning technologies when I need them.
```

There are several target I want to be better in 2019, such as:
- Getting more better in programming languages, especially `Rust` and `Go`
- Be better of using Cloud
- Understand more about ML/DL, and data engineering
- and more...

Hope these can be list of `Do` in next year.
