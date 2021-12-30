---
layout: post
title:  What is reverse proxy for
date:   2021-11-29
description: 
tags:
- proxy
- reverseproxy
- rust
- network
permalink: what-reverse-proxy-for
---


About year ago, I've wrote some post about BFF architecture. One of the architecture's component was 'reverse proxy'. This is for more description about it.


## Proxy server
Definition of `proxy server` in networking, means server application works as relay server between client and target server. By using this, client can request file or data through proxy server, which evaluates the requests to make required network transaction.

#### Forward & Reverse 
'Forward proxy' is server that sits in edge of network, which regulates outbound traffic by defined policies in network cluster.

Position of 'reverse proxy' also locates in front of the client(same as forward). Clients send requests to the origin server of a website, those requests are intercepted by the reverse proxy server. The reverse proxy server will then send requests to and receive responses from the origin server.

![60fa69b8b70d92626672eac3_Reverse Proxies vs Forward Proxies](https://user-images.githubusercontent.com/1720209/147718893-f11e4450-4669-48da-8893-2ca650af2820.png)

The difference between these 2 are pretty delicate. An easy approach to summarize it is saying that `forward proxy` sit before a customer and guarantee that no beginning worker at any point discusses straightforwardly with that particular customer. And `reverse proxy`, intermediary sits before a beginning worker and guarantees that no customer at any point discusses straightforwardly with that beginning worker.


## Simple implementation of Reverse proxy
Assume that there are web server running `localhost:9000`, which returns following at `/proxy`
```
{"hello":"world"}
```
Now I'll make simple reverse proxy server running in `localhost:4000`, which will catch request and send it again to specific target to get response if API is `/proxy`.

<img width="1186" alt="Screen Shot 2021-12-30 at 11 33 43" src="https://user-images.githubusercontent.com/1720209/147717189-496f5b7b-6f01-4a54-840d-8092417b53a1.png">

This is simple implementation based on [axum](https://github.com/tokio-rs/axum) web framework for this.

```rust
use axum::{
    extract::Extension,
    http::{uri::Uri, Request, Response},
    routing::get,
    AddExtensionLayer, Router,
};
use hyper::{client::HttpConnector, Body};
use std::{convert::TryFrom, net::SocketAddr};

type Client = hyper::client::Client<HttpConnector, Body>;

#[tokio::main]
async fn main() {
    let client = Client::new();

    let app = Router::new()
        .route("/proxy", get(proxy_handler))
        .layer(AddExtensionLayer::new(client));

    let addr = SocketAddr::from(([127, 0, 0, 1], 4000));
    println!("server running on {}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn proxy_handler(Extension(client): Extension<Client>, mut req: Request<Body>) -> Response<Body> {
    let path_query = req
        .uri()
        .path_and_query()
        .map(|v| v.as_str())
        .unwrap_or(req.uri().path());

    let uri = format!("http://127.0.0.1:9000{}", path_query);
    *req.uri_mut() = Uri::try_from(uri).unwrap();

    client.request(req).await.unwrap()
}
```

Make this run, and you can receive the data `{"hello":"world"}` through reverse proxy server by calling `localhost:4000/proxy` instead of calling `localhost:9000/proxy`.


## What is it for?

#### Performance
World's top web services, such as `Google` or `Facebook`, should get billions of requests every day, and it is mission impossible to handle all incoming traffic with single origin server. In this case, reverse proxy can be work as load balancer solution and distribute traffics to divided origin servers.

Not only this, it can store data to make it run as caching system. So instead of making requests to origin server for every case, it can return the result to client with cached data directly. It can reduce response time of request dramatically.

#### Efficient security
Reverse proxy are placed in front of network cluster, so web service don't need to reveal IP address of origin servers. Hackers can only face to reverse proxy server, and maintainer also only need to focus on security of proxy server.

Also, because there are unified endpoint, it doesn't need to encrypt and unscramble SSL for every system.


## Reference
* <https://en.wikipedia.org/wiki/Proxy_server>
* <https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/>
* <https://www.wallarm.com/what/what-is-the-reverse-proxy>
* <https://github.com/tokio-rs/axum>

