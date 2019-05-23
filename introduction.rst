Introduction
============

-------
Purpose
-------

The Xronos Scheduler API exists to allow the development of other
computer programs which interact with Scheduler. Such programs can
query the Scheduler database and create new entries within it,
subject of course to the usual authentication and access permissions.

--------------
Pre-requisites
--------------

In order to be able to use the API, you need an existing running
installation of Xronos Scheduler.  You are strongly advised to
set up some kind of development system against which to do your
software development.  This can easily be on the same machine where
you are doing your development, with a copy of the Scheduler code
running in development mode.

To get the feel of the API, you can also try it out against
the public Scheduler demonstration system at:

https://schedulerdemo.xronos.uk/

All the examples given here use that system and can be run directly
from your computer.

If you were instead using a local copy of Scheduler you would
change the URL prefix to:

http://localhost:3000

and then for your live system to:

https://myscheduler.example.org/

or whatever your installation's FQDN is.

--------
Concepts
--------

The API uses the normal https communication interface, but with
the associated data (both in and out) formatted as JSON.  This means
it isn't accessible from a normal web browser, but it can be driven using
tools like
:ref:`Curl <curl>` described below.

---------------------------
Fundamental data structures
---------------------------

Scheduler is built around the idea of things which can be involved in
an event.  Internally it uses the name **Element** to refer to anything
which meets this requirement.

The following kinds of Elements exist within Scheduler.

- Staff
- Pupils
- Locations
- Subjects
- Services
- Groups
- Properties

Each of these types has its own table within the database, but any time
an item is added to one of those tables a corresponding Element record
is also created and linked to it.

Each Staff record has an id, but that id is unique only within Staff
records.  There may easily be a Pupil record with the same id.

However, the Element records also have ids, and these will be unique across
the whole system.  No two items will have the same Element id.  This makes
Element ids particularly suitable for referencing items through the API.

The Element record holds information about the item which is generic to
all items (e.g. the name) whilst the linked specialist record (referred
to as an Entity) will have fields specific to the type of item.

When you retrieve information about an Element, you will always be told
what kind of Entity the record is linked to.  For example:

.. code-block:: json

  {
    "element":{
      "id":          1234,
      "name":        "Able Baker",
      "entity_type": "Staff",
      "entity_id":   12
    }
  }

This is how you would receive summary information about the Element
with id 1234.  Its name is "Able Baker", it is linked to a Staff
record, and the id of the corresponding Staff record is 12.

Using that Element id (1234) you could use the API to ask for fuller
information about the element.

------------
System setup
------------

In order to make use of the API, your Scheduler installation needs a
small amount of preparatory work.

To create events through the API, your system needs an Eventsource
with a name of "API".  New installations will have this
already, but if you're upgrading an older system you will need to
create it manually through the usual menus.

Without this, attempts to create events will receive an error code of
503 (Service unavailable).

You also need to set up at least one user within the system with permission
to make use of the API.  By default, users cannot issue any API calls.

To enable a user, edit your chosen user record and tick the "Use API"
permission box.  You can also do this by way of a User Profile if you
prefer.

At the same time take a note of the chosen user's UUID, displayed directly
below the permission bits.  This is a unique string for each user, and
will be needed for the API authentication process.

All user's have a UUID, but unless the "Use API" bit is set on a user,
it has no purpose.

.. _curl:

----
Curl
----

Because the API requires all requests to be in JSON format, it is not
generally possible to invoke it from a normal web browser.  For
initial experimental work, a very useful tool is "curl" - a free
utility available for most computing platforms.

This can be used to invoke the API from the command line and will
display the results which it gets back.  All the examples in this
guide will be couched in terms of how to make the relevant request
using curl.

You probably wouldn't want to use curl in your actual implementation,
but it's a handy way of getting started and checking you have the
concepts right.  Your programming language of choice should have suitable
libraries available for making https requests.


----
URLs
----

All functions of the API require you to access a URL, using one of
GET, POST or DELETE.  A typical URL would look like this:

  https://schedulerdemo.xronos.uk/api/elements?namelike=smith

Obviously you need to change the "schedulerdemo.xronos.uk" bit to match
your own host's domain name, but the next bit is always "/api/" and
then a path using the normal Rails conventions.

The above URL is asking for a listing of all elements with a name
which contains "smith".  The search is not case sensitive.

--------------
RESTful routes
--------------

Scheduler is built on top of the `Rails`_ application framework
which makes use of `RESTful routing`_.  All the URLs used by
the API therefore follow this convention.  Section 2.2 of the
second web page lists the kind of paths which might be used.
The Scheduler API makes use of a subset of these.

.. _Rails: https://rubyonrails.org/
.. _RESTful routing: https://guides.rubyonrails.org/routing.html


-------
Cookies
-------

Each response from Scheduler will carry a cookie.  It's important that
your code saves this cookie and sends it again in your next request.
It's a simple session-identification cookie, but without it Scheduler
will have no way of knowing that you have already logged on.

The cookie persists through your session, so you need save it only
at login.  After that you can simply send the same one in every
request until you log out.

------
Errors
------

The API code tries to ensure that it always sends a status code
and error message formatted as JSON.  However, if your request is
so deformed that it never gets to the API component of Scheduler,
you may get an HTML error page instead.

If for instance, you were to change "api" above to "aappii" then
the request would be handled by Scheduler's main error handler
and you would get an HTML response - an error page.

--------
Security
--------

For development work, it is acceptable to use "http://localhost:3000/"
as your basic target, but it is important to make sure that your live
installation uses https instead.

The authentication process involves sending the user's UUID and if
you do that over an http connection then anyone could see it.

All the examples in this guide use https://schedulerdemo.xronos.uk/ as a
base URL, so you can run them yourself immediately against this
demonstration server.

The entire database on this server is reset every night, so it doesn't
matter if you create events on it.
