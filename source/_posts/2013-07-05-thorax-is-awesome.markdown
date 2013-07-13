---
layout: post
title: "Thorax is Awesome"
date: 2013-07-05 13:45
comments: true
categories: backbone thorax defineProperty
---

I like Backbone.js a lot. It's a fantastic utlity library for building your own
best of breed javascript applications. It is much more configurable than Angular 
or Ember - both of which make numerous assumptions about the nature of your app,
which may or may not lead to frustration when working with a legacy codebase. While
you certainly won't be cranking out apps as quickly as you would in more magical frameworks,
Backbone gives you enough to get started. It stays slim in features to avoid stepping 
on your toes. Backbone is the fixed gear bike of frontend javascript frameworks.

I recommend anyone new to Backbone build a few projects to learn it. In practice,
I don't recommend using Backbone alone for building out a full mvc system. Its missing
a few really important features that you would have to implement yourself.

## Enter Thorax

In the world of Backbone extensions, there are three major players, Marionette, Thorax, and 
Chaplain. I haven't looked at Chaplain and I found Marionette heavy for my needs.
Thorax on the other hand, was a breath of fresh air. I especially loved the
[handlebars.js](http://handlebarsjs.com) templates integration.

In the following posts, I will outline some of the major benefits of using Thorax.

  * Out of the box render() that works
  * Child view management
  * Layout Views
  * The MVC in Backbone's MVC

####Out of the box render() that works

Backbone.View's default render method is a noop(), (ie: function() {}).
Backbone's author's intend for you to write your own render function which
sets its instance's *el* property to your view's generated representation of
the model's data. Backbone avoids adding in this feature to keep it agnostic to
your templating system.

> Note: Handlebars
>
> Handlebars is an html templating language for javascript.
> 

{% codeblock lang:javascript %}

<h1>{% raw %}{{ value }}{% endraw %}</h1>
<p>
  {% raw %}{{#if foo}}{{foobar}}{{/if}}>{% endraw %}.
</p>
{% endcodeblock%}

###Handlebars
This handlebars code is fed into Handlebars.compile() which returns a template function.
This function is then passed a "context" object. example: 

 {% codeblock lang:javascript %}
 
    { 
      value: "hello there" 
      foo: true,
      foobar: "hello Douglas"
    }

{% endcodeblock %}

and generates a string of html that you can inject into the dom.


{% codeblock lang:html %}

  <h1>hello there</h1>
  <p>hello Douglas</p>

{% endcodeblock %}

Since Thorax makes the decision of using Handlebars, it can give us a default render()
method that's usable rather than a noop(). Thorax.View.render() constucts a
context object containing all of the *properties* of the view instance and the attributes
of the model which is passed into the Handlebars template function so that it's available
in the rendered view.

{% codeblock lang:javascript %}

//render function, this is what a Thorax render does, 
// *this* resolves to the thorax instance

var render = function() {
  //get the attributes of the property
  var viewProperties = {};
  for ( var property in this) {
    viewProperties[property] = this[property];
  };
  //underscore.js's extend
  if (this.model) {
    viewProperties = _.extend(viewProperties, this.model.attributes);
  };
  this.$el.html(this.template(viewProperties));
  return this;
}
{% endcodeblock %}

By default, properties of the view insance get passed to the template 
but not the functions. What if we want to overide this?

In the past, I would write view helpers in order to render attributes that required computation.
There's a more elegant way using the 
[defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) function.

{% codeblock lang:javascript %}
  var peter = new Thorax.Model({
    firstName: 'Peter',
    lastName:  'de Croos',
    githubHandle: 'Cultofmetatron'
  });

  var PersonView = Thorax.View.extend({
    template:
      Handlebars.compile("we can call
               {% raw %}{{firstName}}, 
               {{lastName}} and 
               {{fullName}}{% endraw %}"),
    initialize: function() {
      Object.defineProperty(this, 'fullName', {
        get: function() {
          return this.model.get('firstName') + ' ' +
          this.model.get('lastName');
        },
        enumerable: true // see note below
      });
    }
  });

  var peterView = new PersonView({
    model: peter
  });
{% endcodeblock %}


> *Technical note:* By default, properties defined by *defineProperty* are not enumerable. 
>
> IE: They won't show up when you do the *for (property in this) {}*. Because
> thats how thorax's render view gets at the attributes, it won't show up in the template 
> either. Luckily that is easy enough to fix by specifying the option
> *enumerable:true*




####Child view management

Backbone does nothing to address embedded views. It expects you to write your own logic
for handling child views in the render function. As you can imagine, it is kind of a pain.
It sounds simple enough in theory, but in practice it means that you have to write code to check
if a parent view is being removed and recursivly have it descend into all it child views to
undelegate events from the dom. This is a major potential source of javascript memory leaks.
Thorax provides some great helper functions for this. It just wins.

Thorax.Views can be embedded in another view simply by adding the subview as a property.

{% codeblock lang:javascript %}
  
  var blogModel = Thorax.Model({
    name: 'console.log(this.thought)',
    post: new Thorax.Model({
      title: "Thorax is awesome",
      content: "maximum callstack exeeded"
    })
  });

  var PostView = Thorax.View.extend({
    template: Handlebars.compile('{% raw %}<h2>{{title}}</h2><p>{{content}}</p>{% endraw %}')
  });

  var blogView = new BlogView({
    template: Handlebars.compile("{% raw %}<h1>{{name}}</h1>{{view postView}}{% endraw %}"),
    model: blogModel,
    postView: new PostView({
      model: this.model.get('post')
    })
  });
{% endcodeblock %}

As you can see, it was as easy as embedding {% raw %} {{view postView }}{% endraw %}
inside the template and Thorax takes of care of yielding control of that region to the
child view.

####Layout Views

Let me just say that layout views are really awesome. A layout view is 
a view with the sole purpose being a container which can hold whatever 
view/model combo fits the current context.


#####The MVC in Backbone's MVC

There's a Backbone.Model and a Backbone.View but where is Backbone.Controller? (hint: there isn't one)

Currently Marrionette augments Backbone with a Controller object but
Thorax does nothing of the sort. I guess the real question is, where does a controller fit?

Philosphically, MVC stands for a seperation of concerns between

  1. Models: the data containing the application logic we are modelling on the computer
  2. Views: Objects that manage the presentation of the models. This includes binding 
event handlers and rerendering when the underlying model(s) change.
  3. Controllers: Objects that take care of managing what view and model are relevant.

In the Rails world, the architectural philosphy revolves around the concept
of skinny controller/fat Model. The brunt of the code for manipulating the models should 
be inside the instance methods in the models themeselves and the controller interacts with it
through an api. 

When we really get down to it, controllers perform a few functions.

  1. Watch out for some event that should trigger a change in view and/or model.
  2. Manage the creation of new view instances to represent new or altered models.
  3. Render html and insert it into the DOM.

Our controller will handle only the details of getting the proper model and binding 
it to a view so that it can be rendered.

Lets create a function PostController that does nothing more than lookup a
model by its id and creates a post view.

{% codeblock lang:javascript %}
var postController = function(postId) {
  //for info on Thorax.Views, Thorax.Models and Thorax.Collections,
  //see the note below
  var postView = new Thorax.Views.FullPagePostView({
    model: Thorax.Collections.posts.get(postId)
  });
  return postView;
}
{% endcodeblock %}

>  **Note** About Thorax Registries
>
>  Thorax provides a Registry: a series of hashes 
>  where you can store the Constructor Functions for your 
>  extended models and views.
>
>  For more information visit [The throax docs](http://thoraxjs.org/api.html#registry)

Now we have a basic controller that handles cases 2 and 3 of our list. We still need to take care
of figuring out how we want to determine when this controller gets called. In the projects
I have worked on, I've solved this problem using the router.

Thorax does nothing special with the router so we will use Backbone.Router as is. It will manage one 
Layout view, a special Thorax view made for holding other views.

{% codeblock lang:javascript %}
//router.js
var layoutContainer;
var callController = function(controller) {
  return function() {
    //arguments is used to shift responsibilty of knowing the 
    //amount of paramaters to the controller
    var view = controller.apply(this, arguments);
    layoutContainer.setView(view);
  };
};
{% endcodeblock %}

The function *callController* takes a controller, passes along
the arguments given to it by the router and takes care of the
boiler plate of creating a view and setting it into the container.
*layoutContainer.setView()* takes care of undelegating events of
all elements attached to the previous view (if there was a previous view) 
as it swaps in the html of the new view.

{% codeblock lang:javascript %}
//continued from above
var Router = Backbone.Router.extend({
  initialize: function() {
    layoutContainer = new Thorax.LayoutView({
      template: '#AppLayout'
    });
    /*
      this is the point in the html where our container will be 
      injected in. from here on out, the router and controller
      system will take care of swappng out application views 
      within this container.
    */
    layoutContainer.appendTo('#entry-point');
    $(document).trigger('popstate');
  },
  routes: {
    'posts/:postId': 'postRoute'
  },
  postRoute: callController(postController)
});

{% endcodeblock %}

This is a pretty straight forward setup with one possible new thing. So you 
probably noticed the *$(document).trigger('popstate')*. In standard backbone,
the application instance Router has a method *navigate()* usually called by

{% codeblock lang:javascript %}
  router.navigate('path/to/route', { trigger: true });
{% endcodeblock %}

The trigger true is needed by backbone for the router to trigger any associated actions.
The browser by default writes to history when the document recieves the
[popstate](https://developer.mozilla.org/en-US/docs/Web/Reference/Events/popstate) event.
The browser triggers popstate automatically when you enter a page or hit the backwards or forward buttons.
When you click back, the new url will load and the router which is listening
for the popstate event on 'document' will perform the behavior associated with that url.

By having the *$(document).trigger('popstate')* at the end of the initialization, we guarantee 
that once the router is finished initializing, it will read in the url and trigger the appropriate
context for the app for the url. This is great if we want multiple url entry points through
which the user enters the app. The user gets the same javascript no matter what url of the 
website they visit. The app takes care of loading the right content based on the url.

Finally, the template option that was passed into the instantiation of layoutView is optional.
By default, Thorax wraps the layout in a div tag. We can customize the layoutView by giving it a template 
and using the layout-element helper in that template.

{% codeblock lang:html %}
<script id="appLayout" type="handlebars/template">
  {% raw %} {{layout-element tag="div" id="currentContext" class="container"}} {% endraw %}
</script>
{% endcodeblock %}
