Elements
========

The API provides various calls for getting information about Elements
within the Scheduler system.

------------
Find by name
------------

If you know the name of an existing element in the system, you can
retrieve more information about it through the API.

The following call through curl:

::

  curl -K curl.opt http://localhost:3000/api/elements?name=<element name>

Note that this will match only elements with exactly that name.

A successful response will have a status code of 200 (OK) and contents
like:

::

  {"status":"OK","elements":[{"id":1023,"name":"Element name","entity_type":"Location","entity_id":"234","valid":"true"}]}

A well formed request which nonetheless matches no elements would return:

::

  {"status":"Not found","elements":[]}


------------------
Fuzzy find by name
------------------

The API can also search for elements which contain a given string within
their names.  This kind of search is not case sensitive.

The following call through curl:

::

  curl -K curl.opt http://localhost:3000/api/elements?namelike=smith

Will return all the elements where the name contains "smith".

A successful response might look like:

::

  {"status":"OK","elements":[{"id":1234,"name":"Able Baker Smith","entity_type":"Pupil","entity_id":4321,"valid":true},...]}

It will contain as many entries in the elements array as there were matching
records in the database.

A well formed request which nonetheless matches no elements would return:

::

  {"status":"Not found","elements":[]}


----------------
Get more details
----------------

Having identified a particular element and obtained its element id, you can
get more details with the following call on the API

::

  curl -K curl.opt http://localhost:3000/api/elements/<element id>

The exact details returned will depend on the entity type.

----------------
Find commitments
----------------

-------------
Find requests
-------------
