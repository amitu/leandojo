# Dojo For JQuery Developers

In this tutorial we will implement a feature with dojo. The feature is
Kudos, and it is live at the bottom of the page. It shows how many
"kudos" a page has got.

We will start from scratch, I am assuming you have never used dojo
before, but have reasonable understanding of javascript and jquery.

## Basic Setup

In a jquery based project, you include jquery in the page as the first
thing, typicall from Google's CDN.

The story is a little more complicated than this in dojo world, as dojo
is AMD based. What it means is dojo is split into 100s of small files as
individual "modules", and based on the features the project requires
dojo fetches and loads the modules.

This means instead of one HTTP request, potentially 100s of requests
could be made for your project. Which sounds bad, but it is not, because
there is a dojo build process that combines and optimizes all those 100s
of files into one or few files, which is used for production, and the
build thus produced only contains the bit of dojo and even your
javascript that you actually use.

This allows developers to create rich libraries without worrying about
code size. But all this consideration makes dojo seem more complex than
jquery.

The good news is that it is still not more complex than the jquery use
case, as dojo comes with pre built package that contains "most" of dojo
features you will need in your project.

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello Dojo</title>
    </head>
    <body>
        <!-- our page -->
        <script
            data-dojo-config="async: true"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            /* our code */
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

You will see some funny attributes like dojo-\* a lot in dojo. These
attributes are compatible and HTML5 recommended way for passing data
from HTML to javascript.

In this example we have data-dojo-config, which is a way to configure
dojo.  Dojo can be configured in more than one way. Here is another way
to do the same thing.

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello Dojo</title>
    </head>
    <body>
        <!-- our page -->
        <script>
            // this block must come before the script that loads dojo
            // dojoConfig variable is used to configure dojo
            var dojoConfig = { async: true };
       </script>
        <script src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            /* our code */
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

Here we are configuring dojo to run in **async mode**. Dojo in is a
transition phase, AMD feature was introduced relatively recently in
dojo's history, and async mode tells dojo to act in AMD mode. The
default value of this is false to maintain compatibility with code
written prior to AMD. In dojo 2.0, this will be the default. 

## Hello World with Dojo

Among the first things one does in jquery is wait for DOM to load. Lets
see how it is done in dojo:

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello Dojo</title>
    </head>
    <body>
        <script
            data-dojo-config="async: true"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function dom_is_ready() 
            {
                alert("Hello World, the DOM is ready!");
            }

            function modules_loaded(domReady) 
            {
                domReady(dom_is_ready);
            }

            require(["dojo/domReady"], modules_loaded);
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

I will explain this code below, first lets talk about AMD.

## Dojo Kernel and AMD basics

jQuery is available as $. Most jquery plugin go on $.{name space}, or
$.fn.{name space}. And all these libraries are loaded and available
before our code starts executing. Eg:

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>jQuery Hello World</title>
    </head>
    <body>
        <script src="jquery.js"></script>
        <script src="jquery.helloworld.js"></script>
        <script>
            // by the time we are here, both $ and $.helloworld are available
            $(function(){
                $.helloworld();
            })
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

On the other hand Dojo kernal only provides two functionality, an
ability to define modules and an ability to require modules.

What is a module? Modules have a name, the name of a module is also
called its MID, module ID. If a module is defined in
somefolder/somefile.js, the name or MID of the module is
"somefolder/somefile". MID directly maps to file name.

The **require** method takes an array of modules that are required and
(an optional) callback method. **require** function will try to load all
required modules, some modules may already be loaded, in which case it
will use it, and for the modules that are not yet loaded, dojo will
fetch the javascript file, and use it. Once all dependencies have been
"resolved", the callback will be called with each module as function
parameter to callback.

What does a module contain? No module should touch globals, instead each
module should be an object. The object could be a function, or it can be
any other javascript object, string, array, etc. The javascript of the
module must use **define** to inform dojo kernel what should be
considered the "content" of the module. Dojo then keeps a mapping of
module name, MID with the "content" of the module, so a module can be
required by any number of other modules without any performance penalty.
We will look into how to define our own modules later.

Please note that for all practical purpose any dojo functionality other
than dojo's **require** and **define** is at the same level as our user
code. You may use dojo's kernel without using any dojo modules at all.

One of the uses of dojo-config is to tell dojo how to map a module name
to the actual HTTP URL of javascript file that defines the module. While
ideally all modules would already be part of dojo build, in some cases
we may want to not load all functionality in a single file. We will
discuss it later, suffice to say, dojo has thought about all use cases
and has it covered for you.

## Lets understand the hello world above

The crucial bit is this:

~~~~~~~~~~~~~~~
<script>
    function dom_is_ready() 
    {
        // this callback will be called by domReady when the DOM is actually
        // ready. in some case DOM could become ready before the module is 
        // available, in which case this callback will be called as soon as
        // the module becomes available.

        alert("Hello World, the DOM ready!");
    }

    function modules_loaded(domReady) 
    {
        // this will be called when require has loaded "dojo/domReady"
        // either internally if it was baked in, or from HTTP if it was not
        // the argument passed is the "content" of the module, which happens
        // to be a function in case of "dojo/domReady"

        domReady(dom_is_ready);
    }
    
    // by this time we have only access to require function by dojo
    // and two of our methods defined above.

    require(["dojo/domReady"], modules_loaded);

    // in other words

    // my dear dojo,
    // please load the AMD module named "dojo/domReady" for me and when it is 
    // available for use, call the function I am passing to you as second 
    // parameter. 

    // thank you very much. 

    // PS: please also be kind enough to pass the "content" of AMD module as an
    // argument to my callback method, as you always do.
</script>
~~~~~~~~~~~~~~~

Unlike jQuery with one callback for DOM ready, we have two callbacks
here.

We start by calling the **require** method discussed above, with a list
of modules we are interested in, in this case only "dojo/domReady", and
with a callback *modules_loaded* that will be called when the module is
available.

When the script starts executing "dojo/domReady" module may or may not
be loaded. In this case, because we are using dojo.js from Google CDN,
that happened to have "dojo/domReady" baked in, it does not mean every
dojo.js will have it.

## Lets play with DOM

Now that we have some notion of what AMD is, and we have the DOM
available to us, lets do something with it.

I contend that the alert does not look nice, its too needy, we better
replace it with something else.

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello Dojo</title>
    </head>
    <body>
        <script
            data-dojo-config="async: true"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function modules_loaded(domReady, domConstruct) 
            {
                function dom_is_ready() 
                {
                    // we have declared this function inside modules_loaded
                    // so that is has access to domConstruct variale via
                    // closure

                    var h1 = domConstruct.create("h1", {
                        innerHTML: "Hello World, the DOM ready!"
                    });
                    domConstruct.place(h1, document.body);
                }

                domReady(dom_is_ready);
            }

            require(["dojo/domReady", "dojo/dom-construct"], modules_loaded);
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

Now we are using list of two AMD modules as first parameter to call to
dojo's **require** method. Both of them happen to be provided by dojo,
and are baked in Google's CDN version of dojo.js, so our callback
function modules_loaded would be called right away, and will be passed
two parameters, and we are accepting them with the arguments named
domReady and domConstruct. There is no restriction on what we named
those parameters and we could have called them anything, though having
sane name helps. Dojo's build process will optimize the javascript by
renaming those variables to single letter ones for example.

One of the first thing you will notice is that, yes, domConstruct means
the module only deals with constructing a DOM node (and destruction of
them). You have a different AMD module in dojo that is used for styling
("dojo/dom-style") them, manipulating class of DOM ("dojo/dom-class"),
and so on. And there is a different module for findind DOM nodes,
"dojo/query", and a set of modules to manipule them,
"dojo/NodeList-manipulate", travese them, "dojo/NodeList-traverse" and
so on.

This might put off some jquery developers, but in practice it is a good
idea for dojo to have such fine grained functionality, so you do not
have to load the bits you do not need, and at the same time it is very
easy to write a module that combines all these modules and give a dojo
on steriods for DOM. One such project is
[plugd](https://github.com/amitu/plugd).

## Where we are so far

So you are basically getting a hang of AMD, you have seen a two AMD
modules provided by dojo in action, and have heard about a few others.
You may not like the verbosity of dojo so far, but lets accept it for
the time being and look at what dojo provides that jquery does not.

Lets try to learn more by working on a concrete problem

## Problem: Kudos for this blog

Kudos: 

> [Dustin Kurtis](http://dustincurtis.com/) came up with an excellent
> idea for the [Svbtle](http://svbtle.com/) blogging network (go visit
> [Svbtle.com](http://svbtle.com/) to see an example). He called them
> Kudos.  They're little widgets next to each post that enable users to
> give "Kudos" to posts they really like. You hover over the widget, it
> gives a fun little animation, and changes the icon and count after a
> moment.

Source: [masukomi/kudos](https://github.com/masukomi/kudos).

A working demo at the bottom of this post.

Implementation details: We are going to use Parse.com as a "backend" to
store the data. One of the reasons I want to with Parse for this example
is this:

> As more logic is moved from the server to the browser, a client-side
> application can get arbitrarily complicated. This situation makes it
> essential to come up with an intuitive decomposition of the problem
> into smaller and understandable pieces matching your mental model of
> application architecture.

Quote:
[sirprize](http://sirprize.me/scribble/dojorama-introduction-building-a-dojo-single-page-application/)

And don't worry, there would hardly be a few lines of codes dealing with
parse, so you won't have to learn both dojo and parse in the same time.

jQuery way of doing things was good when server did everything and the
client did a little bit of progressive enhancements. With passing time
clients kept doing more and more, and with time we see that jquery on
itself is not adequate, and there is a flurry of new age libraries,
Backbone, AngularJS, EmberJS and so on. I hope to demonstrate that dojo
does, not only what jquery does, but also makes all these other
libraries irrelevant. 

The simplicity of jquery seduces developers into declaring it far
superior to dojo, but then they have to end up with a solution patched
with Backbone etc, and almost none of them match the quality that dojo
provides, and not only that, but it also means every team is reinventing
new ways to do the same things.

What we will build would be a "widget". Lets get to it.

## Kudos Widget

What is a "widget"? A button is a "widget", a span or a div can be
called a "widget", and the compose dialog of GMail is a "widget".

In dojo world, widget is something much more concretely defined, widget
is a "subclass" of "dijit/\_WidgetBase". "dijit/\_widgetBase" is a
module, whose content happens to be a "class". 

So what is a "class"? One of the reason that backs the claims I made in
previous section is that dojo provides a unified class system. Sure
javascript is object oriented, but subclassing a class is a mess, so
much so that almost everyone does it their own way. 

Dojo provides a class system based on "dojo/\_base/declare" module, with
support of subclassing, multiple inheritance, method over riding and
calling parent methods and so forth.

Coming back to Kudos widget, we will be creating KudosWidget class,
which will be a subclass of "dijit/\_WidgetBase". Why should it be
subclass of \_WidgetBase?

Not only that unifies how things are done, \_WidgetBase is also a
subclass of "dojo/Stateful", which means it is quite easy to use them in
a MVC like application by directly connecting them to a "dojo/Store"
based storage (which happens to be Backbone like architecture, only
saner).

On top of all this, dojo also has a "dojo/parser", the parser can be
made to parse the HTML page, find DOM nodes marked with special tags and
convert them into "live" widgets. If that reminds you of AngularJS, then
you are spot on.  (Though not as live as angularjs, but then its only a
matter of time before someone changes that).

So what will be kudos widget? Kudos widget will be available as
KudosWidget, and to use it in HTML like this:

~~~~~~~~~~~~~~~
<figure data-dojo-type="KudosWidget"></figure>
~~~~~~~~~~~~~~~

Lets see how it is done.

## Our first widget!

Lets look at the most basic widget we can create in dojo.

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello Widget</title></head>
    <body>
        <script
            data-dojo-config="async: true"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function modules_loaded(domReady, declare, _WidgetBase) 
            {
                // declare is used to create a new class.
                // in this case we want to subclass _WidgetBase so we pass it as
                // first arguments. if we were creating a new class with no 
                // parent we would have passed null, and in case we wanted to 
                // have multiple parents, we would have passed a list of parent
                // classes as first argument

                // we are storing the newly concstructed *class* in a variable

                var HelloWidget = declare(_WidgetBase, {

                    // this second object is used to define the body of the class.

                    // since we are subclassing _WidgetBase, we already have
                    // access to this.domNode, which usually is considered the
                    // root node of each widget. its a *very* strong convention
                    // to have a single root node of each widget. 

                    // our widget has one member "message". please note that
                    // even if we do not specify this member here, and if message
                    // is passed in constructor, it would still be available to
                    // us in the class as this.message. the value passed in 
                    // constructor will override the value specified here.

                    message: "default message",

                    // _WidgetBase.postCreate method is called once the widget
                    // has been constructed for us. in this case we have not
                    // informed dojo how we want the widget to be constructed
                    // so dojo defaults to constructing a DIV DOM node for us.
                    // there are other ways to construct the DOM of widget that
                    // we will see later. its a good practice to do all event
                    // binding etc for the DOM of widget in this postCreate()
                    // instead of while constructing the DOM, as in future you
                    // might switch from one way of constructing the DOM to 
                    // another

                    postCreate: function() {

                        // when we are overriding a method and if we want to 
                        // call the parents version of the method, this is how 
                        // you do it.

                        // whether or not you want to do it depends on your 
                        // logic, this "feature" is available to us because
                        // we are using declare, and is not specific to 
                        // _WidgetBase. 

                        // parents version of postCreate does nothing, I am 
                        // keeping it here for demonstration only, and its a 
                        // good practice, tomorrow you might inherit from 
                        // something else.

                        this.inherited(arguments);

                        // _WidgetBase has constructed this.domNode for us.
                        // this.message is available to us because we defined
                        // it above.

                        this.domNode.innerHTML = this.message;
                    }
                });

                // now we create an instance of our class. 
                // even if in the class definition we do not say explicitly that
                // class has a member named "message", dojo will still create 
                // the member for us because its passed in constructor and
                // make it available as this.message

                // please also note that this way 

                var widget = new HelloWidget({
                    message: "Widget says hello!"
                });

                // we should call startup on each widget, even tho our widget does
                // not actually do anything. depending on the logic of widget
                // you may want to call startup after it has been instantiated
                // or after it has been placed in the document. if you want to
                // fetch something off net, start as soon as possible, if you 
                // want to do animations, start when added to document.

                widget.startup();

                // we have access to our widget instance, but we can not place
                // it in the DOM as it may not be ready yet. 

                function dom_is_ready() 
                {

                    // finally now the DOM is ready and we place our widget
                    // in the body

                    widget.placeAt(document.body);
                }

                domReady(dom_is_ready);
            }

            require(
                ["dojo/domReady", "dojo/_base/declare", "dijit/_WidgetBase"], 
                modules_loaded
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

Go ahead, copy paste it in a html file, make sure to serve it using http
server instead of directly opening in browser via file:/// path, as that
is not supported in dojo if not all modules are baked in a single file.

If you notice Network tab in Developer Console in Chrome lets say, you
will see that that number of network request made has jumped to 7. Till
the previous example we saw, the number of network requests was 2, one
for HTML page and other for dojo.js. dojo.js frovided by Google's CDN
included the AMD modules we requested for manipulating DOM, but not all
the modules needed in this case.

Don't worry about this, by the time we are done with it we will create a
custom build of dojo that will include everything we want, and the
number of network requests would be back to 2.

Aside: If you are curious about constructor of a Class which is not
derived from \_WidgetBase, this is how you will do it.

~~~~~~~~~~~~~~~
var MyClass = declare(null, {
    constructor: function(x, y) {
        this.z = x + y;
    }
});
~~~~~~~~~~~~~~~

And if you want to "automatically make things passed to constructor to
become available to class":

~~~~~~~~~~~~~~~
var MyClass = declare(null, {
    constructor: function(args) {
        declare.safeMixin(this, args);
        this.z = this.x + this.y;
    }
});

var c = new MyClass({x: 10, y: 20})
// c.z == 30
~~~~~~~~~~~~~~~

In general you should probably always define the default value for
parameters you accept in constructor.

## Lets use our widget "declaratively"

The code we saw above used our widget "programmatically". The other way
to use a widget is "declaratively". 

So what is programmatically exactly?

~~~~~~~~~~~~~~~
// in the example we created the instance of our widget ourselves in program
var widget = new HelloWidget({
    message: "Widget says hello!"
});

// we started it ourselves in program
widget.startup();

// and we placed it programmatically
widget.placeAt(document.body);
~~~~~~~~~~~~~~~

There is another way to do it:

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello Widget</title></head>
    <body>
        <!-- look at me here! -->
        <div 
            data-dojo-type="HelloWidget" 
            data-dojo-props="message: 'I am declarative'">
        </div>
        <script
            data-dojo-config="async: true"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function modules_loaded(domReady, declare, _WidgetBase, parser) 
            {

                // this time HellowWidget is globally available
                // global variables are usually bad, we will do it better later

                window.HelloWidget = declare(_WidgetBase, {
                    message: "hello world",
                    postCreate: function() {
                        this.inherited(arguments);
                        this.domNode.innerHTML = this.message;
                    }
                });

                function dom_is_ready() 
                {
                    // this is where the "magic" happens. "dojo/parser" module
                    // has a method parse(), that can traverse the entire DOM
                    // and find any DOM node that has special dojo specific
                    // attributes and do special magic with them.

                    // parse can optionally take a DOM node as parameter and only
                    // parse that node. if nothing is passed it parses the whole
                    // document.body.

                    // dojo also has parseOnLoad configuration parameter, but 
                    // its better to be more explicit about when to call parse

                    parser.parse();
                }

                domReady(dom_is_ready);
            }

            require(
                [
                    "dojo/domReady", "dojo/_base/declare", "dijit/_WidgetBase",
                    "dojo/parser"
                ], 
                modules_loaded
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

We have the same widget, but this time its globally accessible. We are
now using "dojo/parser" and asking it to parse the entire body searching
for DOM nodes tagged with special attributes.

Here is the our DOM node with special attributes:

~~~~~~~~~~~~~~~
<div data-dojo-type="HelloWidget" data-dojo-props="message: 'I am declarative'">
</div>
~~~~~~~~~~~~~~~

data-dojo-type is special to parser, and parser assumes that if this is
present you are trying to treat this DOM node as a widget.
data-dojo-props is optional, and if it is present, its content is passed
to the constructor of the widget.

The parser is also smart enought to convert the values in
data-dojo-props to right type, if it can guess the type of each member
variable of the class based on its default values. In our case dojo
knows HelloWidget's .message is of type string and would have tried to
converted the value to string if it was not. This can be handy when
dealing with dates for example.

Also note that when we constructed the widget instance programmatically,
dojo used DIV as the type of DOM node to construct. But if you attach
the data-dojo-type="HelloWidget" to SPAN, or any other tag, dojo will
use that node as the .domNode root for the widget. Dojo will leave the
DOM tree intact for widget to "enhance" in whatever way one wants.

If you are wondering how to get a handle over those widgets, lets say
you want to do something with them after they have initialized? You can
add data-dojo-id="someid" to them, and dojo comes with a registry,
"diji/registry", that contains a registry.byId() method that will return
the instance of widget by its data-dojo-id attribute. You can also use
registry.byNode() if you have access to DOM node that has been converted
to widget by parser. 

Further, and this may be a little bit controversial, if the DOM node
that was converted to widget had an HTML attribute id set on it, dojo's
parser will make the widget available globally by that id. Eg &lt;a
data-dojo-type="..." id="foo"/&gt; will be converted to right widget and
that widget instance will become available through window.foo. 

If either of data-dojo-id or id on a DOM node to be converted to widget
is not unique, dojo gets really upset, so be careful.

## Two kinds of widgets

In dojo you will see primarily two kinds of widgets. 

First kind of widget works with existing DOM tree, and is usually used
to enhance it in some fashion, connect some event handler, overtake a
form and provide validation for it etc. The widget we saw was of this
nature. For such widgets .postCreate is the place where all the action
happens.

The second kind of widgets create their own DOM tree. For example a
calendar widget may take an input, and convert it to a complicated DIV
based DOM tree which shows a full calendar when focussed. 

We have already seen how we can create new DOM nodes using
"dojo/dom-construct" module. But in practice it gets tedious fast to
programmatically create whole DOM heirarchy. In case we want to do it,
for example if the heirarchy we want is not that difficult or for any
other reason, we should do that by subclass .buildRendering() method.

The better way to build DOM tree us using "dijit/\_TemplatedMixin"
module. Lets see how it works.

## Templated Widgets

Dojo actually comes with two template libraries, the first one is dojo's
custom template, and then they support Django's Template Lanugage based
tempaltes. In our KudosWidget we will be using dojo's main template
engine not django one.

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello Widget</title></head>
    <body>
        <div 
            data-dojo-type="HelloWidget" 
            data-dojo-props="message: 'I am declarative'">
        </div>
        <script
            data-dojo-config="async: true"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function modules_loaded(domReady, declare, _WidgetBase, _TemplatedMixin, parser) 
            {
                // we are going to subclass more than one class, so first 
                // argument is now a list instead of class like before.

                HelloWidget = declare([_WidgetBase, _TemplatedMixin], {

                    // this is how _TemplatedMixin works, it assumes either
                    // this.template or this.templateString is available, 
                    // if this.template is available, it assumes that it is the
                    // URL from where the template can be obtained, and if 
                    // this.templateString is present, its value is directly
                    // used for template.

                    // the template will be used to construct a DOM tree, and 
                    // will be *appended* to domNode (existing elements will 
                    // remain as it is)

                    // it is also possible for template to specify a DOM element
                    // under which all existing children of domNode should be
                    // reparented to. by using data-dojo-attach-point="containerNode"
                    // its an advance topic, and this paragraph can be ignored,
                    // just that this feature may come handy one day.

                    // now this may sound a little bad, you can not hard
                    // code more complex template in a single line like below,
                    // but you do not want to make extra HTTP requests to load
                    // templates if you have a lot of widgets.

                    // dojo build system again comes to rescue, and we will look
                    // at how to get dojo to populate tempalteString on build
                    // time based on content of html file.

                    // note the special syntax ${message}, it will be evaluated
                    // to this.message. 

                    // one very important point to note: the template must 
                    // contain only one root level node. 

                    // one can also attach event handlers to DOM nodes by using 
                    // the attribute data-dojo-attach-event="click:clicked", in 
                    // this case this.clicked would be used as click handler

                    templateString: '<table><th><td>Message:</td><td data-dojo-attach-point="mnode">${message}</td></th></table>',

                    postCreate: function() {

                        // since we set data-dojo-attach-point attribute on one
                        // DOM node in template, we have access to this.mnode, 
                        // which points to the DOM node on which the attribute 
                        // was added

                        this.mnode.style.color = "red";
                    }
                });

                function dom_is_ready() 
                {
                    parser.parse();
                }

                domReady(dom_is_ready);
            }

            require(
                [
                    "dojo/domReady", "dojo/_base/declare", "dijit/_WidgetBase", 
                    "dijit/_TemplatedMixin", "dojo/parser"
                ], 
                modules_loaded
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

## Configuring dojo

Lets start with configuring things first. So far we have inline
javascript, now we will create our own AMD module. During development we
want to not bother with dojo's build, so we will have to configure dojo
where our AMD modules are going to be.

So what do we have to configure? We want dojo to look for Google's CDN
for any modules that is part of dojo, if it is not baked in dojo.js. And
we want dojo to load modules loaded from /dkudos/ if they are part of
dkudos library.

As it turns out its not that difficult, I have added "paths" setting to
config as you can see below, it is reasonably self explainatory, any
MID, AMD module name, if it starts with dkudos, dojo will look for under
/dkudos, rest of them it will look on a path relative to dojo.js, which
is on Google's CDN.

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello AMD!</title></head>
    <body>
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            ...
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

As you can see understanding the concept may require some effort, but
actually doing it is a breeze.

## Our first AMD module!

So we want to remove our HelloWidget and place it in its own module:
"dkudos/HelloWidget". This is how our new HTML will look like:

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello AMD!</title></head>
    <body>
        <div 
            data-dojo-type="dkudos/HelloWidget" 
            data-dojo-props="message: 'I am from AMD!'">
        </div>
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function modules_loaded(domReady, parser)
            {
                domReady(function(){
                    parser.parse();                    
                });
            }
            require(
                ["dojo/domReady", "dojo/parser", "dkudos/HelloWidget"], 
                modules_loaded
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

As you can see the data-dojo-type now points to "dkudos/HelloWidget"
instead of "HelloWidget". You will also see that we have added
"dkudos/HelloWidget" module in **require** call. 

Dojo parser can possibly automatically load the widget while parsing if
it is not already loaded, but it is considered a bad idea. It is better
to be explicit about things, because dojo build system only inspects the
list of modules passed in the require calls, and does not parse all HTML
to find the modules that may be automatically loaded. For this reason
always explicitly require each module instead of letting dojo's parser
autoload a module.

Further you may notice that we have not captured the third parameter in
**modules_loaded** method. This is a common dojo pattern, many times we
have to load modules that we may not care to capture in callback, and
they are siltently dropped from the argument list.

And here is how our hello world widget looks like when converted to a
AMD module. Actually, no, first lets start with something simpler. 

Lets see a module "dkudos/PI". This module will simply return the value
of PI to us.

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello AMD!</title></head>
    <body>
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            require(["dkudos/PI"], function(PI) {
                console.log("PI=", PI);
            });
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~
// content of /dkudos/PI.js

define(3.14);
~~~~~~~~~~~~~~~

**define** is the other function in dojo.js. You can define anything to
be content of the module. Anything that is not a function.

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello AMD!</title></head>
    <body>
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            require(["dkudos/logPI"], function(logPI) {
                logPI();
            });
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~
// content of /dkudos/logPI.js

var PI = 3.14;

define(function(){
    // this line will be executed when the module is being loaded by dojo.
    console.log("module is getting initialized, PI=", PI); 

    return function() {
        // this is the actual "content" of the module. whoever *requires* this
        // module will get access to this anonymous function

        console.log("PI=", PI); 
    }
});
~~~~~~~~~~~~~~~

In case of function, **require** calls the function and the content of
module is whatever the function returns.

What if our module depends on functionality defined by other modules?
**define** can be called with two argument, first being a list of
modules just like **require** call, and if second paramter is a
function, that function will recieve as arguments contents of
dependencies.

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello AMD!</title></head>
    <body>
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            require(["dkudos/logPI2"], function(logPI2) {
                logPI2();
            });
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~
// content of /dkudos/logPI2.js

var PI = 3.14;

define(["dojo/string"], function(string){
    return function() {
        var templ = "The value of PI is ${thePI}.";
        console.log(string.substitute(templ, {thePI: PI})); 
    }
});
~~~~~~~~~~~~~~~

There you go, now you understand both **require** and **define**. 

We will now see the whole HelloWidget. You have already seen this is how
we will use our widget:

~~~~~~~~~~~~~~~
<html>
    <head><title>Hello AMD!</title></head>
    <body>
        <div 
            data-dojo-type="dkudos/HelloWidget" 
            data-dojo-props="message: 'I am from AMD!'">
        </div>
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            function modules_loaded(domReady, parser)
            {
                domReady(function(){
                    parser.parse();                    
                });
            }
            require(
                ["dojo/domReady", "dojo/parser", "dkudos/HelloWidget"], 
                modules_loaded
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

And this is our HelloWidget as an AMD module.

~~~~~~~~~~~~~~~
// content of /dkudos/HelloWidget.js
var  template = '<table><th><td>Message:</td><td data-dojo-attach-point="mnode">${message}</td></th></table>';

define(
    ["dojo/_base/declare", "dijit/_WidgetBase", "dijit/_TemplatedMixin"], 
    function(declare, _WidgetBase, _TemplatedMixin){

        return declare(
            [_WidgetBase, _TemplatedMixin], {
                templateString: template,

                postCreate: function() {
                    this.mnode.style.color = "red";
                }
            }
        );
    }
);
~~~~~~~~~~~~~~~

Now if you are like me, line that hardcodes HTML template in javascript
must be really hurting your eyes. Lets fix that first.

## Dojo AMD Plugins

While there are more than on way to fix it, the one we are going to see
can be argued to be the "best" one, as it plays well with dojo. 

If we do not like HTML hardcoded in javascript, it is obvious that we
will store HTML outside of javascript and in some .html file. Which
means the content of that file has to be loaded at some point. And we
can use plain vanilla AJAX, and dojo has awesome support of it since
2004, but it will mean we have to make an HTTP request to fetch the
HTML. If we did that then we will resist dividing our code in dozens of
small widgets and composing our application, as no one wants dozens of
HTTP requests.

Now dojo has a build system, and it can "bake" in all our javascript
files together and we would want something like this with our templates
too \[and eventually CSS\]. But dojo can not provide us with a module
that will do that, as the name of file will have to be passed to module
after dojo has passed it to us. Which means unless dojo does too much
magic with that module, we can not convey which file contains our HTML.
Dojo does not want to auto detect the name of HTML file from widget
name, as that would be fragile.

For this reason and for a few other use cases, dojo has a concept of
"plugins".

What are plugins? Plugins are just AMD module, just that they have a
.load() function defined. So any module can be used as a plugin, we will
see soon what it means, and if it is used as plugin, dojo will expect
that it has a .load() method defined and it will call that. 

Normally when a module is loaded as a module, it is returned as soon as
it is loaded, but when it loaded as a plugin, the .load() method of the
plugin is called, and that .load() plugin has a mechanism to delay the
execution till, lets say, it loaded something over HTTP. 

The module of interest is "dojo/text", and to instruct dojo to use it as
a plugin, we have to pass "dojo/text!" in the require list. The "!"
tells dojo that its supposed to be a plugin. Some plugins may not
require any data, but some do, and our "dojo/text!" plugin needs the
name of HTML file to load, so we have to passed that too:
"dojo/text!dkudos/HelloWidget.html". (.load() of "dojo/text" module will
be called with "dkudos/HelloWidget.html" as argument and the plugin will
return the content of HTML).

So lets look at our HelloWidget.js that uses "dojo/text!" plugin:

~~~~~~~~~~~~~~~
// content of /dkudos/HelloWidget.js

define(
    [
        "dojo/_base/declare", "dijit/_WidgetBase", "dijit/_TemplatedMixin",
        "dojo/text!dkudos/HelloWidget.html"
    ], 
    function(declare, _WidgetBase, _TemplatedMixin, template){

        return declare(
            [_WidgetBase, _TemplatedMixin], {
                templateString: template,

                postCreate: function() {
                    this.mnode.style.color = "red";
                }
            }
        );
    }
);
~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~
<table> <!-- content of "/dkudos/HelloWidget.html" -->

    <!--
        please remember that the HTML must contain just one top level node,
        not even a html comment. the parse dojo uses is too finicky.
    -->

    <th>
        <td>Message:</td><td data-dojo-attach-point="mnode">${message}</td>
    </th>
</table>
~~~~~~~~~~~~~~~

So far we have not made any dojo build, so an HTML request will be made
to load it, but during build dojo will see the "dojo/text!" and will
"bake" it in the built dojo.js and "dojo/text!" plugin will load it from
it instead of making HTTP request.

## Now to KudosWidget

Our first job is to port the [Kudus jquery plugin](
https://github.com/masukomi/kudos) to dojo.

Any page where we want to use KudosWidget will look like this:

~~~~~~~~~~~~~~~
<html>
    <head>
        ... normal head content ...
        <link href='/dkudos/KudosWidget.css' rel="stylesheet" />
        <!--
            As you can see I am keeping css separate for now. There are ways
            to avoid that, we will discuss it later during build time.
        -->
    </head>
    <body>
        ... the actual body ...
        <!-- where ever we want to place the widget -->
        <div data-dojo-type="dkudos/KudosWidget"></div>
        ... rest of body ...
        <script data-dojo-config="async: 1, paths: {dkudos: '/dkudos'}"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            require(
                ["dojo/parser", "dojo/domReady!", "dkudos/KudosWidget"],
                function (parser) {

                    // we have used domReady! too as a plugin, its .load()
                    // simply waits till DOM is ready.

                    parser.parse();                    

                    // a further "optimization" could be to use 
                    // "parseOnLoad: true" as a configuration in data-dojo-config
                    // but we will still nead the *require* call to load
                    // "dkudos/KudosWidget" module.

                }
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

The plugin requires a DOM structure like this:

~~~~~~~~~~~~~~~
<figure class="kudo kudoable" data-dojo-attach-event="mouseenter: start, mouseleave: end">
    <a class="kudobject">
        <div class="opening">
            <div class="circle">&nbsp;</div>
        </div>
    </a>
    <a href="#kudo" class="count">
        <span class="num">0</span>
        <span class="txt">Kudos</span>
    </a>
</figure>
~~~~~~~~~~~~~~~

So you know that it will go in /dkudos/KudosWidget.html. As you can see
I have use data-dojo-attach-event there, you can guess what it means
(start and end are defined on widget, they are not referring to global
functions start and end, though I guess if there were no start/end on
the widget itself, they could have referred to globally defined funtions
too). One can argue this is bad "design", and event binding should be
part of javascript, and should be done in .postCreate(), the reason I
like this is that the HTML developer can be communicated that .start and
.end exist, and (s)he can attach it to other DOM if the structure ever
changes. 

KudosWidget.js itself will look something like this:

~~~~~~~~~~~~~~~
define(
    [
        "dojo/_base/declare", "dijit/_WidgetBase", "dijit/_TemplatedMixin",
        "dojo/text!dkudos/KudosWidget.html", "dojo/dom-class", 
        "dojo/_base/lang", "dojo/on"
    ], 
    function(
        declare, _WidgetBase, _TemplatedMixin, template, domClass, lang, on
    ){

        return declare([_WidgetBase, _TemplatedMixin], {

            templateString: template,

            isKudoable: function() {
                return domClass.contains(this.domNode, "kudoable");
            },

            isKudod: function() {
                return domClass.contains(this.domNode, "complete");   
            },

            complete: function() {
                this.end();
                this.incrementCount();
                domClass.add(this.domNode, 'complete');
                this.emit("kudo_added");
            },

            unkudo: function(event) {
                event.preventDefault();
                if (this.isKudod()) {
                    this.decrementCount();
                    domClass.remove(this.domNode, 'complete');
                    domClass.remove(this.domNode, 'complete');
                    this.emit('kudo_removed');
                }
            }, 

            setCount: function(count) {
                return this.counter.innerHTML = count;
            },

            currentCount: function() {
                return parseInt(this.counter.innerHTML);
            },

            incrementCount: function() {
                return this.setCount(this.currentCount() + 1);
            }, 

            decrementCount: function() {
                return this.setCount(this.currentCount() - 1);
            },

            start: function(evt) {
                if (this.isKudoable() && !this.isKudod()) {
                    this.emit('kudo_active');
                    domClass.add(this.domNode, 'active');
                    this.timer = setTimeout(
                        lang.hitch(this, this.complete), 700
                    );
                }            
            }, 

            end: function() {
                if (this.isKudoable() && !this.isKudod()) {
                    this.emit('kudo_inactive');
                    domClass.remove(this.domNode, 'active');
                    if (this.timer != null) {
                        clearTimeout(this.timer);
                    }
                }
            },

            markDone: function() {
                domClass.remove(this.domNode, "animate");
                domClass.add(this.domNode, "complete");
            },

            postCreate: function() {
                on(this.domNode, "click", lang.hitch(this, this.unkudo));
            }
        });
    }
);
~~~~~~~~~~~~~~~

This is almost a line by line port of [the jquery plugin](
https://github.com/masukomi/kudos/blob/master/kudos.js). You will notice
that we have got "dojo/dom-class" in there, which is used to add/remove
a class from an element. You will also notice we have used .hitch()
method from the module "dojo/\_base/lang", it is used to "bind" a
function to an object so that *this* points to the right object in our
callback.

You will notice a bunch of on() and .emit() calls. Lets look briefly on
event system in dojo to understand them.

## Event management in dojo

The powerhorse of events in dojo is "dojo/on" module. "dojo/on" module's
content is a function. "dojo/on" can be used with DOM nodes, or it can
be used with objects that are instances of class inheriting
"dojo/Evented". Any javascript object can subclass itself from
"dojo/Evented" and start emitting events.  "dijit/\_WidgetBase" happen
to be a subclass of "dojo/Evented" so it can fire events too.
"dojo/Evented" subclasses have access to a methods
this.emit("event_name", event_data) and this.on("event_name", callback),
and they are shortcuts for on.emit(this, "event_name", event_data) and
on(this, "event_name", callback). 

.on() returns a handle with .remove() on it that can be used to "unbound" any 
event handler. 

on from "dojo/on" also has on.once() method that can be used just like
on(), but the event is fired only once and is .remove()ed subsequently. 

dojo also has a module "dojo/aspect" that contains aspect.before(),
aspect.after() and aspect.around() methods, that can be used to "bind"
any method of any object "before", "after" or "around". So if an object
o has .start() method on it, we can do aspect.before(o, "start",
mymethod), and then if anyone calls o.start(), our callback mymethod
would be called first, and then .start() would be called.

A good widget will define all meaningful "events" and will .emit() them
appropriately, but "dojo/aspect" lets us handle things at even "deeper"
level.

Finally dojo has a module "dojo/topic" with methods
.publish("event-name", data) and .subscribe("event-name", callback) to
be used for "global" events.

on.once(), topic.subscribe() and aspect.before/after/around() return a
handle with a .remove() on it that can be used to remove the resepctive
binding.

With that in mind we can translate the
this.element.trigger("kudo:active") like line to
this.emit("kudo_active"), or just this.emit("active") as don't really
need the name space. (dojo for some reason does not like ":" in event
names).

## Now comes the data

As you have seen, the KudosWidget that have implemented only emits the
events, and does not actually store the data anywhere. For our
application we have two bits of data to be stored, we need a mapping for
URL of the current page with total number of kudos it has recieved so
that we can show it. And we need a way to store that the current user
has already "kudod" a page.

For our case we will be storing the kudos to count mapping on Parse, and
the second mapping using LocalStorage HTML5 storage.

Now we can modify our "dkudos/KudosWidget" with a call to parse and some
calls to localstorage, and implement the functionality. But that would
be a bad design. We can subclass the widgets to become aware of the
data, and fundamentally that approach is actually not that bad. Lets
take a look at how it will be done. 

~~~~~~~~~~~~~~~
    <head>
        ... normal head content ...
        <link href='/dkudos/KudosWidget.css' rel="stylesheet" />
    </head>
    <body>
        ... the actual body ...
        <!-- we are using ParseKudosWidget now -->
        <div data-dojo-type="dkudos/ParseKudosWidget"></div>
        ... rest of body ...
        <script>
            var dojoConfig = {
                async: true,
                paths: {
                    "dkudos": "/dkudos",
                    // for localstorage handing we are going to use this
                    "dojo-local-storage": "https://raw.github.com/sirprize/dojo-local-storage/master"
                }
            }
        </script>
        <script src="http://www.parsecdn.com/js/parse-1.2.7.min.js"></script>
        <script src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            // initialize parse
            Parse.initialize(
                "R4itZTuvbWZ1PHI5RCu2LKkqMaMl5QFVZLzPyeYM", // app id
                "52jAgPNq3iHJitnsqkfB8atmhXOZ5aqFTQPZKF39"  // javascript key
            );

            require(
                ["dojo/parser", "dojo/domReady!", "dkudos/ParseKudosWidget"],
                function (parser) {
                    parser.parse();
                }
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

And the javascript.

~~~~~~~~~~~~~~~
// content of /dkudos/ParseKudosWidget.js
define(
    [
        "dojo/_base/declare", "dkudos/KudosWidget", "dojo/_base/lang",
        "dojo-local-storage/LocalStorage"
    ], 
    function(declare, KudosWidget, lang, LocalStorage) {

        return declare([KudosWidget], {

            url: document.location.pathname,
            pkudo: null, // this will store parse's kudo related data for url
            Kudos: Parse.Object.extend("Kudos"),
            local_store: new LocalStorage({idProperty: "url"}),

            init_pkudo: function() {
                this.pkudo = new this.Kudos();
                this.pkudo.set("url", this.url);
                this.pkudo.set("score", 0);
                this.pkudo.save();
            },

            save_pkudo: function() {
                this.pkudo.save(null, {
                    success: lang.hitch(this, function(k) {
                        this.setCount(k.get("score"));
                    })
                });
            },

            kudo_added: function() {
                this.pkudo.increment("score");
                this.save_pkudo();
                this.local_store.put({url: this.url, kudoed: true});
            }, 

            kudo_removed: function() {
                this.pkudo.increment("score", -1);
                this.save_pkudo();
                this.local_store.put({url: this.url, kudoed: false});
            },

            startup: function() {

                // startup is called after the widget has been initialized, 
                // and has been attached to DOM and is ready for use. 
                // we could have used postCreate too, but that usually where
                // you setup the widget, and in postCreate you can not assume
                // you are already part of DOM tree.

                this.inherited(arguments); // good practice

                new Parse.Query(this.Kudos).equalTo("url", this.url).first({
                    success: lang.hitch(this, function(result) {
                        this.pkudo = result;
                        if (this.pkudo == null) {
                            this.init_pkudo();
                        }
                        this.setCount(this.pkudo.get("score"));
                    }), 
                    error: lang.hitch(this, this.init_pkudo)
                });

                var ls = this.local_store.get(this.url);

                if (ls && ls.kudoed) {
                    this.markDone();
                }

                this.on("kudo_added", lang.hitch(this, this.kudo_added));
                this.on("kudo_removed", lang.hitch(this, this.kudo_removed));
            }
        });
    }
);
~~~~~~~~~~~~~~~


The good news is that it works as expected. The bad, it is kind of a
mish-mash of data and view. 

## MVC

Lets analyze the problem, there is one "view" in the system, the widget.
And we have two data sources. LocalStore and Parse. Between the view and
data there is actually very little connection. The two connecting points
are "kudoed", which can be set either by LocalStore, or by the Widget;
and "count", which is only set by Parse data store. The view should
update itself based on just these two "links" and data stores should
update themselve based on the "kudoed" data when view sends it.

If things are well designed, this is how the code should look like. Not
a 70 lines of mish mash above. A pseudo code that would properly reflect
this reality is as follows:

~~~~~~~~~~~~~~~
function controller() {
    var key = get_key();
    var widget = get_widget();
    var local_data = get_local_store_data(key);
    var parse_data = get_parse_data(key);

    // set the initial value of widget

    widget.set("kudoed", local_data.get("kudoed"));

    // the count will come asynchrounously from parse later on

    // since state changes in widget now onwards, the data should reflect this

    sync(widget, "kudoed", local_data, "kudoed", {bindDirection: sync.from});
    sync(widget, "kudoed", parse_data, "kudoed", {bindDirection: sync.from});

    // either initival value of count, when kudoed is changed parse updates the 
    // count, and that should update the widget

    sync(parse_data, "count", widget, "count", {bindDirection: sync.from});

    // in future one can imagine a websocket keeping the value of parse_data.count
    // updated based on changes done by others, and widget would then 
    // update itself as we have already bound data from that direction in 
    // previous line.
}
~~~~~~~~~~~~~~~

If we can achieve code that looks like this we would call it
model-view-controller design.

## Dojo and MVC elements

Dojo has a concept of a object store. All data of an application should
be accessed by creating an appropriate object store. A store contains
object(s). Dojo makes no assumption about your store, but typically
there should be an "id" for each object that is stored in the object
store. This is just a recommendation, you may or may not do it. Then
each store should have some kind of .get() to retrieve object given its
id, this is optional too. A store may provide a .query(), which will
return a list of objects matching some criteria. Again optional. A store
could have .add() and .put() and .remove() methods, to add a new object,
update it and to destroy it respectively. A store may have
.transaction() to start a transaction, .getChildren(object), to return
children if the store is storing a prent child relationships, and
.getMetadata() if store stores some kind of meta data. The objects
themselves that are stored are assumed be simple javascript object with
no assumption on them.

As you can see almost any data model can be thought in terms of store.
This is just a concept, but dojo comes with a few concrete
implementation of stores, for in memory storage, "dojo/store/Memory", a
rest based store, "dojo/store/JsonRest", and a store that is meant only
for caching data, "dojo/store/Cache". In our code above we have used a
store "dojo-local-storage/LocalStorage" for dealing with HTML5
localStorage. 

After object store, dojo has a concept called "dojo/store/Observable".
Any store can be made "observable" by using that class. What is means is
that the collection of objects returned by a .query() can be "watched"
for changes. Lets say if a new object was added that matches the query,
or removed, updated, or even if its position changed. So if a widget is
working on list of todos that are marked not done, and a new todo was
added by some other widget, which is not done, then the widget just
observes the query, and is notified when something changes, and can
update itself. It is quite a powerful concept.

That was for a collection. For individual objects, dojo has a class
"dojo/Stateful". Any subclass of Stateful can be watched for change in
any of its attribute. So we can design a store that returns Stateful
objects instead of regular javascript objects, and widget can call
.watch() on it, and update itself when ever things change in object. An
Stateful object can .watch() itself to store the changes in database
whenever some other part of system updates it. 

Finally dojo has a helper "dojox/mvc/sync" that can watch one
"dojo/Stateful" and reflect its changes in another. This becomes
interesting when we realize "dijit/\_WidgetBase" itself is subclass of
"dojo/Stateful".

With this knowledge the pseudo code we saw above is not very difficult
to implement.

## MVC Based KudoesWidget

Lets first update our widget to not emit events, and two attributes
.kudoed and .count, and update the widget when they change. 

Here is a modified version of our original KudosWidget:

~~~~~~~~~~~~~~~
define(
    [
        "dojo/_base/declare", "dijit/_WidgetBase", "dijit/_TemplatedMixin",
        "dojo/text!dkudos/KudosWidget.html", "dojo/dom-class", 
        "dojo/_base/lang", "dojo/on"
    ], 
    function(
        declare, _WidgetBase, _TemplatedMixin, template, domClass, lang, on
    ){

        return declare([_WidgetBase, _TemplatedMixin], {

            templateString: template,

            // two new member variables

            kudoed: false,
            count: 0,

            // this is special form, it is called when someone calls
            // widget.set("kudoes", value);

            _setKudoedAttr: function(value) {

                // this._set is how to set things without calling events

                this._set("kudoed", value);
                
                if (value)
                    this.markDone();
                else 
                    this.markNotDone();
            },

            // same for count

            _setCountAttr: function(value) {
                this._set("count", value);
                this.setCount(value);
            },

            isKudoable: function() {
                return domClass.contains(this.domNode, "kudoable");
            },

            isKudod: function() {
                return domClass.contains(this.domNode, "complete");   
            },

            complete: function() {
                this.end();
                domClass.add(this.domNode, 'complete');
                this.set("kudoed", true);
            },

            unkudo: function(event) {
                event.preventDefault();
                if (this.isKudod()) {
                    domClass.remove(this.domNode, 'complete');
                    domClass.remove(this.domNode, 'complete');
                    this.set("kudoed", false);
                }
            }, 

            setCount: function(count) {
                return this.counter.innerHTML = count;
            },

            start: function(evt) {
                if (this.isKudoable() && !this.isKudod()) {
                    domClass.add(this.domNode, 'active');
                    this.timer = setTimeout(
                        lang.hitch(this, this.complete), 700
                    );
                }            
            }, 

            end: function() {
                if (this.isKudoable() && !this.isKudod()) {
                    domClass.remove(this.domNode, 'active');
                    if (this.timer != null) {
                        clearTimeout(this.timer);
                    }
                }
            },

            markNotDone: function() {
                domClass.add(this.domNode, "animate");
                domClass.remove(this.domNode, "complete");
            },

            markDone: function() {
                domClass.remove(this.domNode, "animate");
                domClass.add(this.domNode, "complete");
            },

            postCreate: function() {
                on(this.domNode, "click", lang.hitch(this, this.unkudo));
            }
        });
    }
);
~~~~~~~~~~~~~~~

Very little has changed here. We are introduced to two new methods
.setXXXAttr() and .\_set(). First one is called whenever an attribute is
being .set(), and later should be called when we want to set something
but do not want to notify event handlers and such. We also don't need to
increment/decrement counter methods. 

You can play with it by giving the widget an id, locating it using 
require("dijit/registry").byId("the_id") and calling .set() on it in console.

Now lets look at our local storage, it is simpler because we already are
going to subclass "dojo-local-storage/LocalStorage", which is already a
proper dojo storage. Lets create "dkudos/KudosLocalStorage".

Here is how it looks:

~~~~~~~~~~~~~~~
define(
    [
        "dojo/_base/declare", "dojo-local-storage/LocalStorage", "dojo/Stateful"
    ], 
    function(
        declare, LocalStorage, Stateful
    ){
        return declare(LocalStorage, {

            idProperty: "url",

            // luckily for us we have to only update .get()
            get: function() {

                // first we get the actual data from our parent class
                var obj = this.inherited(arguments);

                // then we wrap it and make it Stateful
                var stateful_obj = new Stateful(obj);

                var store = this; // we can also hitch

                // now we want to watch for changes in property "kudoed"
                stateful_obj.watch("kudoed", function(name, oldValue, newValue){
                    store.put(
                        {"url": stateful_obj.get("url"), "kudoed": newValue}
                    );
                });

                // we return stateful object instead of "naked" one
                return stateful_obj;
            }

        });
    }
);
~~~~~~~~~~~~~~~

Lets look at our HTML to see how we are using them together:

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello MVC!</title>
        <link href='/dkudos/KudosWidget.css' rel="stylesheet" />    
    </head>
    <body>
        <div data-dojo-type="dkudos/KudosWidget" id="kudos"></div>
        <script>
            var dojoConfig = {
                async: true,
                paths: {
                    "dkudos": "/dkudos",
                    "dojo-local-storage": "https://raw.github.com/sirprize/dojo-local-storage/master"
                }
            }
        </script>
        <script src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
        <script>
            require(
                [
                    "dojo/parser", "dkudos/KudosLocalStorage", "dijit/registry",
                    "dojox/mvc/sync", "dojo/domReady!", "dkudos/KudosWidget"
                ], function (parser, KudosLocalStorage, registry, sync) {

                    parser.parse();

                    var key = document.location.pathname;
                    var widget = registry.byId("kudos");                    
                    var ls_data = new KudosLocalStorage();.get(key);

                    widget.set("kudoed", ls_data.get("kudoed"));

                    sync(
                        widget, "kudoed", ls_data, "kudoed", 
                        {bindDirection: sync.from}
                    );

                }
            );
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

As promised, this looks much neater, and is indeed much better, more
maintainable code.

We may not like having that much javascript in each HTML and would want
to keep it separate in a js file. Now if we simple put it in a .js and
include the javascript in the page it would work, but we will have an
extra HTTP request.

What we want to do is keep it in a separate file and inform dojo about
it in a fashion that it will be run after dojo is loaded and also
eventually make it part of dojo.js once we have built a baked version of
dojo.js including all modules.

We can do that by using the dojo configuration "deps", here is how our
new HTML looks like:

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello AMD!</title>
        <link href='/dkudos/KudosWidget.css' rel="stylesheet" />    
    </head>
    <body>
        <div data-dojo-type="dkudos/KudosWidget" id="kudos"></div>
        <script>
            var dojoConfig = {
                async: true,
                deps: ["dkudos/main"],
                paths: {
                    "dkudos": "/dkudos",
                    "dojo-local-storage": "https://raw.github.com/sirprize/dojo-local-storage/master"
                }
            }
        </script>
        <script src="//ajax.googleapis.com/ajax/libs/dojo/1.9.0/dojo/dojo.js">
        </script>
    </body>
</html>
~~~~~~~~~~~~~~~

We have added deps: \["dkudos/main"\] to dojoConfig. And yes, this
configuration itself can be made part of fully baked dojo.js once proper
build has been made, so you would not even see those lines in HTML. 

We create a new /dkudos/main.js file containing:

~~~~~~~~~~~~~~~
define(
    [
        "dojo/parser", "dkudos/KudosLocalStorage", "dijit/registry",
        "dojox/mvc/sync", "dojo/domReady!", "dkudos/KudosWidget"
    ], function (parser, KudosLocalStorage, registry, sync) {

        parser.parse();

        var key = document.location.pathname;
        var widget = registry.byId("kudos");
        var ls_data = new KudosLocalStorage().get(key);

        widget.set("kudoed", ls_data.get("kudoed"));

        sync(widget, "kudoed", ls_data, "kudoed", {bindDirection: sync.from});
    }
);
~~~~~~~~~~~~~~~

We have replaced the call from **require** to **define**. Dojo expects
each proper AMD to call **define** once. 

Ok, now lets look at how we would handle data stored in Parse. Parse is
a available to us as HTTP api, that means when the page is loaded the
data would not be immediately available. 

Object stores in dojo can be either synchronous or asynchronous, meaning
.get()/.query() etc may return right away, synchronous, as was the case
with local storage, or they may return a "promise", that will be
resolved at some later point of time. If you have a promise, you can
call .then(callback) on it and the passed callback will be called in
future when the data is ready. 

The object that is returned by .query() on stores is also supposed to
have .forEach(), .map(), .filter() methods, if we use these methods it
does not matter if the data returned by .query() is synchronous or
asynchrounous, the passed callbacks will be called when the data is
ready, right away or sometime in future.

How do you know if a store is sync or async? Ideally this information
should be available with the documentation for the store you are using.
If we are working with stores that are a mix of synchrounous and
asynchrounous, or if the store can be changed based on configuration
parameters, we can use "dojo/when" as a wrapper for data returned by
dojo object stores. 

Lets look at our "dkudos/ParseObjectStore" now to get a feel of how
asynchronous store implementation may look like.

~~~~~~~~~~~~~~~
Parse.initialize(
    "R4itZTuvbWZ1PHI5RCu2LKkqMaMl5QFVZLzPyeYM", 
    "52jAgPNq3iHJitnsqkfB8atmhXOZ5aqFTQPZKF39"
);

define(
    ["dojo/_base/declare", "dojo/Stateful", "dojo/Deferred"], 
    function(declare, Stateful, Deferred){

        return declare(null, {

            get: function(url, params) {

                console.log("ParseObjectStore.get()", url);

                // we are going to return a promise, lets create one
                // whenever we have data we can .resolve() it.

                var promise = new Deferred();

                // if you do not know how parse works, ignore this line.

                var Kudo = Parse.Object.extend("Kudos");


                var stateful_obj = new Stateful();

                stateful_obj.set("url", url);
                stateful_obj.set("kudoed", params.initialKudo);

                // this is when we make a get requst to fetch data from parse
                // HTTP(s) server. the request .first() is asynchrounous, and
                // our success callback is called when the result is ready.
                // ideally we should pass an error callback too and call 
                // promise.reject() to be good citizens of promise world.

                new Parse.Query(Kudo).equalTo("url", url).first({

                    success: function(result) {

                        console.log("ParseObjectStore.success", result);

                        // we are here when the we have received the response
                        // from parse. 

                        // apologies for score vs count confusion
                        // in parse i have number of kudos scored as score
                        // where as in our widget we are using the term count

                        if (result) 
                        {

                            // at this point we set the "count" on the stateful
                            // object so anybody how is bound to count can 
                            // update themselves. 

                            stateful_obj.set("count", result.get("score"));
                            stateful_obj.set("kudo", result);

                        }
                        else 
                        {

                            // this means there is no record for this "url" on 
                            // parse. no problem, we will care it. it should 
                            // only happen the first time a page is ever loaded

                            stateful_obj.set("count", 0);

                            var kudo = new Kudo();
                            kudo.set("url", url);
                            kudo.set("score", 0);
                            kudo.save(); // happens in background
                            stateful_obj.set("kudo", kudo);

                        }

                        stateful_obj.watch(
                            "kudoed", function(name, oldValue, newValue){

                                // this is a bit of ideosyncracy, even tho the
                                // value has not changed, its been updated. we
                                // ignore it. 

                                if (oldValue == newValue) return;

                                // kudo is our object returned by Parse

                                var kudo = stateful_obj.get("kudo");

                                // somebody changed the value of "kudoed", if
                                // the new value is true, we must increment the 
                                // score, else decrement it. we are going to use 
                                // parse specific api for that.

                                if (newValue) 
                                {
                                    kudo.increment("score");
                                } 
                                else
                                {
                                    kudo.increment("score", -1);  
                                }

                                kudo.save({
                                    success: function(savedKudo) {

                                        // we could have just done a count += 1
                                        // but this is more accurate as its 
                                        // getting latest count from server

                                        stateful_obj.set(
                                            "count", savedKudo.get("score")
                                        );
                                    }
                                });
                            }
                        );

                        // this is how we tell anyone who is waiting for data
                        // from promise that data is available, stateful_obj
                        // is the data they will recieve for the original 
                        // .get() call.

                        promise.resolve(stateful_obj);

                    }
                });

                return promise;
            }

        });
    }
);
~~~~~~~~~~~~~~~

We have to also update our "dkudos/main" to use this new store too:

~~~~~~~~~~~~~~~
define(
    [
        "dojo/parser", "dkudos/KudosLocalStorage", "dijit/registry",
        "dojox/mvc/sync", "dkudos/ParseObjectStore", "dojo/domReady!", 
        "dkudos/KudosWidget"
    ], function (parser, KudosLocalStorage, registry, sync, ParseObjectStore) {

        parser.parse();

        var key = document.location.pathname;
        var widget = registry.byId("kudos");
        var ls_data = new KudosLocalStorage().get(key);

        widget.set("kudoed", ls_data.get("kudoed"));

        sync(widget, "kudoed", ls_data, "kudoed", {bindDirection: sync.from});

        new ParseObjectStore().get(
            key, {initialKudo: ls_data.get("kudoed")}
        ).then(function(data){

            widget.set("count", data.get("count"));

            sync(data, "count", widget, "count", {bindDirection: sync.from});
            sync(widget, "kudoed", data, "kudoed", {bindDirection: sync.from});
            
        });
    }
);
~~~~~~~~~~~~~~~

As you can see, .get() can also take an "options" argument, which could
be anything, and we have used to pass the initial value of kudoed from
local store to parse score. 

We have used the .then() because we know our ParseObjectStore is an
asynchrounous store. 

## Finally lets worry about build

The feature works, and we may be satisfied with the code quality, but it
makes 28 http requests. One of them is for the actual page, another for
CSS, yet another for parse.min.js, and one for dojo.js. Then there is
bunch of requests for individual dojo AMD modules, and one request for
our template. And finally there is one request to Parse we make to get
current count. 

Now we should ideally keep parse.min.js separate from our app, or any
other external javascript, like google analytics, ads javascript api and
so on. If not for anything, just to allow the respective owners to
release updated versions of those libraries in case a serious security
issue is found in their implementation. Some libraries also make an
assumption about the URL of script from which they were loaded,
including dojo.js, so it may not be a good idea to change their final
URL.

Also we can not do anything about the HTTP request that Parse makes to
get the actual count. If we had our own backend, and the page was
dynamically generated we may have stored the initial value of count for
the page in the page itself, but in general it may not be a great
design, as the rest of the blog may be updated more than once a day, the
count may keep changing throught the day.  Strictly speaking, we may be
able to bring the actual number of requests down to 2, HTML + dojo.js
(by removing parse from the equation), but we are going to let it be 4,
(+ parse.min.js + ajax for count) for the purpose of this discussion.

## Playing with dojo's build tool

First of all we will have to get a local copy of dojo, dijit etc. Dojo
splits its code base into different projects, "dojo", "dijit", "dojox"
and "util". Each of them are developed independently in different svn
repositories, with read only git mirrors: https://github.com/dojo/dojo,
https://github.com/dojo/dijit and so on.

Now there are many many ways to organize your project, and we are going
to discuss one organization that is preferred by dojo developers.

Lets say we are working on a folder named dkudos-application, and it
already contains our dkudos related files. We now clone the four dojo
related project in this folder, and since we are depending on
https://github.com/sirprize/dojo-local-storage we should clone this too.

~~~~~~~~~~~~~~~
$ mkdir dkudos-application
$ cd dkudos-application
$ mkdir dkudos
$ vim dkudos/KudosWidget.html
$ vim dkudos/KudosWidget.js
$ # ... and our other dkudos related files we discussed above ...
$ git clone git://github.com/sirprize/dojo-local-storage.git
$ git clone git://github.com/dojo/dojo.git
$ git clone git://github.com/dojo/dijit.git
$ git clone git://github.com/dojo/dojox.git
$ git clone git://github.com/dojo/util.git
~~~~~~~~~~~~~~~

Here is how our folder dkudos-application looks like: 

![dkudos application](/images/kudos-application.png)

We are now ready for our building. As a dependency, dojo requires java
or node to build, we are going to use node, and also it is a good idea
to use clojure compiler to minimize javascript files, so we should have
java too. 

dojo requires a "build profile" in order to decide what to do, lets see
how the profile looks like for our application.

~~~~~~~~~~~~~~~
// prof.js content, this is stored directly inside dkudos-application folder 

// we have a variable "profile" that contains build related settings

var profile = {

    // basePath tell dojo build system where is everything, in our case
    // we are going to store prof.js as well as everything else in our 
    // dkudos-application foder, so it points to ".". 

    // note that this path is relative to prof.js file.

    basePath: '.',

    // for us, action is going to always be release

    action: 'release',

    // dojo by default tries to optimize the generated javascript files using
    // clojure compiler or (older) shrinksafe library that comes with dojo. 
    // we are going to switch the optimization off to iterate quickly, as 
    // those optimizations take a lot of time. i tend to prefer to call clojure
    // after dojo build is done. 

    optimize: '',

    // same as optimize, we will discuss layers later on

    layerOptimize: '',

    // this is a working directory in which dojo build system will create files
    // note that this path is relative to basePath.

    releaseDir: 'dist',

    // dojo build system understand .js as well as .css files, in case of css
    // it can optimize a css by removing comments, inlining any css file that
    // was @import-ed into a given css file. setting it to "comments" does both
    // if we do not want that we can set it to ""

    // in our case we are not inlining anything, but if we have more than one
    // widgets, the best practice is to have a separate css file for each widget
    // and one css for for the entire application that simply @import-s each of
    // them, and use cssOptimize below to optimize it into one css file.

    cssOptimize: "comments",

    // dojo has more than one selector engine. "lite" and "acme", "acme" is the
    // acme supports a lot of different queries, while lite is a much simple, 
    // and much smaller selector engine. lite only supports css2 queries, and is
    // 2kb or so, where as acme supports supports full css3, but is 12kb or so

    selectorEngine: 'lite',

    // now we see a new concept of "packages". a package is just a folder 
    // containing js, css, html README etc files. in order to be a proper 
    // "package", we should also add a "package.json" file in the package folder, 
    // containing name, description etc for the package. as you see it is 
    // optional, we have not created one for ourselves. a package may also want 
    // to have "package.js" file, this package.js file is linked from 
    // package.json by putting {"dojoBuild": "package.js"}. so what is 
    // in package.js file? a package may have some files that need not be built 
    // into baked dojo, eg it may have test files, or it may have demo files 
    // etc. package.js can be used to properly "tag" the files so build system 
    // handles the content of the file intelligently. anyways, we are not going 
    // to use that either as we only have .js files which happens to be proper 
    // dojo AMD modules, and css and html files. when we add tests we might
    // want to create our own package.json and package.js files. 

    // the purpose of "packages" setting is two fold, one is to give a list of 
    // packages to dojo, and also to specify the path on disc where to find them
    // in our case we have simply cloned dojo etc in the current directory, but if
    // we had many projects we may not want to do that and keep dojo in a common
    // location. since in our case package name is same as the name of the
    // folder in which we can find them we can just pass a string containing
    // the name, if name and package were not same we would have passed 
    // {name: "dojo", location: "path of dojo relative to basePath"}. we can 
    // mix string and object in case some package are in current folder and some
    // others are in a separate location.

    packages: ["dojo", "dijit", "dojox", "dkudos", "dojo-local-storage"],

    // now comes layers. the motivation behind layers is this: in some projects
    // the code could be quite complex, and different part of code may be needed
    // for different portions of the site. eg gmail. gmail has a bunch of code
    // that is used when page is loaded, but then there is also "contacts" 
    // feature in gmail, which need not be loaded when main gmail application 
    // starts, and should be loaded only when use clicks on the contacts link.
    // similarly for tasks, which is also not always loaded, and only loaded 
    // when user clicks on tasks. if our application was large like this, we 
    // should try to split them in layers. most usual applications will have 
    // just one layer. 

    // in general a layer only needs a name of the layer and the include list.
    // based on modules in include list dojo will create a javascript file that
    // contains all the modules in it. 

    // a layer can also use exclude: ["amd1", "amd2"], in case amd1/amd2 are 
    // required by the AMDs specified in include list, but we want to force them
    // to be not included, this we may do when we want to a build a separate 
    // layer only containing amd1/amd2 for example.

    // finally at least one of the layers must include boot: true. if boot is true
    // then the code for **require** and **define** methods themselves will
    // be included in that layer. 

    layers: {
        'dojo/dojo': {include: ["dkudos/main"], boot: true} 
    }, 

    // finally we can move dojo configuration from HTML to our baked build

    defaultConfig: {
        async: true, 
        deps:['dkudos/main']
    }

};
~~~~~~~~~~~~~~~

Finally lets create our build!

~~~~~~~~~~~~~~~
$ # make sure we are in dkudos-application folder
$ node dojo/dojo.js load=build --profile prof.js
~~~~~~~~~~~~~~~

Running this will create a new folder dist in dkudos-application, and it
will contain dojo/dojo.js, \(among loads of other files\), our only
"layer". It is a convention to name the boot layer "dojo/dojo.js". 

If we open dist/dojo/dojo.js in a text editor, we will that all our
javascript files have been included in it, as well as our HTML template
is there. We will also find dist/dkudos/KudosWidget.css, which is the
optimized version of our original css file. 

Since we disable javascript optimization, we have to manually call
clojure on our dist/dojo/dojo.js:

~~~~~~~~~~~~~~~
$ # make sure we are in dkudos-application folder
$ java -jar util/closureCompiler/compiler.jar \
    --js=dist/dojo/dojo.js \
    --js_output_file=dojo.min.js
~~~~~~~~~~~~~~~

And we have our dojo.min.js, a single file that contains all our modules
and every bit of dojo/dijit/dojox/dojo-local-storage. 

If you gzip dojo.min.js, it should come around 51KB, which is quite
comparable to 32KB of just the jquery library.

And here is how our final HTML looks like, using the newly created
layer:

~~~~~~~~~~~~~~~
<html>
    <head>
        <title>Hello Build!</title>
        <link href='/dkudos/KudosWidget.css' rel="stylesheet" />    
    </head>
    <body>
        <figure data-dojo-type="dkudos/KudosWidget" id="kudos"></figure>
        <script src="http://www.parsecdn.com/js/parse-1.2.7.min.js"></script>
        <script src="/assets/dojo.min.js"></script>
    </body>
</html>
~~~~~~~~~~~~~~~

And the number of network calls is back to just one for all dojo related
javascript. 

![dkudos netowrk](/images/kudos-network.png)

The source code of dkudos-application is available at [github](
https://github.com/amitu/dkudos).

## Conclusion

This ends a whirlwind tour into dojo. I believe dojo is an excellent
javascript library, and hopefully you would have appreciated the breadth
of its coverage, and the ease of its use, once you grasp the basic
concepts in dojo.

Dojo has been around since 2004, and it has commercial support
available. Dojo has pioneered many of the technologies that rest of the
web world has picked up.

Hopefully you will find dojo as fun to use as the author does.

