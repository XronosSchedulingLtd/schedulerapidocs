Event categories
================

In order to create an event, you need to specify an event category
for it to belong to.

The event categories on your system will be dictated by your system
administrator, and the API provides some facilities for you to
find out what is there.

-------
Listing
-------

You can get a listing of available event categories with the following
call on the API:

::

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/eventcategories

and the result which you'll get back on the demonstration system
(re-formatted) is:

::

  {
    "status":"OK",
    "eventcategories":[
      {
        "id":1,
        "name":"Lesson",
        "valid":true
      },
      {
        "id":11,
        "name":"Sports fixture",
        "valid":true
      },
      {
        "id":12,
        "name":"Trip",
        "valid":true
      },
      {
        "id":13,
        "name":"INSET / Training",
        "valid":true
      },
      {
        "id":14,
        "name":"Interview / Audition",
        "valid":true
      },
      {
        "id":15,
        "name":"Practice / Rehearsal",
        "valid":true
      },
      {
        "id":16,
        "name":"Performance",
        "valid":true
      },
      {
        "id":17,
        "name":"Religious service",
        "valid":true
      },
      {
        "id":18,
        "name":"Date - other",
        "valid":true
      },
      {
        "id":19,
        "name":"Event set-up",
        "valid":true
      },
      {
        "id":20,
        "name":"Meeting",
        "valid":true
      },
      {
        "id":21,
        "name":"Hospitality",
        "valid":true
      }
    ]
  }

For each event category, you get back just its id and its name.

.. note::

  Some of the event categories in any Scheduler installation are
  privileged - available only to certain users.  If you do the same
  call as such a privileged user, you will get back a longer list.


-----------
More detail
-----------

If you want more information about a particular event category you
can use a separate call:

::

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/eventcategories/20

This one is asking for fuller information about eventcategory 20 - Meeting.

The formatted response looks like this:

::

  {
    "status":"OK",
    "eventcategory":{
      "id":20,
      "name":"Meeting",
      "pecking_order":20,
      "schoolwide":false,
      "publish":true,
      "visible":true,
      "unimportant":false,
      "can_merge":false,
      "can_borrow":false,
      "compactable":true,
      "privileged":false,
      "clash_check":false,
      "busy":true,
      "timetable":false
    }
  }

For more information about the these categories, see the
`Event Category`_ page in the Scheduler User Guide.

.. _Event Category: https://xronos.uk/eventcategories.html

For details of each flag, see the Event Category listing within
the admin pages of Scheduler.

