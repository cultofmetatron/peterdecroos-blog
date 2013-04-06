---
layout: post
title: "On this, protototypes, and dunderheads"
date: 2013-03-31 23:34
comments: true
categories: Javascript, prototypes
---


Of all the stranger than fiction things I have seen, I can't say I have experienced as blatant
a miscarriage of conceptual understanding as I have with javascript's object system. (Ok, I might
be exagerating just a little...)

Javascript is an interesting language. It is quite powerful when paired with appropriate libraries
like [underscore](http://underscorejs.org/). Its prototypical inheritence model is a mindblowlingly powerful reductionist critique
of classical inheritance. In the right hands, functions become lego pieces that can be glued onto objects
in staggeringly flexible ways.

### On "this"

Lets take an object

{% codeblock lang:javascript %}
  var myObject = {};
{% endcodeblock %}

This is a basic object. It is also a hash.

{% codeblock lang:javascript %}

  //We can access it via dot notation or hash notation
  myObject['jackson'] = 5;
  5 + myObject['jackson']; // => 10;
  5 + myObject.jackson;    // => 10;

  //we can store strings, numbers, other objects and very importantly,
  // other functions!
  myObject.title = function() {
    return "this is the Jackson " + this.5;
  }

  myObject.title() // => "this is the Jackson 5";

{% endcodeblock %}

If you are currently a javascript programmer, you've probably seen code like this. In
langages like Java, or c, Functions are simply routines that operate on code. they are static.
Once they exist, they exist in only one context. The Java version of *'this'* refers to the instance
of the instantiated object.

Functions in javascript are "First Class."
This is a really important distinction to make because it enables all of the most
powerful code reuse techniques. A function can exist beyond an object. It can be made an
instance function a several different objects. In practice, it means we can assign functions to
variables and pass it into functions as arguments. We can even return functions from other functions.

Put more succinctly...

We can create a function which performs operations related to an abstract "this" attribute without knowing what "this" is going to be till we hook up a context to it when its called.

It is important to discuss the notion of call time.  Languages where functions are not first class
do not have a call time.

Lets take a cannonical example in Java.


{% codeblock lang:java %}
  public class ExampleClass {

    public int foo;

    public static void main(String[] args) {
        System.out.println("Clojure is way more fun");
    }

    public ExampleClass(int foo) {
        // initialize code goes here
        this.foo = foo;
    }
    public getFoo() {
        return this.foo;
    }
  }
{% endcodeblock %}

Java functions are not first class. The *'this'* refers only to the instantiated instance of
this class. If I instantiate this class and call the function, I will get the
class variable "foo" of the instance.

{% codeblock lang:java %}
  ExampleClass exampleClass = new ExampleClass(5); // Java is ridiculous!!!
  exampleClass.getFoo(); // => 5

   // this results in an error because there is no public variable getFoo
  exampleClass.getFoo;

{% endcodeblock %}

Because Java's functions are not first class, there is no notion of this function being
referred to unless its being specifically called. Javascript's functions are dfferent. I can declare
a javascript function and bind it to several objects.

{% codeblock lang:javascript %}
  var getFoo = function() {
    return this.foo;
  };

  var first = { foo:1 };
  first.getFoo = getFoo;

  var second = { foo: 2 };
  second.getFoo = getFoo;

  var third = {foo:3};
  third.theNameIsIrrelevant= getFoo;

  first.getFoo() // => 1
  second.getFoo() // => 2
  third.theNameIsIrrelevant() // => 3

{% endcodeblock %}

This code in javascript works because javascript functions are variables that can be assigned and passed
around like a company pen. (thats about as work safe a metaphor as I could come up with. sorry...) The only
large difference between functions and other javascript types is that functions can be *called.*

Call time is the point where the javascript function is executed. If it contains a *'this'* inside it. That keyword then
resolves to whatever object that function is being called upon. If there is no object, *'this'* resolves
to the global object. In Browsers, the global object is *window.*


{% codeblock lang:javascript %}
  window.foo = "ramalamadingdong";

  getFoo(); // =>  "ramalamadingdong"

{% endcodeblock %}

### on __proto__ and prototype

*new* is a feature of javascript that changes the context of *'this'* in
a constructor function. when new is called on a constructor function being called, *new*
becomes an object that is being modified and eventually returned. It will delegate function calls it
cannot respond to to whatever the function's prototype attribute is. IE: functions in javasript are
objects.

{% codeblock lang:javascript %}
  var dundar = {
    gummy:'bear'
  }

  var ContructorFunction = function() {
    // at this point this == {};
    this.foo = "bar"

    // "return this" is implicit

  }

  ContructorFunction.prototype = dundar;

  newObject = new ContructorFunction();

  newObject.gummy; // => 'bear'
  newObject.foo; // => 'bar'

{% endcodeblock %}

### The \_\_proto\_\_ Attribute

Javascript's prototype delegation is setup such that if an attribute is accessed on an object and the
object does not have that attribute. It will defer that request to the object set as its \_\_proto\_\_.
The \_\_proto\_\_ is determined by the constructor function's *prototype* attribute.

####This begs the question? why doesn't this work?

{% codeblock lang:javascript %}

  var dundar = {
    gummy:'bear'
  }

  newObject = { foo: 'bar' };
  newObject.__proto__ = dundar;

{% endcodeblock %}

####Actually it DOES, But only in V8 - ie: chrome and node.js

(as an interesting aside, this technique is used alot in the express source code. Don't try this at home,
unless you do server side node at home of course!)

Sorry to burst your bubble but I must advise you that you do not do this on the browser as it will not work
becase \_\_proto\_\_ is a protected attribute. Thats right, the song and dance above it using *new* is
the only surefire way to assign the \_\_proto\_\_ attribute.



### call() and apply()

To recap

Unless you call the function with new, *'this'* becomes resolved within the function to whatever is on
the left side of the '.'.

{% codeblock lang:javascript %}
  // if 'this' is in the definition of getFoo, it will resolve to myObject.
  myObject.getFoo()
{% endcodeblock %}

call() and apply() can be called on any function and allow you to explicitly set what *'this'* will
resolve to. the only difference is that call takes the arguments of the function right after the new
meaning of *'this'* and apply takes the arguments as an array.

{% codeblock lang:javascript %}

  myObject1 = {
    foo: 'bar',
    num: 1
  }

  myObject2 = {
    foo: 'manshu',
    getFoo : function() {
      return this.foo;
    }
    inc: function(num) {
      this.num += num;
      return this.num;
    }
  }

  //errors out because it doesn't have a getFoo() method
  myObject1.getFoo();

  myObject2.getFoo() // => 'manshu'

  //heres where is gets interesting
  myObject2.getsFoo.call(myObject1) // => 'bar'

  //this one errors because myObject2 does not have a num attribute
  myObject2.inc() // badd

  myObject2.inc.call(myObject1, 1); //sets myObject1.num to 2

  //apply does the same with the arguments in an array
  myObject2.inc.apply(myObject1, [1]); // myObject1.num is now 3!!

{% endcodeblock %}



Now we're ready for how to create inheritence chains in javascript!
(that article is comming soon.)



