# k-fastify-gateway
A Node.js API gateway that just works!

[![Build Status](https://travis-ci.org/jkyberneees/fastify-gateway.svg?branch=master)](https://travis-ci.org/jkyberneees/fastify-gateway)
[![NPM version](https://img.shields.io/npm/v/k-fastify-gateway.svg?style=flat)](https://www.npmjs.com/package/k-fastify-gateway) [![Greenkeeper badge](https://badges.greenkeeper.io/jkyberneees/fastify-gateway.svg)](https://greenkeeper.io/)  
Read more about it: https://medium.com/sharenowtech/k-fastify-gateway-a-node-js-api-gateway-that-you-control-e7388c229b21

## Get started in two steps

Install required dependencies:
```bash
npm i fastify k-fastify-gateway
```
> NOTE: From v2.x, `fastify-reply-from` is a direct dependency.

Launch your gateway 🔥:
```js
const fastify = require('fastify')({})

// required plugin for HTTP requests proxy
fastify.register(require('fastify-reply-from'))

// gateway plugin
fastify.register(require('k-fastify-gateway'), {

  middlewares: [
    require('cors')()
  ],

  routes: [{
    prefix: '/public',
    prefixRewrite: '',
    target: 'http://localhost:3000',
    middlewares: [],
    hooks: {
      // async onRequest (req, reply) {},
      // onResponse (req, reply, res) { reply.send(res) }
    }
  }, {
    prefix: '/admin',
    target: 'http://localhost:3001',
    middlewares: [
      require('basic-auth-connect')('admin', 's3cr3t-pass')
    ]
  }, {
    prefix: '/user',
    target: 'http://localhost:3001'
  }]
})

// start the gateway HTTP server
fastify.listen(8080).then((address) => {
  console.log(`API Gateway listening on ${address}`)
})
```

## Introduction

Node.js API Gateway plugin for the [fastify](https://fastify.io) [ecosystem](https://www.fastify.io/ecosystem/), a low footprint implementation that uses the [fastify-reply-from](https://github.com/fastify/fastify-reply-from) HTTP proxy library.  

Yeap, this is a super fast gateway implementation!

### Motivation

Creating fine grained REST microservices in Node.js is the [easy part](https://thenewstack.io/introducing-fastify-speedy-node-js-web-framework/), difficult is to correctly integrate them as one single solution afterwards!  

This gateway implementation is not only a classic HTTP proxy router, it is also a Node.js friendly `cross-cutting concerns` management solution. You don't have to: 
 - repeat in/out middleware logic anymore (cors, authentication, authorization, caching, ...)
 - blame Node.js because the asynchronous post processing of proxied requests was hard to implement...
 - ...
 - or just learn Lua to extend nginx ;)

## Configuration options explained

```js 
{
  // Optional global middlewares (https://www.fastify.io/docs/latest/Middlewares/). Default value: []
  middlewares: [],
  // Optional global value for routes "pathRegex". Default value: '/*'
  pathRegex: '/*',

  // HTTP proxy
  routes: [{
    // Optional path matching regex. Default value: '/*'
    // In order to disable the 'pathRegex' at all, you can use an empty string: ''
    pathRegex: '/*',
    // route prefix
    prefix: '/public',
    // Optional "prefix rewrite" before request is forwarded. Default value: ''
    prefixRewrite: '',
    // remote HTTP server URL to forward the request
    target: 'http://localhost:3000',
    // optional HTTP methods to limit the requests proxy to certain verbs only
    methods: ['GET', 'POST', ...], // any of supported HTTP methods: https://github.com/fastify/fastify/blob/master/docs/Routes.md#full-declaration
    // Optional route level middlewares. Default value: []
    middlewares: [],
    // Optional proxy lifecycle hooks. Default value: {}
    hooks: {
      async onRequest (req, reply) {
      //   // we can optionally reply from here if required
      //   reply.send('Hello World!')
      //
      //   return true // truthy value returned will abort the request forwarding
      },
      onResponse (req, reply, res) {  
        // do some post-processing here
        // ...
        // forward response to origin client once finished
        reply.send(res) 
      }

      // other options allowed https://github.com/fastify/fastify-reply-from#replyfromsource-opts
    }
  }]
}
```
## Gateway level caching
### Why?
> Because `caching` is the last mile for low lateny distributed systems!  

Enabling proper caching strategies at gateway level will drastically reduce the latency of your system,
as it reduces network round-trips and remote services processing.  
We are talking here about improvements in response times from `~100ms` to `~2ms`, as an example.  

###  Setting up gateway cache
#### Single node cache (memory):
```js
// cache plugin setup
const gateway = require('fastify')({})
gateway.register(require('k-fastify-gateway/src/plugins/cache'), {})
```
> Recommended if there is only one gateway instance

#### Multi nodes cache (redis):
```js
// redis setup
const CacheManager = require('cache-manager')
const redisStore = require('cache-manager-ioredis')
const redisCache = CacheManager.caching({
  store: redisStore,
  db: 0,
  host: 'localhost',
  port: 6379,
  ttl: 30
})

// cache plugin setup
const gateway = require('fastify')({})
gateway.register(require('k-fastify-gateway/src/plugins/cache'), {
  stores: [redisCache]
})
```
> Required if there are more than one gateway instances

### Enabling cache for service endpoints
Services are in control...

### Invalidating cache
> Let's face it, gateway level cache invalidation was complex..., until now!  


## Breaking changes
In `v2.x` the `hooks.onResponse` signature has changed from:
```js
onResponse (res, reply)
```
to:
```js
onResponse (req, reply, res)
```
> More details: https://github.com/fastify/fastify-reply-from/pull/43

## Benchmarks
`Version`: 2.0.1  
`Node`: 10.15.3  
`Machine`: MacBook Pro 2016, 2,7 GHz Intel Core i7, 16 GB 2133 MHz LPDDR3  
`Gateway processes`: 1  
`Service processes`: 1

```bash
Running 30s test @ http://127.0.0.1:8080/service/hi
  8 threads and 8 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   841.58us  662.17us  35.22ms   98.66%
    Req/Sec     1.23k   130.62     1.29k    95.02%
  293897 requests in 30.10s, 42.60MB read
Requests/sec:   9763.61
Transfer/sec:      1.42MB
```

## Want to contribute?
This is your repo ;)  

> Note: We aim to be 100% code coverage, please consider it on your pull requests.