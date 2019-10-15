Logging in and out
==================

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

Log in
------

The following commands issued at the command line will effect a login
to the Scheduler API:

.. code-block:: bash

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

.. code-block:: json

  {"status":"OK"}

A failed login attempt will have an http status of 401 (unauthorized)
and will carry the following JSON data:

.. code-block:: json

  {"status":"Access denied"}


Log out
-------

Interestingly, a log out request will always succeed - regardless of
whether you have previously logged in.  It can thus be used to return
a session to a known state.

The following curl command will effect a logout:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/logout

and the response will be:

.. code-block:: json

  {"status":"OK"}

with an http status of 200 (OK).


Change user
-----------

A suitably privileged API user can also switch user id in order to
make changes on behalf of another user.  This can be useful if you
are writing a general application, through which multiple users
can create events in Scheduler.

In order to make use of any of the following facilities, your API
user must also have the "can_su" privilege bit set.  Without that,
all the following requests will be rejected with a status of "Forbidden".

Find user id
------------

Given a user's email address, you can find the corresponding internal
user id with a call like the following:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/api/users?email=claire.dunwoody@xronos.uk

This is asking for a listing of all users with the email address
"claire.dunwoody@xronos.uk".

Assuming you have the can_su privilege and the requested user exists,
the response will look like this:

.. code-block:: json

  {
    "status":"OK",
    "users":[
      {
        "id":2,
        "name":"Claire Dunwoody",
        "email":"claire.dunwoody@xronos.uk",
        "valid":true
      }
    ]
  }

The user id shown in this response is what you need to effect an su.

Su
--

The call to change user id looks like you're simply writing a new
user_id to the session.  What happens behind the scenes is slightly
more complicated.  You can have only one session at a time so the
session id in the request URL is irrelevant and ignored.  Leave it as
1.

Provided your request passes the necessary authorisation checks,
the user_id is not actually simply written to the session.  The whole
session record is reset and then you are logged in as the new
user (with a note being kept of who you were before).  For this
reason, the response will include a new session cookie and it's important
to keep that, just as you did when you first logged on.  Without that,
you can't issue any more requests.

Whilst your session is in an su'ed state, permission checking changes
slightly.  Your original user is the one which has permission to use
the API, but your new effective user is the one which will be checked
for everything else.

It is not possible to make nested su requests.  You need to revert the
first one before you can make another one.

To switch to having an effective user of Claire Dunwoody, issue
the following request:

.. code-block:: bash

  curl -K curl.opt -c cookies.jar --request PUT --data '{"session":{"user_id":"2"}}' https://schedulerdemo.xronos.uk/api/sessions/1

Linguistically, this is saying "Amend session 1 and set the value of user_id to be 2 within that session".  The server will take it as a request to switch to being user 2.

Assuming you have the necessary permission, the response will be:

.. code-block:: json

  {"status":"OK"}


Who am I
--------

You can check what your current effective user id is with the following
call:

.. code-block:: bash

  curl -K curl.opt https://schedulerdemo.xronos.uk/sessions/1/whoami

which will get a response like:

.. code-block:: json

  {"status":"OK","user_id":2}


Reverting su
------------

Having finished the work you need to do as a different user you can go
back to your original user id by issuing:

.. code-block:: bash

  curl -K curl.opt -c cookies.jar --request PUT --data '{}' https://schedulerdemo.xronos.uk/api/sessions/1/revert

and again the response will be:

.. code-block:: json

  {"status":"OK"}

You can also get back to being your original user by issuing a fresh su
request specifying your original user id.  This is the only case where
a nested su request will work.  The system notices you're trying to
set your user id back to what it was before and does the "Revert su"
processing.

Again, it is necessary to save the new session cookie.  Reverting also
causes your session to be completely reset and so the session cookie
changes.


