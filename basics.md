# A Taste Of Dojo

In this chapter will learn how to setup dojo and use it for your
program. We will go through all the stages of dojo projects, giving a
glimpse of what dojo has to offer. Further chapters will expand upon
these concepts and give a more in-depth coverage.

No prior experience is assumed with dojo, but simple familiarity with
javascript is required to follow them. The examples presented can be
tested using just a browser, and examples can be written from scratch in
a plain text editor. No other tools are required.

## Basic Setup

### Getting Dojo

As of writing of this book, the latest version of dojo is 1.7.3. Please
visit dojo's official site \url{www.dojotoolkit.com/download/} and
download the SDK version of dojo. 

For working with book it recommended you create a folder somewhere named
dojo-book. Inside that folder extract the downloaded dojo, so that
dojo-book now contains a single folder dojo-1.7.3. Inside dojo-1.7.3 you
will see folders named 'dojo', 'dijit', 'dojox' and 'util'.

### Webserver

Dojo splits its functionality in a number of fine grained modules and
stores them in separate javascript files. Dojo then loads those modules
when needed using XMLHttpRequest. This means getting started is much
easier if you have access to a local HTTP server, as modern browsers do
not allow XMLHttpRequest for local file access otherwise.

If you have python installed on your machine, running the following from
command line would start a basic web server for you:

\lstinputlisting[label=runserver,caption=Running A Basic Web Server,
language=HTML]{../code/runserver.sh}

### Hello Dojo

Let us start with a hello world dojo project. In the dojo-book folder
create hello-world.html containing the following.

\lstinputlisting[label=samplecode,caption=hello-world.html,
language=HTML]{../code/hello_world.html}

Lets load the hello-world.html in browser using the webserver we just
started, open the URL \url{http://localhost:8000/hello-world.html}. If
all went well you will see the message "Hello World" sliding by when the
page loads.

A lot has happened in the short example so lets look a close look at our
example. There are three interesting elements in the page, two script
nodes in head section and a h1 header in the body.

The first script node includes dojo.js. The script node contains an
attribute 'data-dojo-config'. Dojo is modular, when we load dojo, a very
minimal core of dojo is loaded, stored in dojo.js. This core then
inspects the dojo configuration to decide what to do next. There are
mainly two ways to configure dojo, one of them is using
'data-dojo-config' html attribute (the second way of configuring dojo is
discussed in next chapter).

It used to be a bad practice to invent HTML attributes like this till
sometime back, HTML used to not validate, but with advent of HTML5,
using node attributes as a means to pass data to javascript has become
an approved standard. The attributes are required to begin with 'data-',
and HTML5 specs recommend using further namespace for each project to
avoid conflicts. Dojo uses 'data-dojo-' prefix. The meaning of the
attribute is open for interpretation, and it is considered opaque string
from browser's point of view. 

Dojo minimal core in dojo.js loads up, and reads the configuration in
'data-dojo-config' attribute. In this case we have passed 'async: true'.
What this tells dojo is that we are interested in 'async' mode, in this
mode dojo loads dependencies only when needed, in asynchronous fashion.

There are pros and cons of asynchronous mode. Traditionally dojo came in
synchronous mode, loading most of the common set of module that is part
of dojo base on page load. The advantage of this is developer
convenience, you can assume all modules are available and do not have to
explicitly demand anything. In our examples we use asynchronous mode as
this mode gives as full control over what to load, no module we do not
ask is loaded. This book, and dojo developers recommend asynchronous
mode for future projects, and synchronous mode is only there for
backwards compatibility.

Lets come to second script tag. In this script we have called a global
function 'require'. `require' is the only function provided by dojo in
asynchronous mode. 'require' is how dojo implements modular programming.

'require' takes two arguments, a list of modules to load, and an
optional callback function, which is called when all the modules are
loaded. The modules that we pass to require may have their own
requirements, those modules also call 'require' internally, and our
callback function is called when all the modules and their dependencies
have been loaded.

Dojo uses XMLHttpRequest to load the modules passed to 'require', this
was the reason we have to run our examples under HTTP server instead of
local file system file:///. Dojo has a registry of loaded modules, so if
a module is asked more than once, for example if it is a dependency of
more than one module we required, dojo only loads the module once, and
passes its reference to everyone who requires it.

The modules that we require in this example are "dojo/fx" and
"dojo/domReady!". The name of the module has close relation with the
name of javascript file under which they are defined, so dojo translates
module name to file path that has to be loaded. Dojo configuration can
be used to load modules from non standard locations, as we will see in
next chapter.

Modules defined by dojo are of two kind, regular modules and plugins. In
our case "dojo/fx" is a regular module, where as "dojo/domReady!" is a
plugin. The exclamation mark helps dojo distinguish between the two. A
dojo module provides a set of functions and classes. Where as plugins
instruct dojo for special functionalities.

In this case, the plugin "dojo/domReady!", to understand what it does we
have to understand what is DOM. DOM, Document Object Model, is a cross
browser way to represent and interact with the page as objects. DOM
allows to inspect the HTML,  us to traverse the DOM to find interesting
node, or to change properties of any element using methods on DOM.

Browser tries to construct the DOM parallel with loading the
dependencies of a page, like CSS, javascript and images. Because it
happens in parallel, we do not know in advance when will the DOM be
ready, sometimes it will be ready before javascript is downloaded, and
sometimes it will be available after that. 'dojo/domReady!' plugin
instructs 'require' function to wait till browser has prepared the DOM
for us. 

Lets look at our second script block again:

\lstinputlisting[caption=Second Script block in hello-world.html,
language=HTML]{../code/hello_world2.html}

Recap: we loaded dojo in minimal mode, this gave us 'require' function.
We called 'require' function, asking it to load the module 'dojo/fx' for
us. And by using 'dojo/domReady!' plugin we asked it to wait till DOM is
available in the browser. When 'dojo/fx' and DOM are available, our
callback will be called.

When dojo's 'require' loads a module, it stores the module in a private
registry, so that it does not have to load the same module more than
once. 'require' passes the loaded module to our callback function as an
argument. 

Our callback accepts the loaded module 'dojo/fx' as the first argument,
and in the code we can see that we have named it 'fx'. We do not have to
name it 'fx', we can call it 'dojo\_fx', or we can call it
'my\_favorite\_module', it is up to us, but we recommended to chose
names consistently. It is important to understand this relation,
'dojo/fx' is the name of the module, the module is actually stored in a
javascript file 'dojo/fx.js' and our callback receives the module as an
object, whose name is up to us.

Another point to note is that we required two modules as dependencies,
so 'require' will call our callback with two objects as arguments. In
our callback we have accepted only one argument, 'fx'. We have ignored
the second argument, as we are not interested in it, as the only purpose
of 'dojo/domReady!' plugin is to wait till DOM is ready. Other plugins
may return some object that we are interested in, as we will see in
future chapters. The documentation should be consulted to see is
returned by each module or plugin. Javascript allows one to ignore the
argument at the end of argument list if we want. In case we required
more modules, a typical real world example may have dependency on quite
a few modules, in those cases it is important to accept all modules in
the right order.

"dojo/fx" comes with a lot of utility methods to animate DOM nodes, each
of them are available via 'fx' object. One of those utility method is
fx.slideTo(), it  takes a DOM node from its current position and slides
it smoothly to a new position on the page. We pass the ID of DOM node
that we want slide and the destination position where we want it to go,
and slideTo returns an 'animation' object. We will learn more details
about 'animation' objects in future, for now we are only interested in
the .play() method of animation object, which triggers the sliding to
start immediately.

Lets look at our HTML for a moment:

\lstinputlisting[caption=Body of hello-world.html,
language=HTML]{../code/hello_world3.html}

We passed node: "greeting" to slideTo method, this argument can be
either string, as in our example, in which case it is assumed to be the
ID of DOM node that is to slide. Node parameter can also be an
individual DOM node object, as we will can learn in future chapter. We
instructed fx.slideTo() to take the node with id "greeting" and move it
from current position to a new position, identified with top=100 and
left=100, and it did.

% \note{Some Note} % % What is Document Object Model? What is a node? 

## DOM and Events

Lets take a look at few other DOM related functions in dojo and how
events handlers are managed.

To familiarize ourselves with such functions lets try to build a
collapsible menu. The page will contain UL containing LI, any of the LI
can have nested UL. For example:

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree1.html}

Clicking on the anchor tags will open or collapse the UL tree branch
under it. 

\includegraphics[width=12.5cm]{../images/tree.png}

What we want to find all anchor tags in the page. Here is how you do
that in dojo:

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree3.html}

We include dojo as usual, with async mode on. You can see our friend
'require' asking us whats up? The usual 'dojo/domReady!' plugin is there
as well to defer execution of our callback till browser has prepared the
DOM tree for us. What is new the the module 'dojo/query'.

Dojo comes with a very sophisticated query engine, that can be used to
find DOM elements. This query engine is provided by the module
'dojo/query'. The engine supports CSS3 selectors for selecting DOM
nodes.

The query engine is exposed to as a function, and in this case you can
see that 'require' is loading the query function, and passing it to our
callback function, where we are capturing it as first parameter named
'query'.

In our anonymous callback function we first want to find all anchor tags
as direct descendent of LI tag. CSS3 selector "li \textgreater  a" can
be used for that. 'query' function takes the CSS3 selector, and returns
what is called a NodeList. We store the NodeList that contains our
anchor tags in a local variable named 'anchor' in the listing shown
above.

NodeList can be used as an array. Unlike an Array containing DOM nodes,
NodeList has a lot of convenience method to further act on the selection
of DOM nodes contained in NodeList. This is called chaining by some
developers. And we can see an example of chaining in our example.

We have found all the DOM nodes that represent an anchor tag that is
direct descendent of LI node, now we want to be notified whenever user
clicks on any of these nodes. NodeList contains a method .on() that can
be used for this purpose. .on() method on a NodeList is used to bind
event handlers to all the elements contained in the NodeList.

.on() takes the name of the event, in this case it is "click", and a
callback function, the event handler. .on() attaches the passed callback
function as click handler for all the nodes in the NodeList. When the
user clicks on any of those nodes, our function is called in the context
of the clicked node, what that means is the variable this inside the
callback points to the DOM node that was clicked. We can log it on the
console to make sure everything is working fine so far.

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree4.html}

Lets do something useful here. If we look in the HTML structure, the
subtree that is to be hidden when user clicks on an anchor is
immediately after the anchor tag. 

What we want to do is traverse the DOM and get hold of the next element
after anchor tag. Dojo has extension to NodeList to facilitate DOM
traversal, the extension is provided by a module
"dojo/NodeList-traverse". This modules adds methods like .children(),
.parent(), .next(), and so on. Each of these methods return a new
NodeList instance that contains the desired elements. 

So we are inside our callback method, that has access to the node that
was clicked as `this', and we want to find the DOM node that comes right
after that. We have .next() method on NodeList, but where do we get the
NodeList from? The answer is our query() method, so far we saw how it
takes a string containing CSS3 selector, but it can also take an
individual node, like `this', and wrap it into a NodeList:

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree5.html}

Now that we have access to the desired UL, we want to hide it, or show
it if it already hidden. Dojo has yet another module,
"dojo/NodeList-dom", that adds a method .toggleClass() to NodeList.
.toggleClass() takes a classname as argument, goes through each element
in NodeList and checks if that element already has the class, if it is
there, it removes it, and if not then it adds it. Lets do that:

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree6.html}

As you can see we added style for class hidden, and added the new
dependency in require call. This works, clicking on the anchor tag shows
and hides the sub menu as we planned, go ahead and check if everything
is working fine.

\includegraphics[width=12.5cm]{../images/tree-collapsed.png}

You will spot a problem, once hidden, there is nothing there to say that
a node has hidden sub tree. Lets fix that by changing the color of a
node with hidden subtree to red:

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree7.html}

You can see that we are using what is called chaining a lot in the event
handler. This is because most methods on NodeList are written to allow
chaining, by returning the same NodeList, or a modified NodeList. On the
returned NodeList further methods can be called. 

This makes the code concise, and avoids the need of coming up with
intermediary variable names. There is a drawback to this technique, when
chains become long it becomes difficult to debug when things do not work
as expected. It is hard to put breakpoints, or log the elements in chain
in intermediary stage. So it is recommended that if we are debugging, we
unchain by declaring intermediary variables, and call each method on a
separate line so breakpoints can be set.

Lets add buttons to collapse all and expand all branches of the tree.

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree9.html}

We want to do something when user clicks on the anchor tag with
id="collapse\_all". The CSS selector for that is "a\#collapse\_all". We
use our handy 'query' to find it, and attach a "click" handler on it.
Finding the nodes that are collapsed is easy based on above setup, every
collapsed node has got the class "hidden", we have to remove "hidden"
class from all of them. We also have to remove the class "red".

\lstinputlisting[caption=tree.html, language=HTML]{../code/tree8.html}

It is left as an exercise to reader to implement the collapse all
functionality.

\section{Dojo Widgets}

Now lets take a look at something dojo really shines at, its big class
of widget library.

In this example the page will have a button, and clicking on the button
we will show a dialog that greets us.

\includegraphics[width=10cm]{../images/widget.png}

You have already gotten the drill, we will include dojo.js with
async=true, we will call 'require', with the modules we need, and
'dojo/domReady!' plugin to defer execution till DOM is ready. We will
use 'query' to find the button, and to attach the event handler. 

\lstinputlisting[caption=dialog setup,
language=HTML]{../code/dialog1.html}

Dojo comes with a rich set of prebuilt, theme-able dialogs for immediate
use. We will use a class 'Dialog' defined in 'dijit/Dialog'. As you can
guess we will add 'digit/Dialog' to required module list in 'require'
call, and then we will try to create an instance of Dialog in our click
handler.

\lstinputlisting[caption=dialog setup,
language=HTML]{../code/dialog2.html}

When a dialog is created, it is not visible by default, and we have to
call .show() in it to make it visible.

In this example we have used object oriented programming feature of
dojo. 'dijit/Dialog' module returns us a Class as defined by dojo, and
we are creating an instance of this class in our example. In future we
will subclass this class and define our class to modify it behavior.

If you look sharply you will notice that we are creating the instance of
the class inside our click handler, so every time user clicks on the
button, our click handler is called and it creates a new instance of our
class. This is a bit wasteful as we are recreating the instance again
and again. Re-initializing a form also has a side effect, lets take a
look at that in our next example. 

First we add an input in our form:

\lstinputlisting[caption=dialog setup,
language=HTML]{../code/dialog3.html}

If you play with this example you will notice that: if you enter
something in the dialog, and close it, and click on the button again, it
will be forgotten and you will see the dialog with empty input box. This
is because the input we interacted with is getting destroyed, and a new
dialog with new input is getting constructed. 

Lets remedy this, and create the instance of dialog in the main callback
we passed to 'require' instead of inside the click handler:

\lstinputlisting[caption=dialog setup,
language=HTML]{../code/dialog4.html}

When we click on the cross of the dialog, the dialog is not destroyed,
but is simply hidden. Inside the click hander we no longer create the
instance, but simply take the existing instance and call .show() on it.
In this case you can see that the dialog remembers what was entered in
the input when we see it the next time.

Depending on your application requirement, you will pick what you want
to do. Recreating instance of the dialog is not the only way to access
the input field and reinitialize it. Lets see how we can access the
input field and initialize it programmatically.

Before we do that let us discuss some basics of widgets in dojo. Each
widget that comes with dojo is a subclass of "\_Widget", defined in
"dijit/\_Widget" module. The class \_Widget has an underscore in the
name because of a convention in dojo. Anything that has \_ in the name
is not supposed to be used directly, it is considered "abstract". So
what does it mean to us that all widgets in dojo are subclass of
\_Widget? It means they share come common aspects we can rely on, one
example of that is each widget has an attribute called .domNode.
.domNode contains the html DOM node that contain the entire widget. Dojo
constructs our widget inside this node, and then attaches this node to
the page when we ask it to. 

What does it all have to do with our current problem? We want to
initialize the input field in the widget with some value, for that we
have to access the input field. We may have added an ID to our input
field, but that would be wrong, as according to HTML specs one page IDs
of nodes should be unique, and if we end up creating more than one
instance of our widget, we will have inputs with same ID in both these
widgets. What about class? Then each input in each widget we constructed
will have same class, and we will not know which is which. In both cases
we need some help to uniquely identify our input. Which is where domNode
comes in handy.

We have use 'query' method so far, that finds the nodes based on CSS
selector in the page. This method takes a second optional argument too.
This second argument tells query about where to find nodes, if this is
not provided, query finds the node in the entire page, on the other hand
if second argument is passed and points to a DOM node, query only finds
nodes inside this node.

One missing piece is, if we do find the NodeList that contains our input
using query, how do we change its value. For this purpose dojo has
module 'dojo.NodeList-manipulate', that adds .val() method to NodeList.
.val() method without argument returns the value of the first element in
a NodeList, and if we call .val("some value") with some value, it sets
the value of all elements in the NodeList.

With all the pieces falling in line we can see how to find our input
element, we tell query to find "input", and start with domNode of the
widget instance we constructed, and change the value of input to
whatever we want.

\lstinputlisting[caption=dialog setup,
language=HTML]{../code/dialog5.html}

Now the dialog updates the value of input field every time we show it.
If we want to do it only once after the widget is created, we can get
the initializing out from the click handler and it put after created the
instance of widget.

This is a bit of a contrived example, but lets say we want to create a
custom widget that automatically puts the current time in input fields
when initialized. Lets try to decouple the code responsible for widget
behavior from the code that uses a widget. We will call our widget
DialogWithTime. 

For this we will create a subclass of Dialog, and dojo comes with a
utility function called 'declare' to do this. 'declare' is defined in
module 'dojo/\_base/declare', and is used for creating subclasses. It
can be invoked in a variety of fashions for different use cases. 

What we want to do is subclass Dialog and call it DialogWithTime. In
DialogWithTime we overwrite its Dialog's .show() method to initialize
the input and then call the original .show() method of Dialog. Lets see
how we can do it.

\lstinputlisting[caption=dialog setup,
language=HTML]{../code/dialog6.html}

Lets review the new code. We have added another module in our require
list, 'dojo/\_base/declare', and capturing the module as a parameter to
our require callback, and naming it 'declare'. 

We call declare function, telling it to create a subsclass for us. We
store the subclass in a local variable named DialogWithTime. We ask
declare that our subclass has to come from the class Dialog, this is the
meaning of passing Dialog as first argument to declare in our example.
We then tell declare that in our subsclass we want to have a "show"
method. Second argument to declare is an object that is used to
overwrite our base Dialog class. If there was no show method in Dialog,
declare would have created it, but Dialog already had a show method, so
declare decided to overwrite for us. When overwriting an existing
method, declare lets us access the original version of that method as
this.inherited. Depending from method to method, it may or may not be a
good idea to call the original method, in our case, because the .show()
method from Dialog is actually responsible for making the dialog
visible, and we do not want to duplicate that functionality, we simply
call the Dialog's show method using this.inherited.

In our show method we find the input element, note that we are trying to
access the instance of the widget using `this' keyword.

Subsequently we modified our code to use our DialogWithTime instead of
Dialog. 

## Declarative Dojo

Lets look at a different way to use dojo. Dojo is declarative. Or rather
it can be if you ask it to. What does that mean? Lets look an example of
declarative dojo:

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/decl.html}

Go on type it out exactly and make sure you can see an alert message
when you click on that button.

\subsection{What is it?}

First thing you will notice if you observe keenly is that this time we
have put data-dojo-config="async: true, parseOnLoad: true". First thing
we observe is we can pass more than one dojo configuration option by
separating them using comma.

We already know what async: true means, load only the minimal core that
gives us require function. parseOnLoad tells dojo to "parse" the page
once it is loaded, and process all DOM nodes that have special dojo
related data attributes.

In order to parse we have to load a module, "dojo/parser". We also have
to load modules necessary for all the widgets we have used in our page,
in our case we have the following:

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/decl2.html}

data-dojo-type attribute tells dojo the type of widget we want. We want
"dijit.form.Button", this widget represents a button, so we include it
as a dependency in our require call, along with "dojo/parser":

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/decl3.html}

Dojo sees the DOM element <button>, and sees that we want to convert it
to an instance of "dijit.form.Button" widget. Dojo takes the DOM node
for <button> and removes it from the body, creates an instance of
dijit.form.Button class and places its domNode in place of original
<button>. We could have used <div> or any HTML tag instead of <button>,
we used <button> as it semantically close to what we want.

\includegraphics[width=8cm]{../images/decl.jpg}

\subsection{Event Handler}

Directly inside the <button> that has been declared as
dijit.form.Button, we can include dojo specific <script> nodes. Dojo
defines a script type "dojo/method", that can be used to define event
handlers. For example we are interested in onClick event of
dijit.form.Button widget we constructed:

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/decl4.html}

Inside this special <script> node we write the code that acts as event
handler. This handler has access to the widget as "this".

### Layout

So what is the advantage of using dojo's declarative way for creating
widgets? There are two main benefits of using this, first is that the
structure of page becomes obvious at a glance. In the next example we
will use dojo's layout related widget to further elaborate on this
point.

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/layout1.html}

In this example we are using "dijit.layout.TabContainer", a tab widget,
as you can see in the require statement, and dojo-data-type of a div. We
can see in the screenshot how the widget looks like. We have two divs
with dojo-data-type as "dijit.layout.ContentPane", they are the
individual tabs. The title attribute is used as tab header.

\includegraphics[width=10cm]{../images/layout.jpg}

Lets make our second tab "closable".

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/layout2.html}

\includegraphics[width=10cm]{../images/layout.jpg}

And lets add an event handler that gets called before the tab is closed:

\lstinputlisting[caption=Declarative Dojo,
language=HTML]{../code/layout3.html}

Here we created a dojo/method script with data-dojo-event set to onClose
method of second tab. This script is called with user clicks on close
button, and returns true or false, to indicate if the tab should be
closed on not.

In this chapter we are only skimming through the various aspects of
dojo, we will go in detail on layout related widgets in later chapters.

## Building Custom Dojo

If you have been following along with examples discussed so far and
paying attention to what is downloaded by browser when we test the
examples, you will notice that dojo downloads a lot of files, the last
example we saw had a massive 141 dependencies. This is because dojo is
modular and keeps every widget and class in separate javascript file,
and a lot of widgets has their own separate CSS file.

This is acceptable in development setup, but for production, having lots
of separate, small javascript, css files is a very bad idea. Dojo comes
with tools to help alivate this problem by bundeling all the
javascript/css files you need for your project.


