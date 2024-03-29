Events
======

POST requests
-------------

All the API requests so far have involved retrieving information and
so have used the GET verb - the same as your web browser uses every
time you look at a web page.  This is also the default which curl uses
when you invoke it witout options.

As we're now going to be creating new records in Scheduler we need
a new verb - POST.  This is the standard verb for creating new entries
of some sort.  

Parameters
----------

When creating an event within Scheduler, you need to give some
basic criteria.  Without these, the event creation will fail.

These are:

body
        A title for the event.  This is what will be shown in the
        usual Scheduler event display.  Simple explanatory text.

        Required field

event_category_id
        Every event must have an event category.  This might be
        "Lesson", "Meeting", "Performance", or anything else set
        up by your system administrator.  In this field pass the
        numeric id of your chosen category.  What this is will
        depend on your system, but on the demonstration system
        you can use a value of 20, which is "Meeting".

        Required field

all_day_field
        Pass true for an all day event, and false for a timed
        event.

        Default: false

starts_at_text
        The starting date or date/time of the event in textual
        format.  For an all-day event, pass just a date in ISO
        format, e.g. "2019-04-01".  For a timed event, pass
        a date and time, e.g. "2019-04-01 12:27".

        Required field

ends_at_text
        The ending date or date/time in textual format.  For
        a timed event, this will default to being the same as
        the start time, giving an event of zero duration.  For
        an all day event, it will default to a duration of 1
        day.  When giving an explicit end date for an all day
        event, give an inclusive date.  If your event runs from
        the 1st to the 3rd, pass an end_date_text of the 3rd.

        Optional

organiser_id
        The element id of a member of staff who should be regarded
        as the event's organiser.  (Note that for consistency
        it is the staff member's *Element* id, and not the underlying
        staff entity id.)

        Optional

.. note::

  "But", you say, "I've read your page about how to handle dates
  and I've built my whole application using exclusive end dates as
  advised.  Do I need to convert my end dates back into human-friendly
  form in order to send them to Scheduler?"

  No, happily you don't.  If you don't want Scheduler to do the
  massaging work for you then you can instead pass the following
  three parameters, which will be stored without massaging (although
  they will still be checked for validity).

  all_day
          Pass true for an all day event, and false for a timed
          event.

          Default: false

  starts_at
          The starting date or date/time of the event in textual
          format.  For an all-day event, pass just a date in ISO
          format, e.g. "2019-04-01".  For a timed event, pass
          a date and time, e.g. "2019-04-01 12:27".

          Required field

  ends_at
          The *exclusive* end date/time of the event.

          Required field

  Don't try to mix and match these in the same request - use either
  the original three or these three - not a mixture.

Elements
--------

The only reason for events to exist in Scheduler is to keep track
of the elements involved in them. An event with no elements might
as well not be there.

You can add elements to events at the same time as creating the event,
or afterwards.

Creating an event
-----------------

To create an event, we send a POST request to
"https://schedulerdemo.xronos.uk/api/events", passing the additional
information listed above as JSON data.  The data for a simple event
creation might look like this:

.. code-block:: json

  {
    "event":{
      "body":"A timed event",
      "starts_at_text":"2019-04-01 12:26",
      "ends_at_text":"2019-04-01 14:23",
      "eventcategory_id":"20"
    },
    "elements":["119", "127", "20", "172"]
  }

The above would specify that we want a timed event (all_day_field left
to default to false) with four elements attached to it.

The curl command to issue this request would be:

.. code-block:: bash

  curl -K curl.opt \
       --request POST \
       --data '{"event":{"body":"A timed event", "starts_at_text":"2019-04-01 12:26", "ends_at_text":"2019-04-01 14:23", "eventcategory_id":"20"}, "elements":["119", "127", "20", "172"]}' \
       https://schedulerdemo.xronos.uk/api/events

Note that we pass the JSON data as one large string to curl.  The command
has been split into several lines for legibility, but should be entered
either as a single line, or with backslashes as shown to indicate
line continuation.

The response might look like this (re-formatted for ease of comprehension):

.. code-block:: json

  {
    "status":"Created",
    "event":{
      "id":93,
      "body":"A timed event",
      "starts_at":"2019-04-01T12:26:00.000+01:00",
      "ends_at":"2019-04-01T14:23:00.000+01:00",
      "all_day":false,
      "commitments":[
        {
          "id":344,
          "status":"uncontrolled",
          "element":{
            "id":119,
            "name":"Mimi Winters (11/SJP)",
            "entity_type":"Pupil",
            "entity_id":18,
            "valid":true
          },
          "valid":true
        },
        {
          "id":345,
          "status":"uncontrolled",
          "element":{
            "id":127,
            "name":"Emily Simmons (11/SJP)",
            "entity_type":"Pupil",
            "entity_id":26,
            "valid":true
          },
          "valid":true
        },
        {
          "id":346,
          "status":"uncontrolled",
          "element":{
            "id":20,
            "name":"SJP - Simon Philpotts",
            "entity_type":"Staff",
            "entity_id":1,
            "valid":true
          },
          "valid":true
        },
        {
          "id":351,
          "status":"requested",
          "element":{
            "id":1,
            "name":"Calendar",
            "entity_type":"Property",
            "entity_id":1,
            "valid":true
          },
          "valid":true
        }
      ],
      "requests":[],
      "valid":true
    },
    "failures":[]
  }

Your results will vary because the demonstration data is randomly
regenerated each night, but you should get two pupils plus Simon
Philpotts and the school's public calendar.

You will notice that the word "status" occurs a lot in the response
data.  The first one is telling you the status of the request.  The
status of "Created" means that your event has been created.  Note that
it's possible for the event to be created, but then for some of the
requested additions of elements to fail.  To see whether there have
been any problems you need to look at the "failures" array.  In our
case it's empty, so nothing failed.

Then each individual commitment has a status field.  In general this
can take a number of values, but only two could come back in the
response to this request.  The values are:

- uncontrolled
- confirmed
- requested
- rejected
- noted

Of these, we can only get "uncontrolled" or "requested" at this stage.

The two pupils and the teacher are freely allocatable, so the commitment
just gets created.  The status is "uncontrolled".

The school's public calendar on the other hand requires a degree of control.
You can't have just anyone putting things into it whenever they feel like
it.  Typically a school will have one or two people who examine proposed
entries for the public calendar and decide whether to accept them or not.

The demonstration school's calendar is like this, and so our commitment
record has a current status of "requested", and the calendar administrator
will be notified that we have a pending entry.  From here the status
might change to "confirmed", "rejected" or "noted".  That last one means
that your request has been seen, but more information is needed before
it can be confirmed.

As our entries are all brand new, and there hasn't been any time for
anyone to approve them or otherwise, the only two statuses which we
can get are "uncontrolled" or "requested".

Adding elements
---------------

If you want to add elements to an event which already exists then
the request is very similar.  Instead of providing details for
the event (date/time etc.) you provide the event id of an existing
event.

The request:

.. code-block:: bash

  curl -K curl.opt \
       --request POST \
       --data '{"elements":["21", "22"]}' \
       https://schedulerdemo.xronos.uk/api/events/93/add

would attempt to add elements with ids 21 and 22 to the event created
in the previous section.  Note that the event id is passed as part of the
URL in line with RESTful conventions.

The response received back is identical to that for creating an event,
with the exception that the status will be "OK" rather than "Created".


Failures
--------

There are only two ways in which adding an element to an event can
fail - the element is already there, or the element doesn't exist.

The following creation request

.. code-block:: bash

  curl -K curl.opt \
       --request POST \
       --data '{"event":{"body":"A timed event", "starts_at_text":"2019-04-01 12:26", "ends_at_text":"2019-04-01 14:23", "eventcategory_id":"20"}, "elements":["20", "20", "banana"]}' \
       https://schedulerdemo.xronos.uk/api/events

deliberately attempts to add the same element (20) twice, and then a gash
event id.

The response (formatted) is:

.. code-block:: json

  {
    "status":"Created",
    "event":{
      "id":102,
      "body":"A timed event",
      "starts_at":"2019-04-01T12:26:00.000+01:00",
      "ends_at":"2019-04-01T14:23:00.000+01:00",
      "all_day":false,
      "commitments":[
        {
          "id":353,
          "status":"uncontrolled",
          "element":{
            "id":20,
            "name":"SJP - Simon Philpotts",
            "entity_type":"Staff",
            "entity_id":1,
            "valid":true
          },
          "valid":true
        }
      ],
      "requests":[],
      "valid":true
    },
    "failures":[
      {
        "index":1,
        "element_id":"20",
        "item_type":"Commitment",
        "item":{
          "id":null,
          "status":"uncontrolled",
          "element":{
            "id":20,
            "name":"SJP - Simon Philpotts",
            "entity_type":"Staff",
            "entity_id":1,
            "valid":true
          },
          "valid":false,
          "errors":{
            "element_id":["has already been taken"]
          }
        }
      },
      {
        "index":2,
        "element_id":"banana",
        "item_type":"Hash",
        "item":{
          "status":"Not found"
        }
      }
    ]
  }

Note that the status for the event creation is still "Created" - creating
the event and adding elements are separate steps.

However, the event has only one valid commitment attached to it.  The
first attempt to add Simon Philpotts succeeded.  The second attempt failed,
and the attempt to add an element with the id "banana" failed too.

You can do a simple check on whether you've had any errors when
adding elements by looking at the size of the "failures" array.
If it is 0, then all is well.

If it is non-zero, then it contains one entry per failed addition.

In each entry we have the following:

- "index" tells us the index of the relevant element_id in the
  array originally passed in.
- "element_id" tells us the actual element_id passed in
- "item_type" tells us the type of the following item.  It can
  be "Commitment", "Request" or "Hash".  If it's one of the first
  two it means the server got as far as trying to create one of them
  but it was invalid, whilst a Hash means it didn't get that far.
- "item" is the failed item, with more information on what went wrong.

Requests
--------

As well as Commitments (which may require approval), Scheduler has the
concept of Requests.  The crucial difference is that requests apply to
a group (a Resource Group) of possible resources.  A user requests one
or more items from the group, and then and administrator allocates
particular ones.

Resource Groups are used for things like mini-buses, and mobile
phones.  A user might want two mini-buses, but it doesn't generally
matter which ones they are as long as they work.  Likewise for a
school mobile phone.

There exists within the demonstration system a Resource Group called
"Minibus".  We can find it with the following request.

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements?name=Minibus

which gets the response (formatted):

.. code-block:: json

  {
    "status":"OK",
    "elements":[
      {
        "id":259,
        "name":"Minibus",
        "entity_type":"Group",
        "entity_id":59,
        "valid":true
      }
    ]
  }

and then we can get more detail with:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/elements/259

which results in (again formatted):

.. code-block:: json

  {
    "status":"OK",
    "element":{
      "id":259,
      "name":"Minibus",
      "entity_type":"Group",
      "entity_id":59,
      "current":true,
      "description":"Resource group",
      "members":2
    }
  }

So, it is indeed a Resource group and it has two members - the minibuses
themselves.

If we include this item's element id in a request to add to an event
then the system will treat it slightly differently.  This is because
we don't want *all* the minibuses in the group, just one of them.

We can make a request like:

.. code-block:: bash

  curl -K curl.opt \
       --request POST \
       --data '{"event":{"body":"A timed event", "starts_at_text":"2019-04-01 12:26", "ends_at_text":"2019-04-01 14:23", "eventcategory_id":"20"}, "elements":["20", "259"]}' \
       https://schedulerdemo.xronos.uk/api/events

which creates a new event and requests Simon Philpotts and a minibus.
The response looks like this:

.. code-block:: json

  {
    "status":"Created",
    "event":{
      "id":93,
      "body":"A timed event",
      "starts_at":"2019-04-01T12:26:00.000+01:00",
      "ends_at":"2019-04-01T14:23:00.000+01:00",
      "all_day":false,
      "commitments":[
        {
          "id":344,
          "status":"uncontrolled",
          "element":{
            "id":20,
            "name":"SJP - Simon Philpotts",
            "entity_type":"Staff",
            "entity_id":1,
            "valid":true
          },
          "valid":true
        }
      ],
      "requests":[
        {
          "id":1,
          "quantity":1,
          "num_allocated":0,
          "element":{
            "id":259,
            "name":"Minibus",
            "entity_type":"Group",
            "entity_id":59,
            "valid":true
          },
          "valid":true
        }
      ],
      "valid":true
    },
    "failures":[]
  }

Simon Philpotts has been attached to the event as a commitment, but the
minibus has been treated differently.  A *request* for a minibus has
been attached to the event and the minibus administrator should allocate
one in due course.

If you want more than one minibus, simply put the element id in
the array of things to add more than once.  The server will notice
it's a repeat and increment the "quantity" field in the existing request
rather than creating a new request.

Querying
--------

If you know the ID of an event (e.g. because you've queried an element
of the system and found it is involved with that event) then you can
get full details of the event using a GET call:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/events/3

Provided the event exists you will get back a status of OK, plus all
the event information in exactly the same format as documented above
for event creation, plus additional fields for the event's owner (a User)
if any, and organiser (a Staff Element) if any.


Modifying
---------

Provided you have edit rights you can modify an existing event using
the same basic fields as for event creation above.  (If you want to change
the elements involved in an event, use the adding and removing calls
listed above - this call is just for modifying the basic fields of the
event like its title, organiser or event category.)

An example call to rename an event would be:

.. code-block:: bash

  curl -K curl.opt \
       --request PATCH \
       --data '{"event":{"body":"A renamed event"}}' \
       https://schedulerdemo.xronos.uk/api/events/93

And a similar call to change the organiser of an event would be:

.. code-block:: bash

  curl -K curl.opt \
       --request PATCH \
       --data '{"event":{"organiser_id":"36"}}' \
       https://schedulerdemo.xronos.uk/api/events/93

The parameters which you can supply to this call are the same ones listed
at the top of this page for creating an event.

Deleting
--------

If you know an event's id (and you have suitable permission) then you can
delete it.  Deleting an event will automatically delete any commitments
or requests attached to it.

In general you can delete any event which you have created.  You may be
able to delete other events as well depending on your user settings.

To delete the event created in the section above the call would be:

.. code-block:: bash

  curl -K curl.opt \
       --request DELETE \
       https://schedulerdemo.xronos.uk/api/events/93

and the response would be simply:

.. code-block:: json

  {"status":"OK"}


Similarly, you can delete individual commitment or request records.

.. code-block:: bash

  curl -K curl.opt \
       --request DELETE \
       https://schedulerdemo.xronos.uk/api/commitments/344

Again, the response in the case of success is just:

.. code-block:: json

  {"status":"OK"}

but if you try to delete a non-existent commitment (e.g. because you
already deleted the event and the commitment went with it) then
you'll get a response like:

.. code-block:: json

  {
    "status":"Bad request",
    "exception":"ActiveRecord::RecordNotFound",
    "message":"Couldn't find Commitment with 'id'=344"
  }


