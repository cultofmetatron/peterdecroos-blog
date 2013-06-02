---
layout: post
title: "Backbone, a short primer"
date: 2013-06-02 14:42
comments: true
categories: javascript backbone.js mvc
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

Ok, so thats pretty much backbone. The real key is how we use it. Like Chess, it can be
learned very quickly. the problem is that you also need to understand the tectics and strategies.
The offical docs leave alot of that to the imagination.

###Backbone.Model

Backbone.Model is a storage container where we can add and remove items via *set* and *get* attributes

{% codeblock lang:javascript %}

  var newModel = new Backbone.Model();
  newModel.set('foo', bar);
  newModel.get('foo') // => 'bar'

{% endcodeblock %}

You may be wondering, "why don't I jsut use newModel.foo = 'bar'?"  The real power in backbone is
the set of events models can fire. By having you access the model's attributes via *set* and *get*, you
ensure that the associated callbacks get fired everytime an attribute is changed.

####Extending Backbone.Model

Backbone.Model is inherited using extend. This is also where we add methods that may be specific
to our particular subclass of Backbone.Model. For instance, lets create a Comments model which may contain a
username, content, rating and timestamp.

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
     * ie: only use contryctor if you know what you are doing.
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










