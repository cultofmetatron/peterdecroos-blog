---
layout: post
title: "Functional programming with ecmascript6 generators"
date: 2014-02-08 19:12
comments: true
categories: functional-programming ecmascript6 javascript generators
---

The web is abuzz right now with ecmascript 6 on the horizon. If you get node 0.11,
you can use it server side already. Once of the big features I'm excited about 
are generators.

I've [blogged about them previously](http://blog.peterdecroos.com/blog/2014/01/22/javascript-generators-first-impressions/)
Alas, I've only found nothing out there on the web that covers anything beyond basic
instatiation and invocation. With just that to go on, it was hard for me to initially see
the hype. A few days ago, I had an appifany. 

Generators are first class objects. Like functions, they can be composed from smaller parts. Therefore,
much of we know about functions can be applied to generators!

On that note, I will lay the ground work for understanding how to really USE generators.

###It all starts with a bind

If you've worked with javascript for any of length of time, You should be familiar with bind.

{%codeblock lang:javascript %}
  var bind = function(fn, ctx, args) {
    return function() {
      return fn.apply(this, arguments);
    }
  }
{% endcodeblock %}

A function has 2 diffrent modes, literal and called. 

  * Literal: a function itself, its not being run.
    {%codeblock lang:javascript %}
var something = function() {
  console.log('do something');
}
    {% endcodeblock %}

  * Called: running the function which gives us its return value along with side effects.

{%codeblock lang:javascript %}
something();
{% endcodeblock %}

The bind is implimented by taking a literal function and calling it within another
literal function passing along the context and possible arguments using .apply().

A Generator has 3 states,
  
  * Literal: A literal Generator function

 {%codeblock lang:javascript %}
var Gen = function *() {
  var value = yield asyncTask();
  return value;
};
 {% endcodeblock %}
  
  * Instantiated: a runnable instance is created by calling the Generator function

{%codeblock lang:javascript %}
var gen = Gen();
{% endcodeblock %}

  * Run: You can then iterate through the generator by calling next()

{%codeblock lang:javascript %}
gen.next();
{% endcodeblock %}

Unlike a function, We are going to compose generators to be run inside of a co() function.
Co will run a generator until it comes accross a yield. Whatever is on
right side of the yield will be passed into co. the generator will be frozen until the
value can be resolved. This includes thunks, promises and even other generators!

A generator equivilent for a bind looks like this.
{%codeblock lang:javascript %}
var bind = function(genFunc, ctx) {
  return function *() {
    return yield genFunc.apply(ctx, arguments);
  };
};
{% endcodeblock %}

Like functions, generators also have a call() and apply() methods which can be used
to invoke the function with an explicit context. The yield is there because when we 
run this new function inside co(), the instantiated generator will be run and the
return value will be be spat out to be returned by this generator.

With that in mind, How about a function that takes two generators and runs one inside
the other? How would we impliment that?


{%codeblock lang:javascript %}
var join = function(gen1, gen2) {
  return function *() {
    return yield gen1.call(this, gen2.call(this);
  };
};
{% endcodeblock %}

With this function in place, we can now run two generators back to back inside 
a single coroutiine function.

{%codeblock lang:javascript %}
var co = require('co');
co(join(function *(next) {
  var foo = yield next;
  console.log(foo); // => 'hello world';
  return foo;
}, function *() {
  return 'hello world';
})).call(this);
{% endcodeblock %}

If you are interested in exploring this subject further, I'm working on a javacript
library for composing generators called [Shen](https://github.com/cultofmetatron/Shen).
Its a toolkit for composing generators for running inside co like lego pieces.

To give you a taste of its power, Instead of join,
[Shen](https://github.com/cultofmetatron/Shen) implements cascade which allows you to 
merge 1 or more generators.

{%codeblock lang:javascript %}
var shen = require('shen');
co(shen.cascade(
  function *(next) {
//each yield freezes the generator until the next returns.
    console.log(1);
    yield next;
    console.log(1);
    return;
  },
  function *(next) {
    console.log(2);
    yield next;
    console.log(2);
    return;
  },
  shen.cascade(
    function *(next) {
      //thats right, you can nest them too!
      console.log(3);
      return yield next
    },
    function *() {
      return
  })))();

/* Outputs
    1
    2
    3
    2
    1
  */


{% endcodeblock %}


Shen functions all compose with each other allowing you to put together generators
as easily as you would curry a function.

In addition to cascade and bind, Shen also currently includes...

  * branch and dispatch for conditional logic
  * delay for... delaying?
  * parallel: run several generators and get back an array of the return values
  * oscillator: run a generator at a specific interval and get back an immediate event emitter 
  that fires with the latest returned value of the generator

Current use-cases off the top of my head include using oscillator and parrallel to run several 
network requests at the same time. you'd get an event emitter with all the returned values in
one place. One thing to note is that you can't completely escape callbacks but you can create
areas in your code where callbacks are invisible. The generator takes care of the hard stuff.

Its still very much a work in progress and only works on node 0.11 but I invite you all to try it out. 
If you want to help, I'm always welcome to new ideas for pieces to add to the ecosystem. 
contributing a few tests or implementations of ideas would be great too!

[Here's the github to the project](https://github.com/cultofmetatron/Shen) 


Cheers.

