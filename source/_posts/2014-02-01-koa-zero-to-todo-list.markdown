---
layout: post
title: "koa: zero to todo list"
date: 2014-02-01 17:12
comments: true
categories: koa generators javascript harmony ecmascript6
---

##Note: you need to run node 0.11 with --harmony to run the code.

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

Like express, Koa works internally by generating a callback that can be passed to 
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

Unlike Express, the middleware is written using generators.
Downstream middleware in Koa flows upstream on return. Middleware
yields down stream by explicitly calling 'yield next.'. Upon return, the control
flow yields back up to where the upstream middleware yielded downstream.

Where Express passes native node req and res objects through to each function, Koa
manages a context where it encapsulates them behind an interface. They are still available 
through the 'this' keyword as this.req and this.res. However, Its not reccomended in
the docs that you work with these native objects. One could imagine calling this.res.end('')
would throw a monkey wrench in the control flow. Instead you are supposed to work through
the this.response and this.request. Most of the methods are aliased directly to this. 'this.body'
for example, is aliased to this.response.body. 
  
There does not yet appear to be a direct way to get
to the request body. The co-body parser accesses the req.body directly so while the docs say 
don't do it, Koa is still young so you may have to get your hands dirty.

### A Todo app in koa

Now that we've covered the basics, lets try something a little more complex. A todo
List seems like a good thing no one has ever tried to make before in any technology ever!
To simplify assumptions, lets just store the todos in memory.

By itself, Koa is very minimalistic. It does not provide body parsing, sessions, or 
routing in the core. Unfortunately Koa is still young so there just aren't that many 
npm modules for it just yet. A quick search on [the Koa website](https://github.com/koajs/koa/wiki)
shows that we do have the necessary modules for a basic todo.

  1. [koa-route](https://github.com/koajs/route): for routing
  2. [co-body](https://github.com/visionmedia/co-body): for parsing the body of post requests
  3. [koa-static](https://github.com/koajs/static): for serving up static assets

Here's the basic server side api
{%codeblock lang:javascript %}
//jshint esnext
var koa          = require('koa');
var staticServer = require('koa-static');

//this allows us to parse the native req object to get the body
var parse        = require('co-body');

var router       = require('koa-route');
var _            = require('underscore');

var Promise      = require('bluebird');
var path         = require('path');

var fs = Promise.promisifyAll(require('fs'));
var app = koa();
//our very basic data store
var todos = [];

//gets us unique ids
var counter = (function() {
  var count = 0;
  return function() {
    count++;
    return count;
  };
})();

//serve up the public directory where we have all the assets
app.use(staticServer(path.join(__dirname, 'public')));

app.use(router.post('/todos', function *() {
  /*
    yield lets us pass asynchronous functions that return promises or thunks
    It will freeze the middleware till its resolved and pass it back in.
  */
  var todo = (yield parse.json(this));

  todo.id = counter();
  todos.push(todo);
  this.body = JSON.stringify(todos);
}));


app.use(router.get('/todos', function *() {
  this.body = JSON.stringify(todos);
}));

app.use(router.delete('/todos/:id', function *(id) {
  todos = _(todos).reject(function(todo) {
    console.log('what? ', todo, id );
    return todo.id === parseInt(id, 10);
  }, this);
  this.body = JSON.stringify(todos.sort(function(a, b) { return a - b;}));
}));



app.listen(3000);
console.log('listening on port 3000');
{% endcodeblock %}

[Download the code on github](https://github.com/cultofmetatron/koa-todo)
The github version includes frontend code.

####A few things of note:

The 'yield' keyword ca do some interesting things. If we pass into it
an asynchronous function that returns a thunk or promise, it will stop execution of
the middleware and wait till it resolves. It then returns the value of the promise or thunk
and resumes the generator. This is a hell of a lot easier to read.

####A word of caution

The 'yield' keyword lets us do some safe blocking but it isn't always the ideal solution.
While the event loop itself isn't blocked by it the way futures can, it does block resuming
of any operations that occur after it. 

For example, if we run three asynchronous operations top to bottom that do not depend on 
each other, like the following...

{%codeblock lang:javascript %}
app.use(function *() {
  var a = yield async1();
  var b = yield async2();
  var c = yield async3();
});
{% endcodeblock %}

This completely defeats the purpose of node's (almost automatic) concurrency. When we call
async1, we are blocking until async2 runs. This is unoptimal. It would be better to get the promises
for the three functions and yield on a merged promise.


{%codeblock lang:javascript %}
app.use(function *() {
  var a = async1();
  var b = async2();
  var c = async3();
  var result = yield Promise.all([a, b, c]);
});
{% endcodeblock %}


I'm excited to cut my teeth on some bigger problems. As the framework matures, Its going to 
allow more fine grained control for how we write the next generation of web applications.











