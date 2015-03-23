# Working With DOM

Document Object Model is a set of APIs browsers provide to javascript
for working with the content of a page. One of the reasons javascript
libraries like jquery and dojo came into being is because the APIs are
not consistent across browsers. Which means not all APIs are available
in all browsers, and even when available sometimes they tend to not work
the same in each browser at times.

Lets start with an example. Here is page:

<<[hello.html](code/dom/hello.html)

If we open the page in Chrome, this is how it looks like:

![hello.html](images/dom/hello-inspector.png)

## Playing With DOM in Plain Javascript

First of all we need a basic understanding of what is the meaning of DOM
is an API to work with content of page. In order to do so we will use
the Google Chrome's Developer Tools.

If you have not yet installed Google Chrome, please do so. Once you have
Chrome, visit any website, for the purpose of this example, we will go
to out hello.html page. The keyboard shortcut to launch Developer Tools
is Cmd-Option-j on Mac and Ctrl-Alt-j on windows. You can also open it
by following the View -> Developer -> Javascript Console.

In either case the following should pop up. In this "console", you can write
javascript code, and "execute" it in the "context" of the current page.

Lets type 1 + 1 and hit Enter to see what happens:

![Developer Console: 1 + 1](images/dom/one-plus-one.png)

Now for the purpose of this chapter I assume you are suffeciently well
versed with javascript basics. Chapter 12, Javascript Key Concepts, will
cover some basics of javascript, and you may want to review it if you
are new to javascript.

Lets get to DOM now. The javascript we type in the "console" runs in the
"context" of the current page. What is the meaning of running in the
"context" of something? Based on the "context", some global variables
are available to javascript.

If you have header of Node, it is used for writing javascript on server
side. In case of node, the javascript runs in the node "context", which
means javascript in node have access to a different set of global
variables.

So what are those variables? DOM API defines a huge number of global
variables. Actually that is a lie. DOM API specifies one global
variable, "window". "window" variable contains the entire DOM API as
attributes.

### Objects, types and attributes

Javascript is an object oriented programming language. What that means
is everything in javascript is an "object". Objects have "type", and
"attributes". The "type" determines what "attributes" are present on an
object. Javascript is also dynamic in nature, which means "attributes"
that are not decided by "type" can also be present, or "attributes"
required by "type" can be absent on an object.

An attribute "a" of an object "o", can be accessed by "o.a", the dot
notation, or it can be accessed by 'o["a"]', the "hash" notation.

Learning an API in javascript involves learning what all objects are
"exported" by the API, the different "types", and for each "type", the
"attributes" available.

The "types" of object may be integer or string. Or it can be function.
Some object may be "array" or "hash". And finally there can be custom
objects, defined as "classes".

### The DOM API

With that in mind, lets learn DOM API. Dom API at the highest level
exposts an object "window". This object contains almost everything else
that exists in javascript when running in "page" "context".

We can look at the attributes on window object by using Developer
Console, type "window." and this pops up:

!["attributes" of window object](images/dom/window.png)

Go ahead, peak into it, this is all available for you to play with, and
we will look into some of them in this chapter.

One note of caution, the attributes that you see in window are specific
to Google Chrome. This is what I meant by saying the DOM API varies from
browser to browser. Some attributes are expected on every browser, but
some not. Even the attributes that are present may not behave exactly
the same on all brwosers, or even different versions of same browsers.

Dojo and other javascript libraries are there to deal with this mess.

Before we proceed, there is one speciality of window object in browsers.
Every attribute present on window object are also available globally,
and vice versa, that is every global variable automatically becomes
available as a window attribute.

These concpets play crucial role in javascript programming, we will get
to them later on. Lets begin learning window object.

The most important attribute of window is window.document.

![window.document](images/dom/window_document.png)

Instead of accessing "window.document", we can also access it as
"document".

The idea of window is that it captures the current window, where as idea
of window.document is that it represents the currently loaded page.

Lets see what it means by a few examples.

![window and document](images/dom/window-and-document.png)

Now document.all is for humour only, it is a deprecated attribute, that
used to be abused a lot by javascript developers in the old days, and
soon it will not be available anywhere.

As you see in the screenshot, and I highly recommend you type them out
and see it live in browser, and play some more.

So we have discovered window, window.document, window.document.title, and
window.document.all. These are few of the DOM APIs. Lets learn our next
concept.

The page that we see is composed of what is called DOM Nodes. We know a typical
HTML starts with html tag, which contains head and body, and each of them
contain more "elements" like title, h1, p, div etc. All these HTML "elements"
are converted to DOM Nodes by browser, and is available to us throught DOM
APIs.

In the example we saw window.document.all, an array containing all the DOM
Nodes, as one way to access DOM Nodes. If we want to change the text of
paragraph in hello.html, we do something like this:

![document.all](images/dom/document_all.png)

DOM Nodes are themselves javascript objects, and they have attributes. The HTML
of the page is converted to a set of these DOM nodes and they represent all the
informaiton present in the HTML. DOM Nodes can also be modified, attributes
changed, Nodes themselves can be removed from the page, or created dynamically
using javascript. These allow us to control every aspect of the page.

DOM Nodes are internally organized by browser in the form of a "DOM tree".

![DOM Tree](images/dom/dom_tree.png)

window.document is the top level Node, or "Element", which captures html
element in a HTML file. So what is the difference between a DOM Node and DOM
Element? Lets just say the designers of the API have too much time at their
hand, and love creating new stuff. They are almost the same thing, but to
justify their payroll those people have added some subtle difference that will
bite you when you are least suspecting. And this again is the reason the DOM
API should not be directly handled and abstractions like dojo and jquery should
be used instead.

So we have DOM Nodes that represent individual elements in HTML, and there is a
one to one mapping between the content in the browser and the DOM nodes. Also
the DOM Nodes are live, meaning any change to the page reflects in DOM nodes
and any change in DOM node reflects in the page. We can see that modifying the
node.innerText[^innerTextIssue], the attribute that captures the text part of
an element.

[^innerTextIssue]: node.innerText is actually not a standard attribute, and is
    not supported in firefox for example. The "standard" way to access text
    content of node is node.textContent, but .textContent is only supported in 
    IE9 onwards.

And these DOM nodes are organized in a tree internally. So each DOM node can
have some children, and one parent. window.document is the grand parent of the
entire tree.

### What all we can do with DOM?

Before we get into any details, its better to understand a high level overview
of what all we can do with DOM api.

As we have seen we can "traverse" the DOM tree, and find the DOM nodes we are
interested in, using .children and .parentNode attributes of DOM nodes,
starting with window.document, which is grand ancestor of all nodes in the
page.

We also saw how we can get the text of a DOM node by using .innerText
attribute, and how if we assign a new string, the text automatically updates in
the page.

But before we can start playing with DOM nodes, there is one more concept to
understand. We have been so far interacting the the DOM of a page in the
javascript console, the page had been small, and was loaded from local file
system. Due to all these reasons, by the time we get to start looking into DOM
tree, the DOM tree was already ready for us. This is not always the case, for
example, when the page is big, or if its being loaded from a slow network.

Basically what happens is window.document object is constructed almost as soon
as browser window is ready. Once we try to open some page, for example when we
type google.com and hit enter in the browser, browser starts downloading the
page, and instead of waiting for the page to fully download, it starts
"rendering" partial page as soon as any data is available. This browser does so
we can start seeing data as soon as we can, instead of waiting for the whole
page to load.

Now so far we ran our javascript in console, but javascript can be embedded in
the HTML also, and that javascript too has access to the DOM api, and DOM tree.
But since browser does not want to delay things till the entire page is loaded,
the browser starts executing our javascript as soon as it is ready, even before
the page has been fully loaded, and DOM tree fully constructed.

The effect of this is that our javascript embedded in the HTML, can sometime
not see the entire DOM tree and only partial DOM tree, and if we depended on a
DOM node which is not yet available, the javascript can fail. 

For this reason, and for many other good use cases, there is a concept of
"events" in DOM API. DOM API specifies a set of "events", and DOM API has a way
to "bind" those events to our "callback" methods or functions.

Example of "events" are "page load", which is "fired" when the page has been
fully loaded, and a "dom ready" event that is fired when DOM is ready (in case
you are wondering what is the difference, dom tree is constructed first, and
the event is fired, page load is fired when dom tree is constructed, as well as
all images and other media in the page are loaded). Now these "events" are not
always standardized, for example "dom ready" is a particularly tricky event to
get right across all browsers, as different browser implement it differently.
This is why again one has to rely on dojo or jquery like javascript libraries
to make things easy.

Other example of "events" are mouse move or click. These "events" are "fired"
when mouse is moved by user, or when mouse is clicked. Or keyboard events. Or
scroll events, which is fired when user scrolls the page. Then there are node
specific events, mouse moving over or away from a dom node, for example.

Further there are time based events one can set up, which is fired after a
certain time, or at a given interval. This is more of a javascript feature and
not particularly related to DOM.

And finally there are "ajax" events, when we do "ajax request".

We have not yet seen how events are "bound" to "event handlers" or "callback"
methods or functions. We can even programmatically "trigger" these "events",
and there are DOM apis to construct "custom events" and "fire" them from
javascript.

So to recap, we have DOM tree we can "traverse" to find the DOM nodes we are
interested in, and we have "events" that gets fired when we have something
happen in the page, like a user action. Finally DOM API gives us full control
to create DOM nodes, destroy the DOM nodes, or to move DOM node from one place
in the tree to other.

These are the things we do with DOM API. There are more things DOM API can do,
there are cookies, location of the page, window size, HTML5 has added loads of
new capabilities that we have access to through the DOM API. But from this
chapter's point of view, we will focus on these three apsects of DOM api,
traversal, events and modification.

## Dojo And DOM

Lets build a simple application to understand DOM methods. Let us say we want a
page that shows the current time when the page loads.

<<[hello_dojo.html](code/dom/hello_dojo.html)

In this you will see that we have added dojo javascript to our page in the head
section. Dojo provides us one function, require, which can be used to load
other funcationalities as AMD modules. Dojo comes with a set of AMD modules
that we can use, or we can write our own amd modules.

In the HTML you see that there is a paragraph element, p, the second child of
body. In this p, we want to change insert the current time at the end. In
javascript we can do: var p = window.document.body.children[1], to get access
to this paragraph node.

Find out, or getting access to the DOM node, or a set of DOM nodes that we are
insterested in is our first challenge. The first way we can do it, if we know
the "structure" of the page is to start from document.body, and follow the
.children, to get to the node we are interested in.

There is nothing wrong with this, but it is "fragile", as the order of
children, the child index etc can change every time we modify the HTML, so it
is not a good idea to rely on document.children[0].children[2].children[0] like
constructs.

Identifying this problem, people who designed HTML introduced a concept called
"id" of a html element. Lets add "id" to our paragraph:

<<[with "id"](code/dom/hello_id.html)

You can see the "id" html "attribute". We already know of attributes like "src"
on img tags, or "href" on a tags. The attribute "id" can be used on any element
in the page, and it is primarily used to "identify" an attribute. For this
reason "id" attribute should contain unique values.

Now that the paragraph we are interested in has an "id", we have can get
reference to it using var p = document.getElementById("current_time"). You can
easily see this is superior to var p = document.children[1], as it does not
depend on the order, allowing you to modify the HTML structure as much as you
want, and as long as the "id" remains the same, the javascript will still
successfully find the right paragraph node.

.getElementById function on document is "reasonably universal", available in
nearly all browsers, but as is the nature of the DOM APIs, it may not be
available everywhere, and sometimes even when available, it can be
buggy[^getElementByIdBug]. For this reason dojo gives us AMD module "dojo/dom",
which contains a function, .byId(), which is equivalent to .getElementById(),
and .byId() fixes a bug in .getElementById implementation in older versions of
Internet Explorer.

[^getElementByIdBug]: In older versions of Internet Explorer, .getElementById()
    used to search in name attribute as well as id attribute, and since
    .getElementById() return the first element that matches, just using 
    .getElementById() may return a wrong element.

![dom.byId](images/dom/dom_byId.png)

Once we have the dom node, we can update its content by: p.innerHTML =
p.innerHTML + " " + new Date();

Now that we know roughly how we find the right DOM node, and how to modify it
to suit our requirement, lets worry about DOM load issue.

This is our first event, "dom ready". We have actually two options here, "dom
ready" or "page loaded".

<<[with domReady](code/dom/dom_load.html)

As I we discussed in previous chapter, require method can "block" for DOM to
load by using the "AMD plugin", "dojo/domReady!".

### DOM node functions vs NodeList functions

If you have used jquery before, you know that every function in jquery operates
on a set of DOM nodes, instead of individual DOM nodes. This is convenient as
you get to use same set of functions for both operating on individual node as
well as on a collection of them. But there is also a drawback, it is bad from
performance point of view.

While most of the times performance is not an issue, sometimes it does become a
problem, especially for programs of increasing complexity, think GMail/Trello
and so on.

With that in mind, dojo has two sets of functions for working for DOM,
"dojo/dom*", these operate on single DOM node, and "dojo/NodeList-*", these
operated on a "NodeList".

So far we saw .byId() method in "dojo/dom" that returns an individual DOM node
if we know the id. The .byId() returns the actual DOM node, instead of any
wrapper, and since we had the raw DOM Node, any operation on it can be as fast
as possible.

In order to get a list of DOM Node matching a given criteria, for example all
nodes with a given class, or given tag name, or all the children, siblings etc
of a given node etc, dojo comes with an AMD module "dojo/query". "dojo/query"
gives us access to a powerful "selector engine", which gives us full power of
CSS3 selectors to select DOM Nodes.

A> ## Selector Engines
A>
A> Dojo comes with a few different "selector engines", for examle "lite",
A> "css2", "css3" etc. There are also external engines like "sizzle" and
A> "slick". Dojo can be configured to use one of them on the whole project
A> level or on a module to module level.
A>
A> The reason there one may want to chose to use one vs other is feature vs
A> performance. Some selector engines provide access to wider range of
A> selectors, but that slows down things a bit. Ideally you should use the
A> simplest selector that matches the selectors you are using.
A>
A> If you do not care of performance, just forget this discussion for now, 
A> dojo will by default use a sane choice, good for most use cases. Once you
A> start to worry about performance, or size of compressed javascript, then
A> you can revisit the selector engine.
A>
A> Appendix TK mentions details of what query is supported by some of the
A> selectors.

Unlike .byId(), which returns a native DOM object, query() returns a special
dojo wrapper, an instance of "dojo/NodeList" class. "dojo/NodeList" is an array
like object, with .length, .push()/.pop()/.shift()/.unshift() etc methods.
"dojo/NodeList" also has methods operate on each DOM Node.

Since dojo philosophy is about only giving functionalities that you need for
your project, instead of giving you a huge set of methods, dojo has split the
methods available on "dojo/NodeList" in separate NodeList "extension" modules.
So for example we do "dojo/NodeList-dom", which when "required"/loaded, adds
.style(), a method to get or set style to each DOM node in the NodeList,
.addClass(), add a class to each DOM node, .removeClass(), .toggleClass(),
.addContent(), .empty() and so on. Then we have a "dojo/NodeList-data"
extension that gives us .data() to get or set arbitrary data to each node in
NodeList.

Dojo also has a NodeList extension, "dojo/NodeList-traverse", with methods like
.children(), .parents(), .siblings() etc which returns a new NodeList matching
the corresponding Nodes. On these modified NodeLists there is a .end() method
that can be used to go back to original NodeList.

This is to support "chainablity". Eg you want to take all paragraphs, add a
class to the parent node, and add a click handler on the paragraphs, you may do
the following:

~~~~~~~~~~~~~~~~~
require(
    ["dojo/query", "dojo/NodeList-dom", "dojo/NodeList-traverse"],
    function(query) {
        query("p").parents().addClass("parent_class").end().click(function(){
            console.log("paragraph clicked");
        });
    }
)
~~~~~~~~~~~~~~~~~

You will notice that I did not capture the "dojo/NodeList-dom" and
"dojo/NodeList-traverse" module objects in my callback function, which accepted
only one parameter, query, which captures "dojo/query" AMD module. The reason
is because "dojo/NodeList-dom" etc do not return a new module, but instead they
modify the existing module "dojo/NodeList", which is being imported in
background for us as it is a dependency of "dojo/query". If we do not import
them, then the corresponding methods may not be available to us.

Note: Sometimes in large projects, even if the current module has not listed
lets say "dojo/NodeList-dom" as a dependency, it may still be loaded as some
other module may have that dependency, in which case the methods like .style()
would be availble, but it is not a good practice to rely on this behaviour. If
that module changed its dependency and removed "dojo/NodeList-dom", this module
would break. So it is always recommended to "require" all the AMD modules
needed by every module.

### Extending NodeList

NodeList is also meant to be extended by developers who are using dojo to add
custom methods, here is an example of a module that adds .makeRed() method to
NodeList, and this method will change the CSS property "color" of each DOM Node
in the NodeList to "red".

~~~~~~~~~~~
define(
    ["dojo/_base/lang", "dojo/NodeList", "dojo/NodeList-dom"],
    function(lang, NodeList){
        lang.extend(
            NodeList, {
                makeRed: function(){
                    this.style({ color: "red" });
                    return this;
                }
            }
        );
    }
);
~~~~~~~~~~~

Here instead of "subclass"-ing NodeList class, we are "monkey patching" it
(using "dojo/_base/lang".extend(), which takes two objects and adds attributes
of second object into first object). This is because we do not want to create a
new class with all NodeList feature and our makeRed(), we want to add makeRed()
to existing class NodeList. Since "dojo/query" and everything else uses
"dojo/NodeList" class, it is required that we extend that class itself. Usually
this is considered a bad practice, but since it is quite convenient we do it
this way.

In makeRed() we have returned "this" so that our makeRed() method is
"chainable". Since this.style() refers to .style() added to NodeList by
"dojo/NodeList-dom", which goes through each DOM node in the given NodeList, we
did not have to loop ourselves. For other use cases where it is not this easy,
we may have to loop ourselves, eg:

~~~~~~~~~~~
define(
    ["dojo/_base/lang", "dojo/NodeList", "dojo/dom-style"],
    function(lang, NodeList, domStyle){
        lang.extend(
            NodeList, {
                makeRed: function(){
                    this.forEach(function(node){
                        domStyle.set(node, "color", "red");
                    });
                    return this;
                }
            }
        );
    }
);
~~~~~~~~~~~

In this case we used .forEach() method of "dojo/NodeList" class, which can be
used to iterate through all nodes in a NodeList, and use the "raw" method
.set() from "dojo/dom-style" to set color one by one.

## Lets Build A Tree

Now that basics are out of the way, lets apply all this knowledge by building a
"tree". The "tree" would be like you see in Explorer/Finder, a collapsible
tree, containing files and folders. This tree has to be editable, we should be
able to rename any "folder" or "file", we should also be able to add a new
child at any level, or remove an existing one. We should also be able to drag
and drop a branch of the tree from one location to another. You can see the
finished tree on http://amitu.com/leandojo/tree/.

Since we are going to be building a reasonably complex program, it would be a
good idea to "test" it too. So we will do a "test driven development" from
beginning.

A> ## On "project" setup
A>
A> In chapter tk we will look into details about how to setup a project, for
A> personal development I keep dojo related projects, dojo[^dojo],
A> dijit[^dijit], dojox[^dojox] and dojo-util[^util] checked out in a common
A> location on my machine, and "bindfs" each of the four folder into my current
A> quick test folder.
A>
A> Also note that it is highly recommended to do dojo related work by accessing
A> files over http instead of file://, so I usually do a 
A> python -m SimpleHTTPServer to run a http server serving files in current
A> folder.

[^dojo]: https://github.com/dojo/dojo
[^dijit]: https://github.com/dojo/dijit
[^dojox]: https://github.com/dojo/dojox
[^util]: https://github.com/dojo/util

So I start with a folder named "tree" that contains "dojo", "dojox", "dijit"
and "util", and I run "python -m SimpleHTTPServer" to start a webserver from
it.

To run dojo's "test runner", I open my browser the following location:
http://localhost:8000/util/doh/runner.html, which looks like this:

!["doh" runner](images/tree/doh.png)

What you are seeing is the "test runner", and it is trying to run entire dojo
test suite. If you try to run it, you will see that a few tests are failing,
this is expected as some tests rely on the server supporting php, and if you
are using SimpleHTTPServer like me which does not support PHP, they will fail.

So lets begin.

Some thought on what is going to be our end product. Once we are done, will
have an AMD module "leantree/LeanTree", which will contain a class LeanTree,
that will create a tree in the page. This class will optionally accept in
constructor a json "tree", which will be rendered by default. The class will
also accept a parameter "node" in constructor, which when passed will be
assumed to be a DOM node, or if it is string then it must be "id" of a DOM
node. If node is passed LeanTree class will assume its a "ul"/"li" based tree,
and it will add our behaviour to it. If both parameters are present, then the
"json tree" will be "merged" into the passed "node". If "node" is not passed
then "ul" would be constructed and appended into the body.

Before we proceed, this is not "standard pattern" that dojo follows for this
kind of requirement. The proper thing would have been creating a "dijit widget"
and in chapter TK we will see how to convert this a widget.

Lets write our first test. What we want to test foremost is that our class
exists. Lets look at the test first:

<<[our first test](code/tree/test1.js)

We are using "doh/runner" AMD module that comes with dojo-util[^util]. The way
it works is we doh.register() a bunch of "tests" and it runs all of them. The
"tests" could be either a function or an object. If it is a function, then it
simply runs it, and the function can call doh.assertTrue(), .assertEqual() etc
to test different functionalities.

In case the "test" is an object, it can have .setUp(), .tearDown() and
.runTest(). This can be used for tests that require setting something up before
a test is ran etc. We will get to more details of them as we go along.

So how to run the test? Open
http://localhost:8000/util/doh/runner.html?testModule=LeanTreeTest in browser.

![failing test](images/tree/fail_1.png)

You can see that our test has indeed loaded, and failed.

The test you see is actually slightly incorrect. It will fail and pass based
on absence or presence of our class, but not for the right reason. When our 
test fails, it fails because of a javascript exception, which happens because
we are trying to load a module which does not exists. It is not because of
the doh.assertTrue() call inside the callback method.

Since javascript of "event oriented", sometimes some functions are called at a
future time, and not when the corresponding test function is being run. Like in
our example, the callback function we pass to require() method as second
argument runs after the module is loaded, where as the test finishes when the
assertClassExists() exits. We want to allow our callback to run, and inspect
inside the callback method if things go well.  Further, a callback may register
another callback and so forth.

In order to deal with this doh test functions can return a "deferred object", a
"promise", and if doh find a "promise" or a "deferred object", it waits for the
promise to "resolve". Once a promise is resolved, dojo inspects the "value" of
the "promise", and if it is another "promise", for example if a callback lead
to another callback, then dojo will wait for that too.

We will discuss "promises" and "deferreds" in detail in chapter TK.

Lets rewrite our test using deferreds.

<<[proper test](code/tree/test2.js)

In this case we are testing if "dojo/dom" exists or not, and this test passes.
Lets try to get our test to pass for our module.

As of now the tree folder contains the four dojo related folder, and one test
file: LeanTreeTest.js, lets add a new folder there called "leantree" which will
contain a file "LeanTree.js", which will contain our AMD module:

<<[LeanTree Class](code/tree/leantree/LeanTree1.js)

Please refer to Chapter tk for details on basics of creating a class in dojo.

So now we have a class. Lets implement the part where if a "tree" is passed to
constructor, a DOM is created for us.

We will first write the test to make sure we are clear on what we want. Here we
are going to have a test that creates DOM nodes, depends on structure of HTML
etc, so it good idea for us to familiarize ourselves with yet another feature
of DOH test runner.

When runner.html is loaded, all the tests runs as part of current page. Which
means if the test created some DOM nodes, it must cleanly clean up afterwards,
else the test might interfere with other tests. Also sometimes the exiting DOM
structure itself is a problem, and we need complete control over HTML for our
test. For these two reasons DOH supports a third way for writing and running
tests, along with accepting a function and an object as the two methods we
discussed above. The third method is doh.registerUrl(), this method expects a
URL which must contain DOH tests, runner.html will load the URL in an iframe,
giving the URL full control over desired HTML structure, and since contents of
iframes do not interfere with each other, this gives us near complete
isolation.

Lets set this up now. Yes, it might sound like a lot, but once you have learnt
it, it is not a lot, and you are getting some awesome features. One can easily
argue that dojo's DOH is one of the best way to write unittests in javascript.

<<[registerUrl](code/tree/test3.js)

And the corresponding LeanTreeTest1.html:

<<[LeanTreeTest1.html](code/tree/LeanTreeTest1.html)

To recap, we launch http server from a folder containing dojo, dojox, dijit,
util and our html, javascript files. We then launch our runner.html and pass it
test.js as a parameter, test.js calls registerUrl() with LeanTreeTest.html as
parameter. For more tests, if it depends on DOM, it would be good idea to
create more html files, and if it does not contain DOM manipulation etc, put
them in test.js.

In the test we have passed tree=["one", "two"] as constructor parameter. This
should leave to two children with texts "one" and "two". Since no node is
passed, a node has to be constructed, it will have type UL, and has to be
appended in body. Lets test update our test to assert these bits.

<<[LeanTreeTest2.html](code/tree/LeanTreeTest2.html)

We used a set of assertEqual assertions to make sure our document's DOM
structure is as we expect. As of now our LeanTree does not do anything, so the
test fails at document.body.children.length fails. So lets update our class so
these tests pass.

We are going to have to create a UL DOM node. We have to then iterate over the
passed in tree, to create appropriate LI or UL DOM nodes. Once we create the
nodes, we have to "place" them in the right location.

Lets say we want a "tree" represented as following:

{type=javascript}
~~~~~~~~~~~~
{
    "name": "root",
    "children": [
        "foo",
        {
            "name": "bar",
            "children": ["one", "two"],
        }
    ]
}
~~~~~~~~~~~~

To translate into the following DOM structure.

{type=html}
~~~~~~~~~~~~
<div>
    <a href="#">root</a>
    <ul>
        <li>foo</li>
        <li>
            <a href="#">bar</a>
            <ul>
                <li>one</li>
                <li>two</li>
            </ul>
        </li>
    </ul>
</div>
~~~~~~~~~~~~

<<[creating the tree](code/tree/leantree/LeanTree2.js)

We have a recursive method to create the tree. In this example we have used a
new dojo AMD module, "dojo/dom-construct", which gives us .create() and
.place() methods.

As you can see in the following image the test passes and the tree looks roughly
like how we wanted.

![our tree](images/tree/tree.png)

So far so good. What we want next is for the tree to collapse and expand when
we click on a name. Since mouse click is a user action, we have to ways to go
about handing it. One thing we can do is every so often find the current state
of mouse, assuming we have a way to find it, which we do. And if we find the
mouse button depressed, we find out where the mouse pointer is currently, again
assuming we have a way of finding that out, which we do. Once we know the
location of mouse click, we try to find out if it is one of the "folder" nodes,
by detecting the boundaries of that nodes, which again we know how to find
boundary of any DOM node. And once all these conditions are met, we figure out
how to toggle the subtree, in our case the subtree is descendant of a UL
element, and we have a way to hide a DOM node that hides the node and all its
descendants.

This way of doing things is called "polling" technique. This is because we
"poll" repeatedly. The other way of doing things like this is called "interrupt"
driven, or "event" driven technique. The polling technique is inferior to event
driven one as in polling we do more work than needed. How often do we poll? Once
per second? Once every few milliseconds. The more often we poll, the more work
we, and therefore processor has to do. If in the page we are depending on a lot
of such behaviors, and we poll for each of them, and if we have multiple tabs or
browser windows open, you can see how it will soon exhaust all CPU resource, and
our page and users computer would become slow.

Event driven technique on the other hand works in a callback fashion, where we
create an "event handler" and "assign" this event handler to the "event" we are
interested in. When the event happens, our handler will be called exactly once,
so we will do the minimum amount of work that is needed to handle the event. The
limitation of event driven technique is that is it not always possible, only
a finite set of events are available, and if the condition we want does not
have an event associated with, we can not use event technique.

Since events are faster, browsers makers have created a lot of events for us to
use. Common events like "page loaded", "dom ready", mouse and keyboard events,
browser resize and scroll events are already available to us. Though from time
to time you will come across a situation where this technique is not sufficient
and you have to resort to polling. Even in those cases instead of doing a time
based polling, resorting to some other event may be more efficient. Lets say you
want to do something when a particular DOM node becomes visible in the page. Now
we do not have an event for that yet in browsers, so we have to resort to
polling. But if for example we know a DOM node would change visibility only as
an effect of some user activity, we can "poll" for visibility in an event
handler for different user activities that may trigger this. This hybrid
approach would be more efficient than a time based polling.

When we are dealing with events, we also have a concept of "target" of an event.
Target of an event is usually the exact DOM node on which an event was
performed, if applicable. For some events, like browser resize, or page load,
there is no target, and whole document or window can be considered the target,
but for other events like mouse click, the target of the event is the specific
DOM node that was clicked on. Our event handlers then can specify the target
as well as exact event, so we get a very fine grained targeting.

This is very useful but has a problem, what if we were interested in a click
anywhere in the page and if page has 100s of DOM nodes, do we assign our event
handler to all these 100s of DOM nodes? While we can do it, you can see this is
wasteful, and it is. So browsers have this "feature" called "event propagation"
or "event bubbling". An event "bubbles up". What it means is if an event
happened on a target DOM, it will happen again for the target DOMs parents, and
again for its parent and so on till we reach the top of the DOM tree. In most
cases there won't be an event handlers assigned to parent nodes, so this incurs
very little performance penalty, but in some cases it can cause a problem,
either due to performance, or due to program logic, like in our case, if a
child folder is collapsed by a click event, and if we allow the click event to
propagate up, it will fire a click event on each of its parent, and collapse all
of them. This is not what we want.

So browser allows us to "cancel" or stop this bubbling up from our event
handler. Basically our event handler is a javascript function, and when the
event occurs, our handler would be called. And this event handler function will
be called with an "event object" as function argument. The event object has
.target property to find the target of the event. When bubbling up the target
object remains the same. The event object also has some method to stop the
bubbling of the event.

Event bubbling can also be used as a performance optimization, as we can assign
our event handlers to only the top level DOM node instead of all the matching
DOM nodes we are interested in, since in some cases, especially if we have a
lot of DOM nodes, attaching some event handler to each of them could be too
costly, so we instead assign it to only the common parent, and do the targetting
in the common callback handler.

So far so good. The previous paragraphs are more or less correct, but they have
simplified the picture, particularly the part about different browsers dealing
with all these slightly differently. The very process of attaching our event
handler to different events is not same across all browsers, nor the event
object that is passed.

Fear not as Dojo, and jquery etc "normalizes" all these details for us, and give
us a way to handle uniformly across browsers.

So what we have is: there are events of different kinds, each event when fired
can have a target, and we can assign our event handler to event-target
combination, and once our event handler is fired as a response to event, our
event handler has access to event object which contains actual target and few
other event properties like keycode of key for keyboard events, or mouse
position for mouse related events and so on. And events can bubble and can be
cancelled. For sake of completeness some events have a "default" behaviour, eg
key press, which if it happens in a text field has the default behaviour of
inserting the character corresponding to the key in the text field, and this
default behaviour too can be disabled from our event handler by calling some
method on event object. Multiple event handlers can be assigned to same events,
and everything works just that the order in which event handlers are called
should not be relied upon.

Lets proceed to our example. We are interested in the "click" event. In case
we want to support touch screens on mobile we may be interested in "touch" event
but since each touch event also triggers a click event we would not worry about
it for now.

Now we can either listen for click event on each A tag we have in our tree, or
we can listen for a single click event at the root of the tree, tree.node.

While in general there is little difference other very minor performance
difference, there is one case when the difference between the two may become
significant.

So far the "tree" we created had each of the DOM node constructed by our
constructor. And while constructing we can assign the event handler. But in
some cases the tree may be out of our control, what if a branch of tree has
been created by another piece of code, and that code did not connect our event
handler, then that branch of the tree will not collapse and expand while the
rest of the tree will.

Now whether or not this happens will depend on your application, this difference
should be kept in mind. This commonly happens in cases like: for each link in
my page I want some particular behaviour, may be a tool tip showing the
destination URL, and if we setup our event handlers on each anchor tag when
the page loads, but we inject some html in the page using ajax for example, the
behaviour will have to be reapplied on the new anchor tags being added.

Now you know almost everything there is to know about event handling in
browsers. We will do two iterations of our example using both way of event
handling so you can get a taste of it better.

Before we build the feature we should write a test. Testing an event based
code has its own share of challenges. In order to test if the action happens
when user clicks on a link, and since we want an automated test, which does not
require us to click on things, we have to figure out a way to "simulate" or
"fake" events.

Creating a fake event is problematic, as some events browsers allow to be faked
using javascript, but some other events browser does not due to security
reasons. For this reason "doh" from dojo comes with "doh/robot", a java applet
based technology, which since it is based on a trusted java applet, and
"doh/robot" can created special fake events that can be used to test almost
any event.

Lets look at our test for the click behaviour.

<<[our test with "doh/robot"](code/tree/LeanTreeTest4.html)

Here we have used robot.mouseMoveAt() and robot.mouseClick() methods from
"dojo/robot" to move the mouse over our anchor tag node and left clicking on it.
In order to test if it worked we have relied on testing the .style.display
property.





