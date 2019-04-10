Events
======

-------------
POST requests
-------------

All the API requests so far have involved retrieving information and
so have used the GET verb - the same as your web browser uses every
time you look at a web page.  This is also the default which curl uses
when you invoke it witout options.

As we're now going to be creating new records in Scheduler we need
a new verb - POST.  This is the standard verb for creating new entries
of some sort.  

----------
Parameters
----------

When creating an event within Scheduler, you need to give some
basic criteria.  Without these, the event creation will fail.

These are:

.. list-table:: Event parameters
   :widths: 40 200 40
   :header-rows: 1

   * - Field name
     - Meaning
     - Comment
   * - body
     - A title for the event.  This is what will be shown in the
       usual Scheduler event display.  Simple explanatory text.
     - Required field
   * - event_category_id
     - Every event must have an event category.  This might be
       "Lesson", "Meeting", "Performance", or anything else set
       up by your system administrator.  In this field pass the
       numeric id of your chosen category.  What this is will
       depend on your system, but on the demonstration system
       you can use a value of 20, which is "Meeting".
     - Required field
   * - all_day_field
     - Pass true for an all day event, and false for a timed
       event.
     - Default: false
   * - starts_at_text
     - The starting date or date/time of the event in textual
       format.  For an all-day event, pass just a date in ISO
       format, e.g. "2019-04-01".  For a timed event, pass
       a date and time, e.g. "2019-04-01 12:27".
     - Required field
   * - ends_at_text
     - The ending date or date/time in textual format.  For
       a timed event, this will default to being the same as
       the start time, giving an event of zero duration.  For
       an all day event, it will default to a duration of 1
       day.  When giving an explicit end date for an all day
       event, give an inclusive date.  If your event runs from
       the 1st to the 3rd, pass an end_date_text of the 3rd.
     - Optional


------------
Add elements
------------

The only reason for events to exist in Scheduler is to keep track
of the elements involved in them. An event with no elements might
as well not be there.

You can add elements to events at the same time as creating the event,
or afterwards.

-----------------
Creating an event
-----------------

To create an event, we send a POST request to
"https://schedulerdemo.xronos.uk/api/events", passing the additional
information listed above as JSON data.  The data for a simple event
creation might look like this:

::

  {
    "event":{
      "body":"A timed event",
      "starts_at_text":"2019-04-01 12:26",
      "ends_at_text":"2019-04-01 14:23",
      "eventcategory_id":"20"
    },
    elements:["119", "127", "20", "172"]
  }

The above would specify that we want a timed event (all_day_field left
to default to false) with four elements attached to it.

The curl command to issue this request would be:

::

  curl -K curl.opt --request POST --data '{"event":{"body":"A timed event", "starts_at_text":"2019-04-01 12:26", "ends_at_text":"2019-04-01 14:23", "eventcategory_id":"20"}, "elements":["119", "127", "20", "172"]}' https://schedulerdemo.xronos.uk/api/events

Note that we pass the JSON data as one large string to curl.

The response might look like this (re-formatted for ease of comprehension):

::

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

--------
Failures
--------

There are only two ways in which adding an element to an event can
fail - the element is already there, or the element doesn't exist.

The following creation request

::

  curl -K curl.opt --request POST --data '{"event":{"body":"A timed event", "starts_at_text":"2019-04-01 12:26", "ends_at_text":"2019-04-01 14:23", "eventcategory_id":"20"}, "elements":["20", "20", "banana"]}' https://schedulerdemo.xronos.uk/api/events

deliberately attempts to add the same element (20) twice, and then a gash
event id.

The response (formatted) is:

::

  {
    "status":"Created",
    "event":{
      "id":95,
      "body":"A timed event",
      "starts_at":"2019-04-01T12:26:00.000+01:00",
      "ends_at":"2019-04-01T14:23:00.000+01:00",
      "all_day":false,
      "commitments":[
        {
          "id":352,
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
      },
      {
        "status":"Not found",
        "element_id":"banana"
      }
    ]
  }

Note that the status for the event creation is still "Created" - creating
the event and adding elements are separate steps.

However, the event has only one valid commitment attached to it.  The
first attempt to add Simon Philpotts succeeded.  The second attempt
failed, and so the corresponding (attempt at a) commitment record is
in the "failures" array.  It's "valid" field contains false, and the
"errors" field explains why.

Finally the gash element id has resulted in a second entry in the
"failures" array, explaining that there is no element with that id.

--------
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


