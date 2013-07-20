---
layout: post
title: "How to lexically scope like a boss"
date: 2013-07-16 22:27
comments: true
categories: javascript scopong 
---

Lexical scoping is probably one of the most powerful features a programming
language can have. Today, I'm going to show you how to use lexical scoping to 
create powerful abstractions. I will begin by introducing the concept of scope. 
From there I will introduce a few rules for figuing out the scope of your variables
in javascript and then introduce the concept of a closure and how it will make you
positivly badass at javascript.

##what is a scope?

I'm assuming you have written code before. Hell, I'd dare assume you've written a *var.*

{% codeblock lang:javascript %}
var variableName = "variable value";
{% endcodeblock %}

What does it mean to use the *var* keyword? As you may know, good javascript has you 
prepend all your variables with var. Most will also advocate wrapping all your code in
an immediatly invoked function.

{% codeblock lang:javascript %}
//this is being declared in the global scope.
//its functionally equivilant to window.foo or just plain foo
var foo = "super bar";

(function() {
  var foo = "inner bar";
  console.log('this foo refers to the foo inside this function ', foo);
})();
/* I don't always create inline annonymous functions, but when I do, I
immediatly invoke them. Live dynamically my friends <3 */

console.log('this one will print the original number ', foo);

{% endcodeblock %}

lets start with this relativly simple example to begin growing some understanding.
Internally scopes are represented using a tree(in simple cases, jsut a linked list). 
At each node in the tree is a dictionary of values that we refer to using the variable 
names. When we refer to foo, the interpreter (or the compiler at compile time) will look for
it in the most immediate scope. If it doesn't find a value there, it will look up the scope
tree at each point until it finds a node with a value and uses it for the current instruction.

So lets look at that previous code sample. If there wasn't a *var foo* at the top of the function,
The interpreter would look for scope[foo] and find that it isn't there. It would then look for an
entry named *foo* in the current scope's parent. In this case, thats the global scope where
there is an entry foo and it refers to the value "super bar."

##creating scopes and determining the current scope's parent.
The rules for creating scopes differ between languages. In Javascript, we start new scopes with a
function.

{% codeblock lang:javascript %}
var foo = "global scope";
var goo = "ber";
(function() {
  var foo = "inner scope";
  (function() {
    var foo = "super inner scope bro!";
  })();
})();
{% endcodeblock%}

This creates a tree with three nodes we can visualize represent
in pseudo coffeescript.

{% codeblock lang:coffeescript %}
 scope_chain = global:
                  variables:
                    foo: "global scope";
                    goo: "ber"
                  children:
                    inner_scope:
                      variables:
                        foo: "inner scope"
                      children:
                        super_inner_scope:
                          variables:
                            foo : "super inner scope bro!"
{% endcodeblock%}

As you can see, we have three nodes and there are variables that are available 
at each node. If you try to access a variable. The interpreter will look up the nodes
until it finds an entry that matches.

Lets go deeper. How about we not even bother with immediatly invoking the function.
How would that work?

The simple answer is that it changes nothing. The scope chain rules are consistent. The only difference
is that we now have two ways variables can enter the scope created in this inner function and we
delay the instantiation of the scopes till later.

{% codeblock lang:javascript %}
  //scope A
  var binder = function(st) {
    //scope B
    var state = st
    return {
      set: function(newState) {
        //scope C
        state = newState;
        return state;
      },
      get: function() {
        //scope E
        return state;
      }
    }
  }
{% endcodeblock%}

Scope order is created Lexically. Each new function definition in the source (ie: text, hence the lexical) 
is being read into the interpreter as a place to mark a new node in the scope chain on invocation. 
I mentioned that the scope datastructure resembles a tree because both Scope C and E 
belong to B as siblings. B in turn, belongs to A.

None of these functions are being invoked. The scope is locked to A -> B, B-> C, B->E
Things get hairier when we invoke them.

{% codeblock lang:javascript %}
  //scope A
  var b1 = binder(5);
  b1.get();   // => 5
  b1.set(2); // => 2
  b1.get(); // => 2

{% endcodeblock%}

How on earth is this function maintaining state? Well its quite elementary.
the Binder, on invocation, takes the argument 5 and creates a scope. According to the
rules set forward in the source code of binder. Now 2 is in Scope B and being asigned to 
the variable "state" we have instantiated in scope B. We then return the object back which 
has two functions and thus two child scopes of B which we return back to Scope A. 

Ok now its getting weird. So now when we call b1.get(), we invoking a function from scope A
which internally has a scope C and returns a value from scope B back into scope A. Think of it 
like a closed loop of scopes. ie: a closure. ;)

This is pretty powerful. b1.set() does something similar only its injecting a variable from scope
A containing 2 into scope E where it is assigned to a variable in scope B and subsuquently returned. 
B1.get(), on its next invocation returns the same value stored in that variable from the same scope
into scope A, the top level context.

This is pretty damn powerful. We can use this to create objects with completely encapsulated state
which gives us a leg up when trying to wrangle asynchrounous processes.

To give an example, here's an example of a function that runs the function after a delay.
in the meantime, we can register functions to be called when the function is resolved.

{% codeblock lang:javascript %}

  //scope A
  var delay = function(fn, args, timeout) {
    //Scope B
    var status = "pending"; //how we track what the status of the internal state is
    var result; //where we are going to store the result of calling fn.
    var deps = []; //where we store registered callbacks.
    
    setTimeout(function() {
      //Scope C
      /* the scope chain from here is A -> B-> C
      var result = fn.apply(this, args); //call the function
      status = "done";
      //we run through all the functions in deps and run them one
      //by one.
      deps.forEach(function(dep) {
        //Scope D
        dep.call(this, result);
      });
    }, timeout);

    //the two objects being returned have access to this scope
    //which allows us to invoke them to affect this internal 
    //environmental.
    return {
      done: function(func) {
        //Scope E
        //A -> B -> C
        if (status === "pending") {
          deps.push(func);
        }
        if (status === "done") {
          func(result);
        }
      }
    }
  }

  
  var promise = delay(function(msg) {
    //Scope F
    return msg + " world";
  }, ["hello"], 3000);

  //by passing this function to done, you guarantee that
  //when the function passed into delay is called, it will pass
  //the result into the function passed in here.
  promise.done(function(result) {
    //Scope G
    console.log(result);
  });

  //3000 milliseconds later
  //> "hello world"
{% endcodeblock %}




Yea its a wee bit contrived but it illustrates a point. By using a closure 
and wrapping a bunch of data into this enclosed space, we can hide away some 
pretty sophisticated machinery that lets us invert the responsibilty of control. Now the 
object maintains the state of the async call to the file and we register functions 
that it will call when the async call is resolved.

On to something a bit more powerful. Traditionally, in node, we pass a callback into
the third paramater 

{% codeblock lang:javascript %}
var fileObject = function(fileName, encoding) {
  var fs = require('fs');
  var status = "pending";
  var result_data;
  var result_error;
  var deps_success = [];
  var deps_fail = [];
  var deps_always = [];

  encod = encoding || 'utf8';

  fs.readFile(fileName, encoding ,function(err, data) {
    result_data = data;
    result_error = err;
    if (!result_error) {
      deps_success.forEach(function(fn) {
        fn(result_data);
      });
      status = "success";
    } else {
      deps_fail.forEach(function(fn) {
        fn(result_error);
      });
      status = "error";
    }
    deps_always.forEach(function(fn) {
      fn();
    });

  });

  var queueFunction = function(list, fn) {
    list.push(fn);
  };


  var done = function(fn) {
    if (status === "done") {
      fn(result_data);
      return done;
    }
    if (status === 'pending') {
      queueFunction(deps_success, fn);
      return done;
    }
  };

  var fail = function(fn) {
    if (status === "error") {
      fn(result_error);
    }
    if (status === "pending") {
      queueFunction(deps_fail, fn);
    }
  };

  var always = function(fn) {
    if (status === "pending") {
      queueFunction(deps_always, fn);
    } else {
      fn();
    }
  };

  var ret = {
    done: done,
    fail: fail,
    always: always
  };

  //this exposes the status as a readonly property
  Object.defineProperty(ret, 'status', {
    get: function() {
      return status;
    },
    enumerable: true
  });

  return ret;
};
{% endcodeblock %}




{% codeblock lang:javascript %}

var newFile = fileObject('./hello.txt');

newFile.done(function(data) {
  console.log('two ' + data);
});

newFile.done(function(data) {
  console.log('one ' + data);
});

newFile.fail(function(err) {
  console.log('the file errored out. returned: ' + err);
});

newFile.always(function() {
  console.log('this always gets called');
});

{% endcodeblock %}



