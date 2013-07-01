---
layout: post
title: "Backbone, a short primer - Part 1: Models and Events"
date: 2013-06-02 14:42
comments: true
categories: javascript backbone.js models mvc
---

Its been a few months since I've started using backbone on my personal projects.
Its a great library for rolling out MVC structure into your application but the
learning curve is pretty brutal.

The first step to really understanding the backbone.js framework.

###There is no framework

Seriously, get it out of your head entirely that backbone is a frameowrk. Its far
too minimal. Backbone is more like a toolkit library for contructing MVC frameworks.


Backbone provides the following objects

  * [Backbone.Model](http://backbonejs.org/#Model)
  * [Backbone.Collection](http://backbonejs.org/#Collection)
  * [Backbone.View](http://backbonejs.org/#View)
  * [Backbone.Router](http://backbonejs.org/#Router)

The real key is how we use it. Like Chess, it can be
learned very quickly. the problem is that you also need to understand the tectics and strategies.
The offical docs leave alot of that to the imagination.

###Backbone.Model

Backbone.Model is a storage container where we can add and remove 
items via *set* and *get* attributes.

{% codeblock lang:javascript %}

  var newModel = new Backbone.Model();
  newModel.set('foo', bar);
  newModel.get('foo') // => 'bar'

{% endcodeblock %}

You may be wondering why you don't use *newModel.foo = 'bar'?*
The real power in backbone is in the 
events that models can fire. By having you access the model's attributes via
*set* and *get*, you ensure that the associated callbacks get fired everytime
an attribute is changed.

For example, in a View containing a Model,
when we change an attribute on the model via the setter and getter methods.
A callback automatically makes the model emit a 'change' event which we can set 
the view to listen to and trigger a rerender.


####Extending Backbone.Model

Backbone.Model is inherited using extend. This is also where we add methods
that may be specific to our particular subclass of Backbone.Model. For instance, 
lets create a Comments model which may contain a username, content, rating 
and timestamp.

{% codeblock lang:javascript %}
  var Comment = Backbone.Model.extend({
    //defaults are self explanatory
    defaults: {
      rating: 0,
    },
    /*
     * initialize runs whenever new is called, it takes care of
     * setting up __super in the inheritance chain
     * there is also 'constructor' which overides the contructor entirely
     * leaving you to manually impliment the prototype chaining.
     * ie: only use constructor if you know what you are doing.
    */
    initialize: function() {
      this.set('owner', getCurrentUser());
      this.set('timestamp', new Date());
    },
    upvote: function() {
      this.set('rating', this.get('rating')++);
      return this;
    }
  });

  /* now we initialize the model passing in values in an object. */
    var newComment = new Comment({
    content: "backbone needs a library for reactive databindings",
  });

  newComment.upvote().get('rating'); // => 1

{% endcodeblock %}

Here we've set up a basic backbone model for setting representing a comment.
In defaults, we put in an object of default attributes. You may be wondering why I
set up the owner and timestamp attributes in *initialize.*

The defaults object and anything inside it is evaluated into a static object
inside the extend function call. Thus, if you were to try this

{% codeblock lang:javascript %}
 var Comment = Backbone.Model.extend({
    //defaults are self explanatory
    defaults: {
      rating    : 0,
      owner     : getCurrentUser();
      timestamp : (function() { return new Date() })();
    }
    ...
 })
{% endcodeblock %}

you would find that every instance will have the same timestamp. To get around this,
I have the functionality for those defaults set to run in initialize.

####Backbone.Model events

Backbone models should only broadcast events in order to notify things that
contain it. A model itself can contain another model as an attribute. 

Backbone.Model's 'change' event fires when it detects a change in the value of the attribute.
If the value is a model, then a change in that model's attributes won't change the reference
to the model. Unlike the dom, we have to explicitly set up our own event bubbling.

To set up your own event delegation, you have to set up a listener in the parent model to listen for
events in the attribute model.

{% codeblock lang:javascript %}

  var CommentHolder = Backbone.Model.extend({
    initialize: function() {
      /* We set up event listeners here
        the first argument is message being fired
        the second is the function to be invoked
        the third is the context for function to be called in
      */
      this.get('comment').on('all', this.bubble, this);
    },
    bubble: function() {
      // apply is used to propegate all possible arguments
      // that can be coming form multiple events.
      this.trigger.apply(this, arguments); //trigger is a backbone function
    }
  })

{% endcodeblock %}

['all'](http://backbonejs.org/#Events-catalog) is a backbone defined catch all event 
that sends along the name of the event as the first argument. The model will listen
to its comment and broadcast any messages it recieves to any who would hear.



