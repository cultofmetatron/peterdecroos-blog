---
layout: post
title: "Thorax is Awesome"
date: 2013-07-05 13:45
comments: true
categories: backbone thorax defineProperty
---

I like Backbone.js alot. it's a fantastic utlity library for building your own
best of breed javascript applications.Its much more configurable than angular.js 
or ember.js. (both of which make alot of assumptions about the nature of your app 
which may or may not lead to alot of pain when working with a legacy codebase.) while
you certainly won't be cranking out apps as quickly as you would in more magical frameworksm
Backbone gives you enough to get started but stays slim in features to avoid stepping 
on your toes. Backbone is the fixed gear bike of frontend javascript frameworks.

While I reccomend anyone new to backbone build a few projects to learn it. In practice,
I don't reccomend using backbone solely if you are building out a full mvc system. Its missing
a few really important features that you would have to implement yourself.

## Enter Thorax

In the world of backbone extensions, Marionette, Thorax, and Chaplain. I haven't looked at
chapplain and I found Marionette a lil heavy for my needs. Thorax on the other hand was a
breath of fresh air. One big pro is I found was its built in integration of 
[handlebars.js](http://handlebarsjs.com) templates.

####Out of the box render() that works

Backbone.View's default render method is a noop(), (ie: function() {}).
Backbone's author's intend for you to write your own render function which
sets its instance's *el* property to your view's generated represntation of 
the model's data. Backbone avoids adding in this feature to keep it agnostic to
your templating system.

Since Thorax makes the decision of using handlebars, it can give us a default render()
method thats usable. Instead of a noop(), Thorax.View.render() by default, constucts a
constext object containing all the *properties* of the view instance and the attributes
of the model.

{% codeblock lang:javascript %}

//render function, this is what a Thorax render does, 
// *this* resolves to the thorax instance

var render = function() {
  //get the attributes of the property
  var properties = {};
  for ( var property in this) {
    properties[property] = this[property];
  };
  //underscore.js's extend
  if (this.model) {
    properties = _.extend(instanceProperties, this.model.attributes);
  };
  this.$el.html(this.template(properties));
  return this;
}
{% endcodeblock %}

By default, properties of the view insance get passed to the template 
but not the functions. What if we want to overide this?

In the past, I would write view helpers in order to render attributes that required computation.
There's a more elegant way using the *defineProperty* function.

{% codeblock lang:javascript %}
  var peter = new Thorax.Model({
    firstName: 'Peter',
    lastName:  'de Croos'
  });

  var PersonView = Thorax.View.extend({
    template: Handlebars.compile(TemplateSource),
    initialize: function() {
      Object.defineProperty(this, 'fullname', {
        get: function() {
          return this.model.get('firstName')+ ' ' + this.model.get('lastName');
        },
        enumerable: true
      });
    }
  });

  var peterView = new PersonView({
    model: peter
  });
{% endcodeblock %}

By default, properties defined by *defineProperty* are not enumerable. 

IE: they won't show up when you do the *for (property in this) {}*, Since
thats how thorax's render view gets at the attributes, it won't show up in the template 
either. Luckily that is easy enough to fix by specifying the option
*enumerable:true*

now we have a property we can expose in the template and when handlebars accesses that property, 
the fullName function will be accessible in your handlebars templates as a read-only property 
without having to register a helper.

####Child view management

Backbone does nothing to address embedded views. It expects you to write your own logic
for handling child views into the render. As you can imagine, its kind of a pain.
It sounds simple enough in practice but it means that you have to write in logic to check
if a parent view is being removed and recursivly have it descend into all it child views to 
undelegate events from the dom. This is a major potential source of javascript memory leaks.
Thorax provides some great helper functions for this. It just wins.

Thorax.Views can be embedded in another view simply by adding it as a property of 









