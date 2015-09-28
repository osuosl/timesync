.. _draft_api:

=========
Draft API
=========

Below are the API specs for the TimeSync project.

.. contents::

----------

Connection
----------

All requests will be made via HTTPS. Available methods are GET to request an
object, POST to create and/or edit a new object, and DELETE to remove an
object.

------

Format
------

Responses will be returned in standard JSON format. Multiple results will be
sent as a list of JSON objects. Order of results is not guaranteed. Single
results will be a single JSON object.

Throughout this API, any form of dates will use a simplified ISO-8601 format, as `defined
by ECMA International. <http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15>`_

--------

Versions
--------

The API will be versioned with the letter 'v' followed by increasing integers.

For example: https://timesync.osuosl.org/v1/projects

Versions will be updated any time there is a significant change to the public
API (not to the implementation).

-----

Slugs
-----

Slugs appear in many places in TimeSync. They are used to get objects from the
backend, reference objects from within other objects, etc. A valid slug follows
a very specific format:

#) May only contain lowercase letters and numbers
#) Sets of lowercase letters and numbers can be separated with a single hyphen
#) Must contain at least one letter

---------

Revisions
---------

When an object is first created, it is assigned a tracking ID. This is a UUID which will
refer to all versions of the same object.

When an object is updated, a new revision is created. This allows one to easily keep track
of the changes to an object over time (its *audit trail*). This means that a database key
such as an auto-assigned ID is relatively meaningless in referring to an object, as it
will only point to a revision.

Instead, a revision can be referred to by its unique compound key (UUID, revision), where
revision is a number which refers to the position of that version of the object in the
audit trail (where 1 is the original version from object creation, 2 is created after the
first update, etc.). This revision number is re-used between objects.

-------------

GET Endpoints
-------------

*GET /projects*

.. code-block:: javascript

    [
      {
         "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
         "name":"Ganeti Web Manager",
         "slugs":["gwm", "ganeti"],
         "owner": "example-user",
         "uuid": a034806c-00db-4fe1-8de8-514575f31bfb,
         "revision": 2,
         "id": 1
      },
      {...},
      ...
    ]

*GET /projects/<slug>*

.. code-block:: javascript

    {
       "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
       "name":"Ganeti Web Manager",
       "slugs":["ganeti", "gwm"],
       "owner": "example-user",
       "uuid": a034806c-00db-4fe1-8de8-514575f31bfb,
       "revision": 4,
       "id": 1
    }

*GET /activities*

.. code-block:: javascript

    [
        {
           "name":"Documentation",
           "slugs":["docs", "doc"],
           "uuid": adf036f5-3d49-4a84-bef9-062b46380bbf,
           "revision": 1,
           "id": 1
        },
        {...}
    ]

*GET /activities/<slug>*

.. code-block:: javascript

    {
       "name":"Documentation",
       "slugs":["doc", "docs"],
       "uuid": adf036f5-3d49-4a84-bef9-062b46380bbf,
       "revision": 5,
       "id": 1
    }

*GET /times*

.. code-block:: javascript

    [
      {
        "duration":12,
        "user": "example-user",
        "project": "ganeti",
        "activities": ["docs", "planning"],
        "notes":"Worked on documentation toward settings configuration.",
        "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
        "date_worked":2014-04-17,
        "created_at":2014-04-17,
        "updated_at":null,
        "uuid": c3706e79-1c9a-4765-8d7f-89b4544cad56,
        "revision": 1,
        "id": 1
      },
      {...}
    ]

*GET /times/<time entry uuid>*

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-user",
      "project": "gwm",
      "activities": ["doc", "research"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked":2014-06-12,
      "created_at":2014-06-12,
      "updated_at":2014-06-13,
      "uuid": c3706e79-1c9a-4765-8d7f-89b4544cad56,
      "revision": 3,
      "id": 1
    }

In addition, the endpoint at ``/times`` also supports several querystring parameters: user,
project, activity, and date range. These are accessed via ``/times?user=:username``,
``/times?project=:projectslug``, ``/times?activity=:activityslug``, ``/times?start=:date``, and
``/times?end=:date`` (note that dates are in ISO-8601 format). When multiple different
parameters are used, they narrow down the result set (for example,
``/times?user=example-user&activity=dev`` will return all time entries which were entered by
example-user AND which were spent doing development). When the same parameter is repeated,
they expand the result set (for example, ``/times?activity=gwm&activity=pgd`` will return all
time entries which were either for gwm OR pgd). Date ranges are inclusive on both ends.

If a query parameter is provided with a bad value (e.g. invalid slug, valid but non-existent
slug, or date not in ISO-8601 format), a Bad Query Value error is returned. This is also
true when multiple other valid parameters are provided. Any query parameter other than those
specified in this document will be ignored. If multiple ``start`` or ``end`` parameters are provided,
the first one sent is used. If a query parameter is not provided, it defaults to 'all values'.

The endpoint at ``/times/:uuid`` also supports several querystring parameters:

* action
* revision

The ``action`` parameter can take one of three values:

* recent (default)
* trail
* version

If ``action`` is ``recent`` (or is not provided), the most recent revision of the object
is returned.

If it is ``trail``, the most recent revision of the object is returned, also
containing a ``parent`` field, which will be a list of previous versions in reverse
chronological order (i.e. the most recent revision at index 0, etc.), similar to the
structure returned from /times.

If ``action`` is ``version`` and a value for ``revision`` is provided, that revision of
the object is returned.

--------------

POST Endpoints
--------------

To add a new object, POST to */<object name>/* with a JSON body. The response
body will contain the object in the same manner as the GET endpoints above.

In general, the only difference between the request body and the response body
will be the inclusion of the object's ``id``.

*POST /projects/*
~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
       "uri":"https://code.osuosl.org/projects/timesync",
       "name":"TimeSync API",
       "slugs":["timesync", "time"],
       "owner": "example-2"
    }

Response body:

.. code-block:: javascript

    {
       "uri":"https://code.osuosl.org/projects/timesync",
       "name":"TimeSync API",
       "slugs":["timesync", "time"],
       "owner": "example-2",
       "id": 1
    }

*POST /activities/*
~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
       "name":"Quality Assurance/Testing",
       "slugs":["qa", "test"]
    }

*POST /times/*
~~~~~~~~~~~~~~

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-2",
      "project": "",
      "activities": ["gwm", "ganeti"],
      "notes":"",
      "issue_uri":"https://github.com/osu-cass/whats-fresh-api/issues/56",
      "date_worked":null,
      "created_at":2014-09-18,
      "updated_at":null
    }

Likewise, if you'd like to edit an existing object, POST to
*/<object name>/<slug>* (or for time objects, */times/<id>*) with a JSON body.
The object only needs to contain the part that is being updated. The response
body will contain the saved object, as shown above.


*POST /projects/<slug>*
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
       "name":"Ganeti Webmgr",
       "slugs":["webmgr", "gwm"],
    }

*POST /activities/<slug>*
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
       "slugs":["testing", "test"]
    }

*POST /times/<uuid>*
~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
      "duration":20,
      "date_worked":"2015-04-17"
    }

In the case of a foreign key (such as project on a time) that does not point to
a valid object or a malformed object sent in the request, an Object Not Found
or Malformed Object error (respectively) will be returned, validation will
return immediately, and the object will not be saved.

The following content is checked by the API for validity:

* Time/Date must be a valid ISO 8601 Date/Time.
* URI must be a valid URI.
* Activities must exist in the database.
* The Project must exist in the database.
* The owner of the request must be the user in the time submission.
    * This is authorization not authentication.

----------------

DELETE Endpoints
----------------

A DELETE request sent to any object's endpoint (e.g. */projects/<slug>*) will
result in the deletion of the object from the records. It is up to the
implementation to decide whether to use hard or soft deletes. What is important
is that the object will not be included in requests to retrieve lists of
objects, and attempts to access the object will fail. Future attempts to POST
an object with that UUID/slug should succeed, and completely overwrite the
deleted object, if it still exists in the database. To an end user, it should
appear as though the object truly does not exist.

If the object exists, the API will return a 200 OK status with an empty
response body.

If the object does not exist, the API will return an Object Not Found error
(see error docs).

In case of any other error, the API will return a Server Error (see error
docs).

-----------------------

Authorization and Roles
-----------------------

Each timesync user can be of one of two roles: user, and admin. Admins have special
permissions, including adding, updating, and deleting activities and projects, creating
and promoting users, as well as acting as automatic managers/viewers of all projects.

In addition, each user has a role within each project to which they belong:

* member
* data viewer
* project manager.

These roles exist independently, and are defined by their permissions:

* a member has permission to write time entries
* a data viewer may view time entries
* a project manager may update the project information.

A user may be a member, viewer, or manager of multiple projects, and a project may have
multiple members, viewers, and managers.

If a user attempts to access an endpoint which they are not authorized for, the server
will return an Authorization Failure.

*GET Endpoints*
~~~~~~~~~~~~~~~

GET endpoints do not have authorization at this time, and so any user can request data
from a GET endpoint.

*POST and DELETE Endpoints*
~~~~~~~~~~~~~~~~~~~~~~~~~~~

POST /activities, POST /activities/:slug, and DELETE /activities/:slug are all only
accessible to admin users.

POST /projects and DELETE /projects/:slug are only accessible to admin users.
POST /projects/:slug is accessible to that project's manager(s).

POST /times is accessible to that project's member(s), given that the 'user' field of
the posted time is the user authenticating.
