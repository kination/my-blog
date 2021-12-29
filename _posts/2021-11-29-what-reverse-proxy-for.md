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

## Proxy, and reverse-proxy


## What is it for?


## Simple implementation

It's based on axum[https://github.com/tokio-rs/axum] web framework, which I've started looking on from few days ago...
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
    println!("ruver running on {}", addr);
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

    let uri = format!("http://127.0.0.1:3000{}", path_query);
    *req.uri_mut() = Uri::try_from(uri).unwrap();

    client.request(req).await.unwrap()
}
```


## Reference
* <https://en.wikipedia.org/wiki/Proxy_server>
* <https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/>
* <https://github.com/tokio-rs/axum>

