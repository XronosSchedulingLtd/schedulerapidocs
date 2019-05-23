Notes
=====

Any event within Scheduler can have notes attached to it by suitably
privileged users.  Some of these notes are attached automatically
by the system (e.g. notes about pupils projected to be missing from
a lesson) whilst others can be simple notes used to pass information
between staff.

Listing 
-------

To list the notes attached to an event, we send a GET request to
"https://schedulerdemo.xronos.uk/api/events/NN/notes".  For example:

::

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/events/5/notes

which is asking for any notes attached to event number 5 - the
rowing event at Dorney Lake on Saturday.  Before issuing this request,
I have logged in to the demonstration system as SJP and added an
extra note of my own.

The (re-formatted) response to this request is:

::

  {
    "status":"OK",
    "notes":[
      {
        "id":1,
        "contents":"Please could parents not attempt to take rowers away\nbefore the end of the last event.\n\nRefreshments will be provided in the school marquee.",
        "visible_guest":true,
        "visible_staff":true,
        "visible_pupil":false,
        "formatted_contents":"\u003cp\u003ePlease could parents not attempt to take rowers away\u003cbr\u003e\nbefore the end of the last event.\u003c/p\u003e\n\n\u003cp\u003eRefreshments will be provided in the school marquee.\u003c/p\u003e\n",
        "owner":null,
        "valid":true
      },
      {
        "id":9,
        "contents":"Hello API users",
        "visible_guest":false,
        "visible_staff":true,
        "visible_pupil":false,
        "formatted_contents":"\u003cp\u003eHello API users\u003c/p\u003e\n",
        "owner":{
          "id":1,
          "name":"Simon Philpotts",
          "email":"sjrphilpotts@gmail.com",
          "valid":true
        },
        "valid":true
      }
    ]
  }

Note that the first note has no owner because it was added as part
of the seed data, whilst the second note belongs to SJP.

In the formatted_contents field, special characters have been sent
as Unicode, in line with the requirements of JSON.


More detail
-----------

For (slightly) more information about any given note you can issue
a SHOW request for it, specifying its id number.

::

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/notes/9

which for the above note will result in this response:

::

  {
    "status":"OK",
    "note":{
      "id":9,
      "contents":"Hello API users",
      "visible_guest":false,
      "visible_staff":true,
      "visible_pupil":false,
      "formatted_contents":"\u003cp\u003eHello API users\u003c/p\u003e\n",
      "parent":{
        "id":5,
        "body":"Rowing at Eton Dorney",
        "starts_at":"2019-05-25T10:30:00.000+01:00",
        "ends_at":"2019-05-25T15:00:00.000+01:00",
        "all_day":false,
        "valid":true
      },
      "owner":{
        "id":1,
        "name":"Simon Philpotts",
        "email":"sjrphilpotts@gmail.com",
        "valid":true
      },
      "valid":true
    }
  }

The only extra thing being returned here is some basic information
about the note's parent event.  In the previous request we were
asking about the note in the context of the event, so there was
no point in the API passing back information about the event.

Here we've asked about the note in isolation, so it might be useful
to know what event it is attached to.

Adding
------

Provided your API user has permission to add notes, you can add
a note to any event in the system.  Because you are creating a new
record in the database, this calls for a POST request.

::

  curl -K curl.opt \
       --request POST \
       --data '{"note":{"contents":"Note created through the API","visible_pupil":true}}' \
       http://schedulerdemo.xronos.uk/api/events/5/notes

and the response to this request is:

::

  {
    "status":"OK",
    "note":{
      "id":11,
      "contents":"Note created through the API",
      "visible_guest":false,
      "visible_staff":true,
      "visible_pupil":true,
      "formatted_contents":"\u003cp\u003eNote created through the API\u003c/p\u003e\n",
      "owner":{
        "id":1,
        "name":"Simon Philpotts",
        "email":"sjrphilpotts@gmail.com",
        "valid":true
      },
      "valid":true
    }
  }

Note that the visible_staff field has defaulted to true, whilst the
visible_pupil field has been set explicitly in the request.  See below
for more details.

Updating
--------

Provided your user owns an existing note within the system you can also
update it.  Because you're changing an existing record within the
database this is a PUT request.

::

  curl -K curl.opt \
       --request PUT \
       --data '{"note":{"contents":"Amended contents","visible_guest":true}}' \
       http://schedulerdemo.xronos.uk/api/notes/11

Here we are changing the contents of the note, plus making it visible
to guest users.  The response is:

::

  {
    "status":"OK",
    "note":{
      "id":11,
      "contents":"Amended contents",
      "visible_guest":true,
      "visible_staff":true,
      "visible_pupil":true,
      "formatted_contents":"\u003cp\u003eAmended contents\u003c/p\u003e\n",
      "parent":{
        "id":5,
        "body":"Rowing at Eton Dorney",
        "starts_at":"2019-05-25T10:30:00.000+01:00",
        "ends_at":"2019-05-25T15:00:00.000+01:00",
        "all_day":false,
        "valid":true
      },
      "owner":{
        "id":1,
        "name":"Simon Philpotts",
        "email":"sjrphilpotts@gmail.com",
        "valid":true
      },
      "valid":true
    }
  }

The visible_guest flag has changed to true, whilst the other two
are unchanged because no values were given in the update request.


Parameters
----------

The parameters which you can pass in a create or update request are
as follows:

contents
        The body text of the note.  This is interpreted as Markdown,
        a lightweight markup language, meaning you can achieve a
        certain amount of formatting.  There's a good article
        on `Wikipedia
        <https://en.wikipedia.org/wiki/Markdown>`_ about it.
        In particular, you can embed links within your content.

visible_guest
        A flag controlling whether the note will be visible to guest
        users of the Scheduler system.

        Default: false

visible_staff
        A flag controlling whether the note will be visible to
        logged-in staff using the Scheduler system.

        Default: true

visible_pupil
        A flag controlling whether the note will be visible to
        logged-in pupils using the Scheduler system.

        Default: false


Deleting
--------

Finally, you can delete any note belonging to your user.

To delete the note which we've been playing with the request
would be:

::

  curl -K curl.opt \
       --request DELETE \
       https://schedulerdemo.xronos.uk/notes/11

and the response would be simply:

::

  {"status":"OK"}


If you were to attempt to delete it a second time, the response would
be:

::

  {
    "status":"Not found",
    "exception":"ActiveRecord::RecordNotFound",
    "message":"Couldn't find Note with 'id'=11"
  }

because the record no longer exists in the database.

