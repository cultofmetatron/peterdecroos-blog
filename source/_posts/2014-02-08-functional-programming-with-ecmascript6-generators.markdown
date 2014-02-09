---
layout: post
title: "Functional programming with ecmascript6 generators"
date: 2014-02-08 19:12
comments: true
categories: functional-programming ecmascript6 javascript generators
---

The web is abuzz right now with ecmascript 6 on the horizon. If you get not 0.11,
you can use them already server side. Once of the big features I'm excited about 
are generators.

I've [blogged about them previously](http://blog.peterdecroos.com/blog/2014/01/22/javascript-generators-first-impressions/)
Alas, I've only found nothing out there on the web that covers anything beyond basic
instatiation and invocation. With just that to go on, it was hard for me to initially see
the hype. A few days ago, I had an appifany. 

Generators are first class objects and like functions, they can be composed from smaller parts. Therefore,
much of we know about functions can be applied to generators!

On that note, I will lay the ground work for understanding how to really USE generators.

###It all starts with a bind

If you've worked with javascript for any of length of time, You should be familiar with bind.

{%codeblock lang:javascript %}
  var bind = function(fn, ctx, args) {
    args = Array.prototype.slice.apply(arguments, 2);
    return function() {
      args2 = Array.prototype.slice.apply(arguments);
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
  1. Literal: A literal Generator function
 {%codeblock lang:javascript %}
var Gen = function *() {
  var value = yield asyncTask();
  return value;
};
 {% endcodeblock %}
  2. Instantiated: a runnable instance is created by calling the Generator function
 {%codeblock lang:javascript %}
var gen = Gen();
 {% endcodeblock %}

  3. Run: You can then iterate through the generator by calling next()
 {%codeblock lang:javascript %}
gen.next();
 {% endcodeblock %}













