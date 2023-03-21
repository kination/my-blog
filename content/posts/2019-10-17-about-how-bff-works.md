---
layout: post
title:  "BFF(Back-end for Front-end) inside, for microservice"
date:   2019-10-17
description: What is BFF pattern, and how it work for service
tags:
- web
- microservice
- backend
- frontend
- node
- express
permalink: about-how-bff-works
---

Microservice, or Microservice architecture is not a special technology anymore. There are lots of project which will help build-up micro-service quickly, and major cloud service provider is offering options for this too. And engineers are starting to think the efficiet ways to offer/use micro-service based system.


## API Gateway and BFF(Back-end for Front-end)
Basically, microservice is composed of multiple services. It means that all of them are included in different machines with different hosts.

![Screenshot](/assets/post_img/about-how-bff-works/without-api-gateway.png)

When calling API, you need to setup different host to get information from different domain. Moreover, you need to check authentication data(such as token) everytime before requesting.

![Screenshot](/assets/post_img/about-how-bff-works/with-api-gateway.png)

Gateway is not just making single request point. It encapsulates the internal system, and make single interface which can work as 'gate' of total infrastructure. Service only needs to handle lot of parts related with authentication, load balancing, caching, and more inside this gate. 

Of course, there are some problems. 
- Because all API reqeust are coming through this gate, it causes bottle-neck issue.
- Different type of devices could require different data, but it cannot cope with it flexable. How they can do is to make a new(or fix current ones) API in gateway.

So some smart engineers in SoundCloud make a new approach for this, and named as BFF(Back-end for Front-end) pattern.

![Screenshot](/assets/post_img/about-how-bff-works/sc-bff-pattern.png)

This is probably the most famous diagram which explains BFF pattern. Instead of designing API in gateway to handle every cases(or devices), make separate 'gateway' for different type of clients(web, mobile).

Actually, these 2 kinds of pattern are logically similar. It is to make layer in the middle between client and server, to arrange the request and handle common functions in single point.

If you want to know more, please [take a look](https://philcalcado.com/2015/09/18/the_back_end_for_front_end_pattern_bff.html).


## Proxy for Express Framework
Most simple design for BFF(or Gateway) is to use proxy. There are good open source module [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) for NodeJS user to implement it with ease.

```javascript
const proxy = require('http-proxy-middleware')
// ...

function onProxyReq(proxyReq, req, res) {
  // do something on request
}

function onProxyRes(proxyRes, req, res) {
  // do something on response
}

function onError(err, req, res) {
  res.writeHead(500, {
    'Content-Type': 'text/plain'
  })
  res.end('Something went wrong. And we are reporting a custom error message.')
}

app.use(
  '/api/v1/order',
  proxy({
    target: 'https://order-service-host/',
    changeOrigin: true,
    onProxyReq,
    onProxyRes,
    onError
  })
)

app.use(
  '/api/v1/shipping',
  proxy({
    target: 'https://shipping-service-host/',
    changeOrigin: true,
    onProxyReq,
    onProxyRes,
    onError
  })
)
```

In this case, every API requests which includes `/api/v1/order` will be redirect to microservice with domain `https://order-service-host/`. If we need to modify request or response data, we can do it by default middleware offered by this module.

For example, if you want to add 'Bearer' token for every request, it can be done as:
```javascript
function onProxyReq(proxyReq, req, res) {
  // add custom header to request
  const token = apiToGetBearerToken()
  proxyReq.setHeader('Authorization', `Bearer ${token}`)
}
```

or you could want to send error logs to ELK for alert system. Do it like:
```javascript
let winston = require('winston')
let Elasticsearch = require('winston-elasticsearch')

let esOption = {
  level: 'error'
}
let logger = winston.createLogger({
  transports: [
    new Elasticsearch(esOption)
  ]
})

// ...
function onError(err, req, res) {
  logger.error(`ERROR: ${err.message} / CODE: ${err.code}`)
  res.writeHead(500, {
    'Content-Type': 'text/plain'
  })
  res.end('Something went wrong. And we are reporting a custom error message.')
}
```


## Bit advanced logic for better performance
There is some case to avoid using proxy for better performance.
![Screenshot](/assets/post_img/about-how-bff-works/only-proxy-chart.png)

As in image, there can be a case which needs to get data from multiple domains, and response from 'API-1' and 'API-2' is useless for rendering in client. It means there are 'waste of http request/response' on client. This is usually very small, but it can be very big.

Moreover, if your service are hosted by cloud computing service provider, network performance will be stable(and fast!) between microservices if they are in same cluster. 
 But between BFF and client, network speed will be highly effected by where user are, and if client is in place with poor network signal, this 'waste of http request/response' will effect more to the service performance.

![Screenshot](/assets/post_img/about-how-bff-works/without-proxy-chart.png)

This is bit advanced flow to reduce request/response size and(or) number between client and BFF. But using proxy, we cannot change the flow by status.

So for example if we want to find shipping status of the order, and need to call API from 'order' and 'shipping' to domain(assume client will call `/api/v1/bff/shipping-of-order` for this...), we can go on as:

```javascript
const axios = require('axios')

app.use(
  '/api/v1/bff/shipping-of-order',
  bffAPIs.getShippingOfOrder
)

// ...

const bffAPIs = {
  getShippingOfOrder: async (req, res, next) => {
    try {
      const token = apiToGetBearerToken()
      const orderInfo = await axios({
        baseUrl: 'https://order-service-host/',
        data: {
          // put data to get order...
        },
        headers: {
          `Authorization': Bearer ${token}`,
        }
      })

      if (orderInfo.shippingId) {
        const shippingInfo = await axios({
          baseUrl: 'https://shipping-service-host/',
          data: {
            // put data to get shipping
          },
          headers: {
            `Authorization': Bearer ${token}`,
          }
        })

        res.status(200).send({
          data. shippingInfo.data
        })

      } else {
        res.status(404).send({
          message: 'No shipping ID!'
        })
      }
    } catch (e) {
      // send exception...
      res.status(e.response.status).send({
        message: 'Exception!!'
      })
    }
  }
}
```

As you can see, BFF side code has been bigger, and logic has more complexity because we need to check data, and return value by status in person. But if there are cases to check status from multiple domain and client needs only few data from final result, this could make user more happy.


## Reference
* <https://microservices.io/patterns/microservices.html>
* <https://philcalcado.com/2015/09/18/the_back_end_for_front_end_pattern_bff.html>
* <https://www.nginx.com/blog/building-microservices-using-an-api-gateway/>
* <https://docs.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends>
* <https://tsh.io/blog/design-patterns-in-microservices-api-gateway-bff-and-more/>
* <https://fullstackdeveloper.tips/seeing-the-bff-pattern-used-in-the-wild>

