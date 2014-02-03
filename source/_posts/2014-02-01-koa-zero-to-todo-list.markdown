---
layout: post
title: "koa: zero to todo list"
date: 2014-02-01 17:12
comments: true
categories: koa generators javascript harmony ecmascript6
---

From the creators of express comes a brand new framework for node powered by
the new ecmascript 6 generators syntax. [Koa](http://koajs.com/) is an interesting
reimagining of how we will be able to build web applications in javascript.

###The old paradigm

In the standard node library, The 'http' module is used to create servers.

{%codeblock lang:javascript %}

  var server = http.createServer(function(req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    //server logic goes here
    res.end('');
  });

  server.listen(3000, '127.0.0.1');
  console.log('listening on port 3000');

{% endcodeblock %}

Express exposes a function that we can give to 
http.createServer as a callback. Express middleware is a set of functions that 
take 3 arguments, req, res and next. The middleware performs some operations, modifies
either the request or response objects and passed down to the next middleware in the stack
by calling next(). Its akin to a water flow model where the the response ends somewhere 
near the end of the middleware stack.

### Enter Koa, The generators based framework

Like express, Koa works internatlly by generating a callback that can be passed to 
http.createServer().  Unlike Express, it uses generators to provide a much more fine grained
model of control flow.

A very basic koa app looks like this, lets make it serve up the contents of a file

{%codeblock lang:javascript %}
  var koa          = require('koa');
  var Promise      = require('bluebird');

  //creates promise yielding versions of fs
  var fs = Promise.promisifyAll(require('fs'));
  //create the koa instance
  var app = koa();

  app.use(function *(next) {
    //here's an example middleware that logs to the console
    console.log('timestamp: before request => ', time.now());
    yield next
    console.log('timestamp: after request  => ', time.now());
  });

  app.use(function *() {
    this.body = yield fs.readFileAsync('./app.js', 'utf8');
  });
  
  app.listen(3000);
  console.log('now listening on port 3000');

{% endcodeblock %}





