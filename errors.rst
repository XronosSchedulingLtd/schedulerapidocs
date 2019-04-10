Error codes
===========

The following error codes are used by the Scheduler API

Each time an error occurs it will be indicated both by a numeric http
status code returned with the response, and by a textual status within
the response.

The error codes returned by the Scheduler API are as follows:

.. list-table:: Error codes
   :widths: 30 80 170
   :header-rows: 1

   * - Value
     - Text
     - Meaning
   * - 200
     - OK
     - The request completed successfully
   * - 201
     - Created
     - The item requested was created
   * - 400
     - Bad request
     - The request did not make enough sense to be processed
   * - 401
     - Access denied
     - The client has not yet authenticated as a suitable user
   * - 403
     - Forbidden
     - The client has authenticated, but does not have permission to
       perform the requested action
   * - 404
     - Not found
     - The item requested could not be found
   * - 405
     - Method not allowed
     - The client has requested an action which isn't consistent.
       E.g. to create a Request record for something which doesn't
       have requests.
   * - 422
     - Unprocessable entity
     - The parameters given on an action make it impossible to
       complete.  E.g. the parameters to create an event are
       incomplete or inconsistent.  See the JSON response for
       more details of exactly what is wrong.
   * - 503
     - Service unavailable
     - The API is unable to handle the creation of an event, probably
       because the appropriate eventsource does not exist in the
       system.



