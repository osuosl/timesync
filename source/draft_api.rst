.. _draft_api:

=========
Draft API
=========

Below are the API specs for the TimeSync project.

.. contents::

----------

Connection
----------

All requests will be made via HTTPS. Available methods are ``GET`` to request
an object, ``POST`` to create and/or edit a new object, and ``DELETE`` to
remove an object.

------

Format
------

Responses will be returned in standard JSON format. Multiple results will be
sent as a list of JSON objects. Order of results is not guaranteed. Single
results will be returned as a single JSON object.


.. note::

    Throughout this API, any form of dates will use a simplified ISO-8601
    format, as `defined by ECMA International.
    <http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15>`_

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

When an object is first created, it is assigned a tracking ID. This is a UUID
which will refer to all versions of the same object.

When an object is updated, a new revision is created. This allows one to easily
keep track of the changes to an object over time (its *audit trail*). This
means that a database key such as an auto-assigned ID is relatively meaningless
in referring to an object, as it will only point to a revision.

A revision can be referred to by its unique compound key (UUID, revision),
where revision is a number which refers to the position of that version of the
object in the audit trail (where 1 is the original version from object
creation, 2 is created after the first update, etc.). This revision number is
re-used between objects.

----------------

Auditing History
----------------

There are three variables in all objects that assist in an audit process
(vewing revisions of an object through it's history).

* ``created_at``: the date at which a given object (specified by a uuid) was
  created.
* ``updated_at``: The date at which an object was modified (the created_at date
  of a new object revision).
* ``deleted_at``: When the DELETE operation was performed on an object (either
  the object is deleted or depricated for a new version of the object).
  Depricated version of an object will include a deleted_at field but will
  appear in an audit trail (the ``parent`` field of an object)

**To view the audit trail of an object pass the ``?revisions=true`` parameter
to any endpoint.**

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
        "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
        "created_at": 2014-04-17,
        "deleted_at": null,
        "updated_at": 2014-04-19,
        "revision": 2,
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
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 4,
      "created_at": 2014-07-17,
      "deleted_at": null,
      "updated_at": 2014-07-20,
    }

*GET /activities*

.. code-block:: javascript

    [
      {
        "name":"Documentation",
        "slugs":["docs", "doc"],
        "uuid": "adf036f5-3d49-4a84-bef9-062b46380bbf",
        "revision": 1,
        "created_at": 2014-04-17,
        "deleted_at": null,
        "updated_at": null,
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
      "created_at": 2014-04-17,
      "deleted_at": null,
      "updated_at": null,
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
        "revision": 1,
        "created_at":2014-04-17,
        "updated_at":null,
        "deleted_at": null,
        "uuid": "c3706e79-1c9a-4765-8d7f-89b4544cad56",
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
      "deleted_at": null,
      "uuid": "c3706e79-1c9a-4765-8d7f-89b4544cad56",
      "revision": 3,
    }

In addition, the endpoint at ``/times`` also supports several querystring
parameters:

* user
* project
* activity
* date range

These are accessed via

* ``/times?user=:username``: Filters based on username
* ``/times?project=:projectslug``: Filters based on project slugs
* ``/times?activity=:activityslug``: Filters based on activity slug
* ``/times?start=:date``: Filters to dates after and including the given date.
* ``/times?end=:date``:  Filters to dates after and including the given date.
* ``/times/?revisions=:bool``: Returns objects and an audit list for that
  object in the form of a list ``parent``.


For example:

``GET /projects/<slug>?archived=true``:

.. code-block:: javascript

    {
      "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
      "name":"Ganeti Web Manager",
      "slugs":["ganeti", "gwm"],
      "owner": "example-user",
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 4,
      "created_at": 2015-04-17,
      "deleted_at": null,
      "updated_at": null,
      "parent":
      [
        {
          "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
          "name":"Ganeti Web Manager",
          "slugs":["ganeti", "gwm"],
          "owner": "example-user",
          "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
          "revision": 3,
          "created_at": 2015-04-16,
          "deleted_at": 2015-04-17,
          "updated_at": 2015-04-17
        },
        {...},
        {...},
      ],
    }

``GET /times/<uuid>?archived=true``:

.. code-block:: javascript

    {
      "duration":20,
      "user": "example-user",
      "project": "gwm",
      "activities": ["doc", "research"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked":2015-04-17,
      "created_at":2014-06-12,
      "updated_at":2015-04-18,
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "deleted_at": null,
      "revision":2,
      "parent":
        [
          {
            "duration":20,
            "user": "example-user",
            "project": "gwm",
            "activities": ["doc", "research"],
            "notes":"Worked on documentation toward settings configuration.",
            "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
            "date_worked":2015-04-17,
            "created_at":2014-06-12,
            "updated_at":null,
            "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
            "deleted_at": 2014-04-18,
            "revision":1,
          },
        ],
    }

``GET /activities/<slug>?archived=true``

.. code-block:: javascript

    {
      "name":"Testing Infra",
      "slugs":["testing", "test"],
      "updated_at": 2015-04-18,
      "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
      "created_at": 2014-04-17,
      "deleted_at": null,
      "updated_at": 2014-04-18,
      "revision":2,
      "parent":
        [
          {
            "name":"Testing Infrastructure",
            "slugs":["testing", "tests"],
            "created_at": 2015-04-17,
            "deleted_at": 2015-04-18,
            "updated_at": null,
            "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
            "deleted_at": null,
            "revision":1,
          }
        ]
    }

When multiple different parameters are used, they narrow down the result set
(for example, ``/times?user=example-user&activity=dev`` will return all time
entries which were entered by example-user AND which were spent doing
development). When the same parameter is repeated, they expand the result set
(for example, ``/times?activity=gwm&activity=pgd`` will return all time entries
which were either for gwm OR pgd). Date ranges are inclusive on both ends.

* If a query parameter is provided with a bad value (e.g. invalid slug, or date
  not in ISO-8601 format), a Bad Query Value error is returned.
* Any query parameter other than those specified in this document will be
  ignored.
* For more information about errors, check the
  :ref:`draft_errors<draft_errors>` docs.

If multiple ``start`` or ``end`` parameters are provided, the first one sent is
used. If a query parameter is not provided, it defaults to 'all values'.

.. note::

    All endpoints (``/activites/``, ``/activities/:slug``, ``/projects/``,
    ``/projects/:slug``, ``/times/``, ``/times/:uuid``) support the
    ``?archived`` flag which will show the full audit trail of an object.

--------------

POST Endpoints
--------------

To add a new object, POST to */<object name>/* with a JSON body. The response
body will contain the object in the same manner as the GET endpoints above.

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
       "owner":"example-2",
       "uuid":"b35f9531-517f-47bd-aab4-14298bb19555",
       "created_at":2014-04-17,
       "updated_at":null,
       "deleted_at":null,
       "revision":1,
    }

*POST /activities/*
~~~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
       "name":"Quality Assurance/Testing",
       "slugs":["qa", "test"]
    }

Response body:

.. code-block:: javascript

    {
       "name":"Quality Assurance/Testing",
       "slugs":["qa", "test"],
       "uuid": "cfa07a4f-d446-4078-8d73-2f77560c35c0",
       "created_at": 2014-04-17,
       "updated_at": 2014-04-18,
       "deleted_at": null,
       "revision":2,
    }


*POST /times/*
~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-2",
      "project": "ganet_web_manager",
      "activities": ["documenting"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osu-cass/whats-fresh-api/issues/56",
      "date_worked":2014-04-17,
    }

Response body:

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-2",
      "project": "ganet_web_manager",
      "activities": ["documenting"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/56",
      "date_worked":2014-04-17,
      "created_at":2014-04-17,
      "updated_at": null,
      "deleted_at": null,
      "uuid": "838853e3-3635-4076-a26f-7efe4e60981f",
      "revision":1,
    },

Likewise, if you'd like to edit an existing object, POST to ``/<object
name>/<slug>`` (or for time objects, ``/times/<uuid>``) with a JSON body.  The
object only needs to contain the part that is being updated. The response body
will contain the saved object, as shown above.


*POST /projects/<slug>*
~~~~~~~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
       "name":"Ganeti Webmgr",
       "slugs":["webmgr", "gwm"],
    }

Response body:

.. code-block:: javascript

    {
      "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
      "name":"Ganeti Webmgr",
      "slugs":["webmgr", "gwm"],
      "owner": "example-user",
      "created_at": 2014-04-16,
      "updated_at": 2014-04-18,
      "deleted_at": null,
      "uuid": "309eae69-21dc-4538-9fdc-e6892a9c4dd4",
      "revision":2,
    }

*POST /activities/<slug>*
~~~~~~~~~~~~~~~~~~~~~~~~~

Request body:


.. code-block:: javascript

    {
      "slugs":["testing", "test"]
    }

Response body:

.. code-block:: javascript

    {
      "name":"Testing Infra",
      "slugs":["testing", "test"],
      "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
      "created_at": 2014-04-16,
      "updated_at": 2014-04-17,
      "deleted_at": null,
      "revision":2,
    }

*POST /times/<uuid>*
~~~~~~~~~~~~~~~~~~~~

Request body:


.. code-block:: javascript

    {
      "duration":20,
      "date_worked":"2015-04-17"
    }

Response body:

.. code-block:: javascript

    {
      "duration":20,
      "user": "example-user",
      "project": "gwm",
      "activities": ["doc", "research"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked":2015-04-17,
      "created_at":2014-06-12,
      "updated_at":2015-04-18,
      "deleted_at": null,
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "revision":2,
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

.. note::

    While they won't produce an error, empty data structures such as ``""`` and
    ``[]`` will be ignored when sent to the api as the value of an object
    variable.

.. note::

   When an object is updated it's ``parent`` is soft-deleted and a copy is
   created with the new information. This results in the object having an
   incremented 'revision' field the updated information specifed in the POST
   request.


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

If the object exists and the ``?archived=true`` parameter is not passed, the
API will return a 200 OK status with an empty response body.

If the object does not exist and the ``?archived=true`` parameter is not
passed, the API will return an Object Not Found error (see error docs).

In case of any other error, the API will return a Server Error (see error
docs).

-----------------------

Authorization and Roles
-----------------------

Each timesync user can be of one of two roles: user, and admin. Admins have
special permissions, including adding, updating, and deleting activities and
projects, creating and promoting users, as well as acting as automatic
managers/viewers of all projects.

In addition, each user has a role within each project to which they belong:

* member
* data viewer
* project manager.

These roles exist independently, and are defined by their permissions:

* a member has permission to write time entries
* a data viewer may view time entries
* a project manager may update the project information.

A user may be a member, viewer, or manager of multiple projects, and a project
may have multiple members, viewers, and managers.

If a user attempts to access an endpoint which they are not authorized for, the
server will return an Authorization Failure.

*GET Endpoints*
~~~~~~~~~~~~~~~

GET endpoints do not have authorization at this time, and so any user can
request data from a GET endpoint.

*POST and DELETE Endpoints*
~~~~~~~~~~~~~~~~~~~~~~~~~~~

POST /activities, POST /activities/:slug, and DELETE /activities/:slug are all
only accessible to admin users.

POST /projects and DELETE /projects/:slug are only accessible to admin users.
POST /projects/:slug is accessible to that project's manager(s).

POST /times is accessible to that project's member(s), given that the 'user'
field of the posted time is the user authenticating.
