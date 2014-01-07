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
I've solved architectural problems in contructing a large scale backbone framework.

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

Of course there's a major component missing. Any good backbone framework 
has a way of embedding subViews. Turns out its sort of tricky.

The Lifecycle of a Backbone view in this case is to listen to changes on 
model and call render() which updates the view's $el property. Thus its 
impossible to proceed until this.$el is completely demystified.

####what is $el?

I've heard alot of confusion about the nature of 'this.$el'. Typically, a Backbone
View is a controller for a node. This node is a structure relevant to the
document object model.

you may be familiar with a dom object if you've used jquery.

{% codeblock lang:javascript %}
  var $body = $('body'); // => returns a jquery wrapped object
  $body[0] //returns the wrapped object representing the body dom node.
{% endcodeblock %}

In much the same way, a backbone view is a controller for a dom object.

You may have seen the kickstart for a backbone app that involves doing a jquery
lookup on a node and setting its html to your view's $el property

{% codeblock lang:javascript %}
  var $container =  $('.container');
  $container.html(view.render().$el);
{% endcodeblock %}

Assuming the render() method of the view returns 'this', then calling it before calling the
$el property guarantees that the dom object managed by the view will be updated before being 
placed in the dom. When render() is called again, it updates the *contents* of that $el. The
$container has a reference to $el stored in it thus ensuring that calling render() will change
the webpage to reflect the latest state of the model.

####Extending out Render() to support subViews.

With the afformentioned definition of $el in mind, The best way to embed a subview 
(with string based template engines, you do something else with dom based ones.) is to do an 
initial render of the dom node marking off somehow places to embed subviews. Then afterwards,
go back through the $el and systematically embed $el for each subview in their respective places.

{% codeblock lang:javascript %}
  //updated render()
  this.$el.html('');
  var context = {};
  context._subView = function(viewName) {
    return '<div class="subView view-' + viewName + '"></div>';
  };
  if (this.model) {
    _.extend(context, this.model.attributes);
  }
  //pop it in the dom
  this.$el.html(this.template(context));
  //notice we do this AFTER we rerender the new this.$el
  var subViews = this.subViews;
  this.$el.find('.subView').each(_.bind(function(index, el) {
    var $el = $(el);
    // view-
    var subView = _(Array.prototype.slice.call(el.classList)).chain()
      .filter(function(className, index) {
        return (className.match(/^view-/));
      }, this)
      .map(function(className) {
        return className.slice(5);
      }, this)
      .value()[0]; //grab the first item
    $el.html(subViews[subView].render().$el);
  }, this));

  return this;
{% endcodeblock %}

There's a new things we've added to this version. 

  1. There is now a 'subView()' helper being passed into context, this will generate
  the slug that we will look for when embedding a subview.
  2. After the $el is rendered, we are searching for dom elements with class subview
  3. for each subView, we find the name of the subview we embed in there given as
  view-[viewName] and look it up in the view's subViews key/val lookup object. 
  Its a property of teh parent view.

With this, we now have an easy way of supporting subviews.

{% codeblock lang:javascript %}
  childView = new application.BaseView({
    tpl: 'child'
  });

  var parentView = new application.BaseView({
    tpl: 'parent'
    subViews: {
      child: childView
    }
  });

  $('div.container').html(parentView.render().$el);

{% endcodeblock %}

In parentView's template, its as simple as using out new subView helper

{% codeblock lang:html %}
  <script type="text/template" date-name="parent">
    <p>
      <%= _subView('child') %>
    </p>
  </script>

  <script type="text/template" date-name="child">
    <span> hi I'm the child view!! </span>
  </script>
{% endcodeblock %}

In the next installment in this series, we'll build a base ControllerView class which we
can use to render generic collections. Yes, they'll be embedable as subviews in our BaseView.



