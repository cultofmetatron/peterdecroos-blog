---
layout: post
title: "lets build a backbone based framework!!"
date: 2014-01-06 13:57
comments: true
categories: backbone javascript
---

I've been building large scale applications in Backbone for about 8 months now.
In that time I've used throax as well as building custom solutions in backbone. 

Last night, I live coded the creation of a demo backbone framework similar in features 
to Thorax. I'm going to walk you through how 
to show you some of the architectural problems I've solved in contructing 
a large scale framework.

[Fork or watch it here](https://github.com/cultofmetatron/backbone-framework-example)

When building a large scale Backbone, there are several consderations to consider.

  * How are we going to have templates load? Inline html declarations in the views do not scale
  * how do we handle render?
  * how do we handle child views?
  
By default, backboneView.render() is a noop. Its meant to be overidden. Backbone.Events gives us
a nice set of methods to allow Backbone objects to listenTo() events off each other, However, the 
actual bindings and callbacks are left to the implimenter.

###Loading Templates

the first thing is figure out how we want the templates to load. If using a tool like require.js 
or browserify, there are plugins for automatically compiling templates and sending to the client 
just a compiled template function we can plug into out view. Since I'm trying to keep it simple, 
the easiest way to get templates is to wrap your templates in a script tag. To prevent the browser 
from executing it as javascript, we add a type that is not 'text/javascript'.

I like to use "text/template."

{% codeblock lang:html %}
  <script type="text/template" data-name="templateName">
    <!-- template contents goes here -->
  </script>
{% endcodeblock%}

Transforming these into working templates is fairly straightforward. For each
script tag with type="text/template", run the template compiler over the html inside and
store it somewhere. In this case I add a data-name attribute which will be the key for the
hash I store the compiled template in.

[/public/javascripts/init.js](https://github.com/cultofmetatron/backbone-framework-example/blob/master/public/javascripts/init.js)
{% codeblock lang:javascript %}
  window.templates = {};
  var $sources = $('script[type="text/template"]');
  $sources.each(function(index, el) {
    var $el = $(el);
    templates[$el.data('name')] = _.template($el.html());
  });
{% endcodeblock %}

In this version, I chose to use express + jade, Of course writing underscore templates
in jade seemed a little odd so I delegated those to jade's include statement. I know there
are projects like jade-browserify so maybe I'll eventually update this using just jade.

[/views/layout.jade](https://github.com/cultofmetatron/backbone-framework-example/blob/master/views/layout.jade)


{% codeblock lang:jade %}
doctype html
html
  head
    title= title
    link( rel="stylesheet", href='/components/bootstrap/dist/css/bootstrap.css')
    link( rel="stylesheet", href='/stylesheets/style.css')
    script( src="/components/jquery/jquery.js")
    script( src="/components/bootstrap/dist/js/bootstrap.js")
    script( src="/components/underscore/underscore.js")
    script( src="/components/backbone/backbone.js")
    script( src="/javascripts/base.js")
    script( src="/javascripts/application.js")
  body
    block content
    script(type="text/template", data-name="main")
      include templates/main.html
    script(type="text/template", data-name="header")
      include templates/header.html
    script(type="text/template", data-name="menuLinks")
      include templates/menulink.html
    script( src="/javascripts/init.js") 
{% endcodeblock %}

The actual contents of the underscore templates in the *.html templates will 
now be compiled and loaded into "templates[data-name]" where I can access it later.


### Creating our own Base Backbone Objects.

The first thing we want to do is determie the shared behavior of backbone objects
in our app. For special cases we can overide them later. However, for the sake of time, we
want some decalrative way of telling the object what template to use and a default render()
method that we can use in most situations.

{% codeblock lang:javascript %}
  //first we declare a namespace to store these new Objects
  application = {};
  application.BaseView = Backbone.View.extend({
    initialize: function() {
      Backbone.View.prototype.initialize.apply(this, arguments);
      if (this.model) {
        this.listenTo(this.model, 'change', this.render);
      }
      /*
        if the BaseView is extended with a tpl string, we want to 
        have that be available but we also want to be able to load 
        it at runtime and have it overide the base tpl string
      */
      this.tpl = options.tpl || this.tpl
      this.loadTemplate(this.tpl);
    }
    loadTemplate: function(tpl) {
      /*  
        if tpl is a function, we assign it directly, 
        otherwise, if its a string, we look it up
        in our templates map.
      */
      if (_.isFunction(tpl)) {
        this.template = tpl;
      } else if (_.isString(tpl)) {
        this.template = template[tpl];
      } else {
        throw new Error('tpl must be a function or a string!');
      }
    }
    render: function() {
      //see below
    }
  });
{% endcodeblock %}

Next up we need to create a render function. Its arbitrary based on the particular
needs of your application. For most purposes however, Its sufficiant to make a
view's model's attributes available inside the templates.

{% codeblock lang:javascript %}
  //from the render in the preceeding example
  render: function() {
    this.$el.html('');//empty the view's node
    var context = {}; //create a context object
    if (this.model) {
      _.extend(context, this.model.attributes);
    }
    this.$el.html(this.template(context));
    return this;
  }
{% endcodeblock %}

Here's a great first start. Now any View that extends BaseView will have a
render. The only interface we must follow is that the View have a model 
and a tpl which tells the View how to resolve this.template.

Of course there's a major component missing. Any good framework has a system 
for embedding subViews. Turns out its sort of tricky.

The Lifecycle of a Backbone app in this case is to listen to changes on 



