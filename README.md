# cachemachine

*cachemachine* is a simple caching engine for the HTTP request your Node.js applications make. It is designed to be run with Redis 
as the cache data-store but can be used without it in a single-node configuration during development.

```js
// startup cachemachine (default = cache every GET request for 1 hour)
var request = require('cachemachine')();
// make a request
request('http://www.google.com/', function(e, r, b) {
  // at this point, the request has completed and has been cached too
  console.log(b);
});
```

## Installation

Install *cachemachine* into your Node.js app with

```sh
npm install --save cachemachine
```

## Setup

Set up *cachemachine* in your own app with the default cache store:

```js
var cachemachine = require('cachemachine')();
```

or with Redis (localhost:6379)

```js
var cachemachine = require('cachemachine')({redis: true});
```

or with a remote Redis server: 

```js
var opts = {
  redis: true,
  hostname: 'myredisserver.com',
  port: 8000,
  password: 'mysecretpassword'
};
var request = require('cachemachine')(opts);
```

## Configure paths to cache

To specify which paths you'd like to be cached by *cachemachine*, then supply a `paths` parameter containing an array of objects e.g.:

```js
var opts = {
  paths: [
    { path: '/api/v1/.*', ttl: 3600 }
  ]
};
var request = require('cachemachine')(opts);
```

The objects that you pass in should containing

- path - a string or RegExp that defines the path you wish to match
- ttl - the time-to-live of the cache key in seconds

### Make requests

Simply use the request object as normal.

```js
request({method: 'get', url: 'http://mydomain.com/api/v1/books/123', qs: {limit:20}}, function(e, h, b) {
  console.log(b);
});
``` 

Note that cachemachine's request object only supports being called with a single 'object' parameter. It doesn't support Node.js streaming, nor can 
you use the `.get`, `.post` helpers.

### How does it work?

When an outgoing request is made through *cachemachine* where the path matches on of your regular expressions,  the url and query string 
are formed into a hash. This becomes the cache key for the cache data store. If the request can be retrieved from cache, it is 
fetched and the callback called. If the item is not in cache, it is fetched using a real 'request' object, cached and then returned to you.


### Why cachemachine?

If you're using `request` already and don't want to change your code, then you can use *cachemachine* as a drop-in replacement and decide which
HTTP calls to cache and for how long. This can take the load of over-burdened API servers and to speed up your service.