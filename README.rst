bdajax
======

AJAX convenience.

bdajax provides JavaScript helper functions and a simple dispatcher system 
driven by XML-attributes. Attributes are defined in its own XML namespace, 
placed in the XHTML markup.

The dispatcher basically provides two behaviors:

- trigger events

- perform actions

Each behavior is bound to a JavaScript event. 

This package bundles required resources for the use of bdajax inside a 
``repoze.bfg/pyramid`` or ``zope`` application. It does NOT include the
required server implementations. bdajax can be used with other Python or 
non-Python server backends too as long as action performing code is implemented
and available through browser URL.

Dispatching
-----------

For those who know KSS (Kinetic Style Sheets) and TAL (Tag Attribute Language):
The syntax reminds of TAL - attributes are interpreted by JS instead of a TAL
interpreter - while the functionality is inspired by KSS.

The main paradigm - at the same time the main difference to KSS - is the use
of an ``event/listener`` model. It enables bdajax to ``listen`` to ``events``
and trigger ``events`` on ajaxified DOM components.

Consider a navtree as example: the idea is to trigger an event when a navigation
item gets clicked. Click semantically indicates a changed context. Then listen 
to this event on all DOM elements to get notified on changing server context.

This makes it possible to have completely decoupled "sub-applications" knowing
nothing but an event contract from each other.

Attributes
----------

Define the following attributes:

ajax:bind="evt1 evt2"
    Indicate bdajax behavior on DOM element and the event(s) triggering
    it/them.

ajax:event="evt1:sel1 evt2:sel2"
    Trigger event(s) on selector. The triggered event gets the target
    as additional parameter on event.ajaxtarget.
  
ajax:action="name1:selector1:mode1 name2:selector2:mode2"
    Perform AJAX action(s) on selector with mode. Selector points to target
    DOM element, mode defines how to modify the DOM tree. Possible
    mode values are ``inner`` and ``replace``.
  
ajax:target="http://fubar.org?param=value"
    AJAX target definition. Consists out of target context URL and a
    query string used for requests on the target context.
    ``ajax:target`` is mandatory when ``ajax:event`` is defined, and
    optional when ``ajax:action`` is defined (depends on if event is triggered
    by bdajax or browser event). See below to get a clue what i am
    talking about.

Provide dependencies on server
------------------------------

This package already includes resource configuration for ``Zope`` and ``bfg``.
This is done due to ZCML. Include the following ZCML include statement to your
ZCML configuration.
::

    <include package="bdajax" />

The expected ``ajaxaction`` view is not provided. Its intended to be provided by 
a custom implementation. See 'Perform actions' below.

Load dependencies in markup
---------------------------

Load dependent JavaScripts and CSS in HTML header 
::

    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.4.2/jquery.min.js"></script> 
    <script src="http://cdn.jquerytools.org/1.2.3/tiny/jquery.tools.min.js"></script>
    <script src="http://fubar.com/++resource++bdajax/bdajax.js"></script>
    <link href="http://fubar.com/++resource++bdajax/bdajax.css"
          rel="stylesheet" type="text/css" media="screen" />

Make sure the contents of ``bdajax.pt`` are rendered.

Define namespace
----------------

In order to keep your XHTML valid when using the XML namespace extension define 
this namespace in the XHTML document
::

    <html xmlns="http://www.w3.org/1999/xhtml"
          xmlns:ajax="http://namesspaces.bluedynamics.eu/ajax">
        ...
    </html>

Event binding
-------------

Indicate bdajax behavior on DOM element.
::

    <a href="http://fubar.com"
       ajax:bind="keydown click">
      fubar
    </a>

Binds this element to events ``keydown`` and ``click``.

Trigger events
--------------

Bind event behavior to DOM element
::

    <a href="http://fubar.com/baz?a=a"
       ajax:bind="click"
       ajax:event="contextchanged:.contextsensitiv"
       ajax:target="http://fubar.com/baz?a=a">
      fubar
    </a>

This causes the ``contextchanged`` event to be triggered on all DOM elements
defining ``contextsensitiv`` css class. The extra attribute ``ajaxtarget`` gets
written to the event before it is triggered, containing definitions from
``ajax:target``.

Perform actions
---------------

An action performs a JSON request to the server and modifies the DOM tree as
defined.

bdajax expects a resource (i.e a zope/bfg view or some script) named  
``ajaxaction`` on server. Resource is called on target url with target query 
parameters. Three additional arguments are passed:

bdajax.action
    the name of the action

bdajax.selector
    the selector must be added to response

bdajax.mode
    the manipulation mode. Either ``inner`` or ``replace``

The resource is responsible to return the requested resource as a JSON
response in the format as follows. 
::

    {
        mode: 'inner',            // the passed mode
        selector: '#someid',      // the passed selector
        payload: '<div>...</div>' // the rendered action
    }

Bind an action which is triggered directly.
::

    <a href="http://fubar.com/baz?a=a"
       ajax:bind="click"
       ajax:action="renderfubar:.#fubar:replace"
       ajax:target="http://fubar.com/baz?a=a">
      fubar
    </a>

On click the DOM element with id ``fubar`` will be replaced by the results of 
action ``renderfubar``. Request context and request params are taken from 
``ajax:target`` definition.

Bind an action acting as event listener. See section 'Trigger events'.
A triggered event indicates change of context on target with params. 
Hereupon perform some action.
::

    <div id="content"
         class="contextsensitiv"
         ajax:bind="contextchanged"
         ajax:action="rendercontent:#content:inner">
      ...
    </div>

Note: If binding actions as event listeners, there's no need to define a target
since it is passed along with the event.

Multiple behaviors
------------------

Bind multiple behaviors to the same DOM element
::

    <a href="http://fubar.com/baz?a=a"
       ajax:bind="click"
       ajax:event="contextchanged:.contextsensitiv"
       ajax:action="rendersomething:.#something:replace"
       ajax:target="http://fubar.com/baz?a=a">
      fubar
    </a>

In this example on click event ``contextchanged`` is triggered and action
``rendersomething`` is performed.

JavaScript helpers
==================

Messages, Infos, Warnings and Errors
------------------------------------

``bdajax`` displays application messages in a jQuery tools overlay. 

``bdajax.message`` displays a plain message. ``bdajax.info`` ``bdajax.warning`` 
and ``bdajax.error`` decorates message with appropriate icon.
::

    bdajax.message('I am an application Message');


Overlay helper
--------------

Load ajax action contents into an overlay.
::

    var overlay_api = bdajax.overlay({
        action: 'actionname',
        target: 'http://foobar.org?param=value'
    });


Modal dialog
------------

Render a modal dialog inside an overlay. The function expects an options object
and a callback function, which gets executed if user confirms dialog. The
callback gets passed the given options object as well. ``message`` is mandatory
in options.
::

    var options = {
        message: 'I am an application Message'
    };
    bdajax.dialog(options, callback);


URL Operations
--------------

Parse hyperlinks for base URL or request parameters.
::

    bdajax.parseurl('http://fubar.org?param=value');

results in
::

    'http://fubar.org'

while
::

    bdajax.parsequery('http://fubar.org?param=value');

results in
::

    { param: 'value' }

Do both at once by calling ``parsetarget``
::

    bdajax.parsetarget('http://fubar.org?param=value');

This result in
::

    {
        url: 'http://fubar.org',
        params: { param: 'value' }
    }


XMLHTTPRequest convenience
--------------------------

``bdajax`` function ``request`` is convenience for XMLHTTPRequests. By default 
it sends requests of type ``html`` and displays a ``bdajax.error`` message if 
request fails.
::

    bdajax.request({
        success: function(data) {
            // do something with data.
        },
        url: 'foo',
        params: {
            a: 'a',
            b: 'b'
        },
        type: 'json',
        error: function() {
            bdajax.error('Request failed');
        }
    });

Given ``url`` might contain a query string. It gets parsed and written to 
request parameters. If same request parameter is defined in URL query AND 
params object, latter one rules.

Options:

``success``
    Callback if request is successful.

``url``
    Request url as string.

``params`` (optional)
    Query parameters for request as Object. 

``type`` (optional)
    ``xml``, ``json``, ``script``, or ``html``.

``error`` (optional)
    Callback if request fails.

Success and error callback functions are wrapped in ``bdajax.request`` to
consider ajax spinner handling automatically.

Perform action manually
-----------------------

Sometimes actions need to be performed inside JavaScript code. 
``bdajax.action`` provides this.
::

    var target = bdajax.parsetarget('http://fubar.org?param=value');
    bdajax.action({
        name: 'content',
        selector: '#content',
        mode: 'inner',
        url: target.url,
        params: target.params
    });

Options:

``name``
    Action name
    
``selector``
    result selector
    
``mode``
    action mode
    
``url``
    target url
    
``params``
    query params


Trigger events manually
-----------------------

Sometimes events need to be triggered manually. Since bdajax expects the
attribute ``ajaxtarget`` on the received event a convenience is provided.
::

    var url = 'http://fubar.org?param=value';
    bdajax.trigger('contextchanged', '.contextsensitiv', url)


3rd Party Javascript
--------------------

When writing applications, one might use its own set of custom JavaScripts
where some actions need to be bound in the markup. Therefore the ``binders`` 
object on ``bdajax`` is intended. Hooking a binding callback to this object 
results in a call every time bdajax hooks some markup.
::

    mybinder = function (context) {
        jQuery('mysel').bind('click', function() { ... });
    }
    bdajax.binders.mybinder = mybinder;


Browsers
========

bdajax is tested with:

    - Firefox 3.5, 3.6
    
    - IE 7, 8
    
    - Chome 7
    
    - Safari 5


Contributors
============

    - Robert Niederreiter <rnix@squarewave.at>


Changes
=======

1.0.1
-----

    - add spinner handling
      [rnix, 2010-12-13]

    - return jquery context by ``jQuery.bdajax``
      [rnix, 2010-12-13]

1.0
---

    - remove call behaviour
      [rnix, 2010-12-04]
    
    - browser testing
      [rnix, 2010-12-04]

1.0b4
-----

    - add ``configure.zcml`` containing all configuration using
      ``zcml:condition``
      [rnix, 2010-11-16]

    - remove overlay data of modal dialog before reloading. otherwise callback
      options are cached
      [rnix, 2010-11-09]
    
    - disable ajax request caching by default in ``bdajax.request``
      [rnix, 2010-11-09]

    - add modal dialog to bdajax
      [rnix, 2010-11-09]
    
    - mark ``ajax:call`` API deprecated. Will be removed for 1.0 final
      [rnix, 2010-11-09]

1.0b3
-----

    - add class ``allowMultiSubmit`` to fit a plone JS contract.
      [rnix, 2010-07-01]

    - fix bug in bdajax.request when finding url including query params.
      [rnix, 2010-07-01]

1.0b2
-----

    - switch to jQuery tools 1.2.3.
      [rnix, 2010-07-01]

    - call binders with correct context
      [rnix, 2010-05-16]

    - add overlay helper function and corresponding styles
      [rnix, 2010-05-16]

1.0b1
-----

    - make it work
      [rnix]