Logging in and out
==================

------------
Curl options
------------

This guide gives all its examples using the curl utility.  For ease
of invoking the utility it is handy to save the common options in
a file.

Create a file called "curl.opt" with the following contents:

::

  -H "Accept: application/json"
  -H "Content-Type: application/json"
  -b cookies.sav

These specify that all our data exchanges are to be in JSON, and the
file cookies.sav is to be used to find previously saved cookies.

------
Log in
------

The following commands issued at the command line will effect a login
to the Scheduler API:

::

  rm -f cookies.sav
  curl -K curl.opt -c cookies.sav https://schedulerdemo.xronos.uk/api/login?uuid=f9c4317f-97d8-48ae-abae-dc7b52b63a11

First we delete the cookies file to get rid of any old cookies which
we might have.  Then we invoke curl to connect to the API.

Note the additional "-c cookies.sav" which tells curl to save any received
cookies to the file.  This is needed only for the login command.  After
that the "-b cookies.sav" in the curl.opt file is enough to tell curl to
read the cookies from the file and send them.

.. note::

  The uid used here is hard-coded into the demonstration data loaded
  on the demonstration server.  It will give you access as the user
  Simon Philpotts - the same as if you log in interactively.

For your own development or live system, change the UUID to that
of your chosen user.

A successful response to this login attempt will have an http status
of 200 (OK) and will carry the following JSON data:

::

  {"status":"OK"}

A failed login attempt will have an http status of 401 (unauthorized)
and will carry the following JSON data:

::

  {"status":"Access denied"}


-------
Log out
-------

Interestingly, a log out request will always succeed - regardless of
whether you have previously logged in.  It can thus be used to return
a session to a known state.

The following curl command will effect a logout:

::

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/logout

and the response will be:

::

  {"status":"OK"}

with an http status of 200 (OK).


