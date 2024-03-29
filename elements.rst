Elements
========

The API provides various calls for getting information about Elements
within the Scheduler system.

------------
Find by name
------------

If you know the name of an existing element in the system, you can
retrieve more information about it through the API.

Using curl, a suitable invocation would look like this.

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements?name=L101

Here we are searching for an element called "L101".  Note that the call
will match only elements with exactly that name.

A successful response will have a status code of 200 (OK) and contents
like:

.. code-block:: json

  {"status":"OK","elements":[{"id":58,"name":"L101","entity_type":"Location","entity_id":"3","valid":"true"}]}

A well formed request which nonetheless matches no elements would return:

.. code-block:: json

  {"status":"Not found","elements":[]}

.. note::

  If the name which you want to pass has spaces in it then life gets slightly
  trickier.  The spaces need encoding to be passed as part of a URL.

  You can do it yourself like this:

  ::

    curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements?name=SJP%20-%20Simon%20Philpotts

  Each space has been replaced with "%20".

  Or you can get curl to do the work for you like this:

  ::

    curl -K curl.opt -G --data-urlencode "name=SJP - Simon Philpotts" https://schedulerdemo.xronos.uk/api/elements

  Curl will do the necessary encoding of your string, then append it
  to the URL with a "?".  The "-G" is needed to tell curl still to use
  a GET call.

  All this has nothing to do with the workings of the Scheduler API - it's
  just tricky passing around strings with spaces.


------------------
Fuzzy find by name
------------------

The API can also search for elements which contain a given string within
their names.  This kind of search is not case sensitive.

The following call through curl:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements?namelike=mi

Will return all the elements where the name contains "mi".

A successful response might look like (re-formatted):

.. code-block:: json

  {
    "status":"OK",
    "elements":[
      {
        "id":119,
        "name":"Mimi Winters (11/SJP)",
        "entity_type":"Pupil",
        "entity_id":18,
        "valid":true
      },
      {
        "id":127,
        "name":"Emily Simmons (11/SJP)",
        "entity_type":"Pupil",
        "entity_id":26,
        "valid":true
      },
      {
        "id":130,
        "name":"Millie Marple (11/SJP)",
        "entity_type":"Pupil",
        "entity_id":29,
        "valid":true
      }
    ]
  }

The response will contain as many entries in the elements array as there
were matching records in the database.

.. note::

  The names in the demonstration database are re-generated randomly each night.
  What you get back if you run the same query will almost certainly
  be different.  That's why this demonstration uses such a short
  search string - "mi" - to raise the likelihood of getting a hit.

A well formed request which nonetheless matches no elements would return:

.. code-block:: json

  {"status":"Not found","elements":[]}


There is a limit of 100 on the number of elements which this call will
return.  This is to prevent excessively large responses, and to discourage
dumping of the database by doing a query for, for instance, "e".

----------------
Get more details
----------------

Having used the calls above to find out Simon Philpotts's element id,
one can get more detail with:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements/20

This will produce a response like this (re-formatted):

.. code-block:: json

  {
    "status":"OK",
    "element":{
      "id":20,
      "name":"SJP - Simon Philpotts",
      "entity_type":"Staff",
      "entity_id":1,
      "current":true,
      "email":"sjrphilpotts@gmail.com",
      "title":"Mr",
      "initials":"SJP",
      "forename":"Simon",
      "surname":"Philpotts"
    }
  }

giving more details of that particular element.

The exact details returned will depend on the entity type.

----------------
Find commitments
----------------

We can also find out what events Simon Philpotts is involved in using
the following call:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements/20/commitments

Note that by default this will fetch just the commitments for today's
date.  As always, the data will arrive as a single long string, but
here it is re-formatted for ease of reading:

.. code-block:: json

  {
    "status":"OK",
    "commitments":[
      {
        "id":20,
        "status":"uncontrolled",
        "element":{
          "id":32,
          "name":"All staff",
          "entity_type":"Group",
          "entity_id":1,
          "valid":true
        },
        "event":{
          "id":9,
          "body":"Assembly",
          "starts_at":"2019-04-10T09:00:00.000+01:00",
          "ends_at":"2019-04-10T09:20:00.000+01:00",
          "all_day":false,
          "valid":true
        },
        "valid":true
      },
      {
        "id":75,
        "status":"uncontrolled",
        "event":{
          "id":25,
          "body":"10 Mat3",
          "starts_at":"2019-04-10T09:25:00.000+01:00",
          "ends_at":"2019-04-10T10:15:00.000+01:00",
          "all_day":false,
          "valid":true
        },
        "valid":true
      },
      {
        "id":79,
        "status":"uncontrolled",
        "event":{
          "id":26,
          "body":"9 Mat1",
          "starts_at":"2019-04-10T10:20:00.000+01:00",
          "ends_at":"2019-04-10T11:10:00.000+01:00",
          "all_day":false,
          "valid":true
        },
        "valid":true
      },
      {
        "id":83,
        "status":"uncontrolled",
        "event":{
          "id":27,
          "body":"13 Mat1A",
          "starts_at":"2019-04-10T12:25:00.000+01:00",
          "ends_at":"2019-04-10T13:15:00.000+01:00",
          "all_day":false,
          "valid":true
        },
        "valid":true
      },
      {
        "id":87,
        "status":"uncontrolled",
        "event":{
          "id":28,
          "body":"12 Mat3P",
          "starts_at":"2019-04-10T14:50:00.000+01:00",
          "ends_at":"2019-04-10T15:35:00.000+01:00",
          "all_day":false,
          "valid":true
        },
        "valid":true
      }
    ]
  }

On the 10th of April, 2019, Simon has commitments to 5 events.  The API
returns an array of these commitments, with each of them including
details of the corresponding event.

Note that the first one is slightly different from the others.  Here
Simon is not directly committed to the event - instead the commitment
is for "All staff", and Simon is a member of the group "All staff"
and so the commitment shows up in his schedule.

For this one commitment, details of the linked element are also returned
because the element is not Simon's own one.

The other commitments attach Simon directly to the corresponding events.
There is no point in returning details about him in each one, so the
commitment records contain just details of the event.

You can also specify the dates on which to search for commitments,
like this:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements/20/commitments?start_date=2019-04-11\&end_date=2019-04-12

Note the need for a backslash before the ampersand in this command line to
prevent the ampersand being interpreted by the command shell.

If only a start date is specified, then just the commitments on that day
will be returned.  If an end date is specified, it is taken as being
inclusive - commitments up to and including that end date.

If required, you can also ask for details of any locations and/or staff
involved in any of the returned events using a command like this:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements/20/commitments?include=staff,locations

This will cause the return of initials of any staff and the short names of any
locations involved in the events as additional data in the event records.

.. code-block:: json

  {
    "status":"OK",
    "commitments":[
      {
        "id":75,
        "status":"uncontrolled",
        "event":{
          "id":25,
          "body":"10 Mat3",
          "starts_at":"2019-04-10T09:25:00.000+01:00",
          "ends_at":"2019-04-10T10:15:00.000+01:00",
          "all_day":false,
          "staff":"SJP",
          "locations":"L101",
          "valid":true
        },
        "valid":true
      }
    ]
  }

Multiple staff and locations will be returned as a comma-separated list.


-------------
Find requests
-------------

Requests for an element can be found using exactly the same kind
of query.

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements/20/requests

Note that only certain specialized elements within a Scheduler system
can have requests - currently only Resource Groups.

Requests are used when someone needs, for instance, a mini-bus but
doesn't care which one they get.  The end user puts in a Request for
a mini-bus, and then the administrator of mini-bus converts this
into a Commitment for a particular mini-bus.

