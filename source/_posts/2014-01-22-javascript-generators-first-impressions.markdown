---
layout: post
title: "javascript generators: first impressions"
date: 2014-01-22 16:03
comments: true
categories: generators promises
---

Ecmascript 6 (harmony) is comming out soon and one of the most exciting features
it offers are generators. Generators are a minimalist flow control system that gives
a much finer grained level of control than we were afforded up till now.

> Note: the code in this blog will only run in node v0.11.x when run as --harmony.

Like a function, a generator is an object that declares some behavior. Its first class
just like javascript functions and you can pass it around as values and return them from other
functions.

A generator is declared like a function only with a '*' before the parens.
We then create an instance of the generator by calling it.

Here's a basic example.

{% codeblock lang:javascript %}
var myGenerator = function *() {
  var foo = yield 5;
  console.log('this doesn't get written until the second call to next()');
};

var gen = myGenerator();
var state = gen.next();
console.log(state.value) //=> 5

{% endcodeblock %}

When we run gen.next(), the code executes until we get to yield. The generator
then stops which is why the console.log() does not get called. The state of the 
generator is returned by next which gives us two things.

  1. state.value: the value on the right side of the yield; in this case 5.
  2. state.done: a boolean that returns true if there are no more yields in the generator.

We've called gen.next() the one time. The second time we call gen.next(), we have the option
of passing in an argument to it that will be returned by yield inside the generator.

This example shows a more advanced example of bidirectional passing. 

{% codeblock lang:javascript %}
var myGenerator = function *() {
  var firstWord = yield 5;
  console.log(foo); //=> "hello"
  var secondWord = yield 10;
  console.log(secondWord); //=> "world"
};

var gen = myGenerator();
var state = gen.next();
console.log(state.value); //=> 5
state = gen.next("hello");
//=> 'hello' gets printed to the screen from inside the generator
console.log(state.value); //=> 10
console.log(state.done); //=> false
state = gen.next('world');
//=> 'world' gets printed
console.log(state.value); //=> 10
console.log(state.done);  //=> true

{% endcodeblock %}

One of the biggest growing pains of javascript development is wrapping one's
head around async programming and functions run asyncronously. Promises give us a
value that represents the eventual value returned from an asynchronous function.

Promises are promised in ecmascript 6 but they aren't available when I run node 0.11
with --harmony  yet so I use [Bluebird](https://github.com/petkaantonov/bluebird)

{% codeblock lang:javascript %}
var Promise = require('bluebird');

//extends node's fileSytem with versions of the async functions that return promises
//the promisified versions are the original name + 'Async'
var fs = Promise.promisifyAll(require('fs'));

fs.readFileAsync('.gitignore',  'utf8').then(function(contents) {
  console.log(contents); // prints the contents of the .gitignore file.
});

console.log('this runs before the callback passed to "then" which is counterintuitive.')
{% endcodeblock %}

This code Works because when the function is called, it creates a closure that doesn't get
garbage collected because the function passed to the promise retains a reference to this scope.
When its called, it can operate on variables in this containing scope. However we
cannot return to the original function call. Thus Unless you are used to thinking about promises,
its a bit unintuitive that the console.log on the following line runs before the callback passed
to the then() handler of the promise.

Generators on the other hand, let us FREEZE the execution context until the file resolves.

There's an excellent library called
[co](https://github.com/visionmedia/co) from the creator of express that allows
us to create coroutines using generators. thus we could write the previous code using
generators.

{% codeblock lang:javascript %}
var Promise = require('bluebird');
var fs = Promise.promisifyAll(require('fs'));
var co = require('co');

co(function *(){
  var a = yield fs.readFileAsync('.gitignore',  'utf8');
  console.log(a); //this doesn't run until the previous function resolves.
  var c = yield fs.readFileAsync('package.json', 'utf8');
  console.log(a);
  console.log(c);
  return;
})();

{% endcodeblock %}

This is pretty exciting, This Asyncrouous code looks downright synchronous. Its also running
in its own context so it doesn't block the event loop. Within the generator, we can write
much more fine grained flow control for asynchronous functions.

That said, how does co work? The basic premise is that we use yield to pass back the promise
into co where it waits till the function resolves. Then we call the next() of this function
passing in the value from the resolved promise.

co itself is extremely flexible allowing you to pass in thunks, or A+spec promises into
yield. Here I'll demonstrate a simplified version that can accept only promises.

{% codeblock lang:javascript %}
var co = function(fngen) {
  /*
  next takes a instatiated generator and calls
  and a value returned from calling next on it
  gen is an instance of a generator
  yieldable is the value returned from calling gen.next()
  */
  var next = function(gen, yieldable) {
    if (! yieldable.done) { //if 
      //we assume yieldable.value is a promise so we call then() to get the value
      yieldable.value.then(function(val) {
        /*
        we call next on gen and pass in the value into gen.next() to 
        inject the value back into out coroutine where it gets returned
        by the yield in the generator. 
        
        By call gen.next(val), gen resumes execustion passing val back 
        and gen.next() return when it hits the next yield keyword returning 
        the value passed in to yield.
        */
        next(gen, gen.next(val));
      });
    }
  };

  return function() {
    //instatiate the generator
    var gensym = fngen();
    //get the first yieldable
    var yieldable = gensym.next();
    if (!yieldable.done) {
      next(gensym, yieldable);
    }
  };
};


{% endcodeblock %}

The concept is pretty simple, yield passed back the value on the right to gen.next() 
which it returns. The value we pass into the gen.next call to gen.next() becomes
the value returned by yield. Sorta like a zig zag or a needle stitching.

I'm excited to see some of the new projects that will take advantage of this new
ecmascript 6 feature. One big example comming to mind is the new koa framework. Unlike
its predecessor express/connect, Koa is a set of pluggable middleware components
that utilize generators heavily for flow control.



