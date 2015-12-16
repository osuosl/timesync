.. _draft_api:

=========
Draft API
=========

Below are the API specs for the TimeSync project.

.. contents::

.. note::

    Variables are indicated with the ``:variable_name`` syntax
    ([colon][variable name]). If you see something like ``:time`` or ``:slug``
    being referenced it is not the literal string ':slug' and ':time' but a
    variable.

----------

Connection
----------

All requests will be made via HTTPS. Available methods are GET to request
an object, POST to create and/or edit a new object, and DELETE to
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
back-end, reference objects from within other objects, etc. A valid slug follows
a very specific format:

#) May only contain lowercase letters and numbers
#) Sets of lowercase letters and numbers can be separated with a single hyphen
#) Must contain at least one letter

---------

Revisions
---------

When an object is first created, it is assigned a tracking ID. This is a UUID
which will refer to all versions of the same object. For example:

.. code-block:: none

     de305d54-75b4-431b-adb2-eb6b9e546014

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
(viewing revisions of an object through its history).

* ``created_at``: the date at which a given object (specified by a UUID) was
  created.
* ``updated_at``: The date at which an object was modified (the created_at date
  of a new object revision).
* ``deleted_at``: When the DELETE operation is performed on an object it's
  ``deleted_at`` field is set to the date it was deleted. Historical
  (``parents``) copies of an object do not have ``deleted_at`` set unless the
  object was deleted for a given historical copy (and later un-deleted).


**To view the audit trail of an object pass the** ``?include_revisions=true``
**parameter to any endpoint and inspect the** ``parents`` **variable (a list of
object revisions).**

-------------

GET Endpoints
-------------

GET /projects
~~~~~~~~~~~~~

.. code-block:: javascript

    [
      {
        "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
        "name":"Ganeti Web Manager",
        "slugs":["gwm", "ganeti"],
        "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": "2014-04-19",
        "revision": 2,
        "users": {
          "members": [
            "patcht"
          ],
          "spectators": [
          ],
          "managers": [
            "tschuy"
          ]
        }
      },
      {...}
    ]

GET /projects/:slug
~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
      "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
      "name":"Ganeti Web Manager",
      "slugs":["ganeti", "gwm"],
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 4,
      "created_at": "2014-07-17",
      "deleted_at": null,
      "updated_at": "2014-07-20",
      "users": {
        "members": [
          "patcht"
        ],
        "spectators": [
        ],
        "managers": [
          "tschuy"
        ]
      }
    }

GET /activities
~~~~~~~~~~~~~~~

.. code-block:: javascript

    [
      {
        "name":"Documentation",
        "slugs":["docs", "doc"],
        "uuid": "adf036f5-3d49-4a84-bef9-062b46380bbf",
        "revision": 1,
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": null
      },
      {...}
    ]

GET /activities/:slug
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
      "name":"Documentation",
      "slugs":["doc", "docs"],
      "uuid": adf036f5-3d49-4a84-bef9-062b46380bbf,
      "revision": 5,
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": null
    }

GET /times
~~~~~~~~~~

.. code-block:: javascript

    [
      {
        "duration":12,
        "user": "example-user",
        "project": ["ganeti-webmgr", "gwm"],
        "activities": ["docs", "planning"],
        "notes":"Worked on documentation toward settings configuration.",
        "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
        "date_worked":"2014-04-17",
        "revision": 1,
        "created_at":"2014-04-17",
        "updated_at":null,
        "deleted_at": null,
        "uuid": "c3706e79-1c9a-4765-8d7f-89b4544cad56"
      },
      {...}
    ]

GET /times/:time-entry-uuid
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-user",
      "project": ["gwm", "ganeti-webmgr"],
      "activities": ["doc", "research"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked":"2014-06-12",
      "created_at":"2014-06-12",
      "updated_at":"2014-06-13",
      "deleted_at":null,
      "uuid": c3706e79-1c9a-4765-8d7f-89b4544cad56,
      "revision": 3
    }

----------------------------

GET Request Query Parameters
----------------------------

TimeSync's response data can be narrowed even further than the /:endpoints
return statements by adding parameters.

==== ======= ======== ========== ================ ===============
user project activity date range object revisions deleted objects
==== ======= ======== ========== ================ ===============

Reference Table
~~~~~~~~~~~~~~~

=================== ======================= =======================
Parameter           Value(s)                Endpoint(s)
=================== ======================= =======================
?user=              :username               /times
?project=           :projectslug            /times
?activity=          :activityslug           /times
?start=             :date (iso format)      /times
?end=               :date (iso format)      /times
?include_revisions= :bool                   - /times
                                            - /times/:uuid
                                            - /activities/
                                            - /activities/:slug
                                            - /projects/
                                            - /projects/:slug
?include_deleted=   :bool                   - /times
                                            - /times/:uuid
                                            - /activities
                                            - /projects
=================== ======================= =======================

?user=:username
~~~~~~~~~~~~~~~

``/times?user=:username``
    Filters results to a set of time submitted entries by a specified user.

?project=:projectslug
~~~~~~~~~~~~~~~~~~~~~

``/times?project=:projectslug``
    Filters results to a set of time entries of a specified project slug.

?activity=:activityslug
~~~~~~~~~~~~~~~~~~~~~~~

``/times?activity=:activityslug``
    Filters results to a set of time entries with a specified activity slug.

?start=:date
~~~~~~~~~~~~

``/times?start=:date``
    Filters results to a set of time entries after a specified date.

``/times?end=:date&start=:date``
    Can be combined with ?end to create a date range.

?end=:date
~~~~~~~~~~

``/times?end=:date``
    Filters results to a set of time entries before a specified date.

``/times?start=:date&end=:date``
    Can be combined with ?start to create a date range.

?include_revisions=:bool
~~~~~~~~~~~~~~~~~~~~~~~~

``/times?include_revisions=:bool``
    * Adds the 'parents' field to the specified object.
    * This field is a list of all previous revisions of the object in
      descending order by revision number (i.e. ``time.parents[0]`` will be the
      previous revision, and ``time.parents[n-1]`` will be the first revision).
    * Without this field the object(s) do not include a 'parents' field and so
      only the most recent revision of the object will be seen.

?include_deleted=:bool
~~~~~~~~~~~~~~~~~~~~~~

Includes deleted entries in the returned results.
    These are objects which have the 'deleted_at' parameter set to an ISO date
    (i.e., a non-null value).

Multiple Parameters Per Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When multiple parameters are used, they narrow down the result set

.. code-block:: none

    $ GET /times?user=example-user&activity=dev&token=...
    # This will return all time entries which were entered by example-user AND
    # which were spent doing development.

When the same parameter is repeated, they expand the result set

.. code-block:: none

    $ GET /times?project=gwm&project=pgd&token=...
    # This will return all time entries which were either for gwm OR pgd.

Date ranges are inclusive on both ends.

Malformed or Exceptional Parameter Usage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a query parameter is provided with a bad value (e.g. invalid slug, or date
not in ISO-8601 format), a Bad Query Value error is returned.

Any query parameter other than those specified in this document will be
ignored.

For more information about errors, check the :ref:`draft_errors<draft_errors>`
docs.

If multiple ``start`` or ``end`` parameters are provided, the first one sent is
used. If a query parameter is not provided, it defaults to 'all values'.

Including Revisions of Objects (include_revisions)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

GET /projects/:slug?include_revisions=true
++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    {
      "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
      "name":"Ganeti Web Manager",
      "slugs":["ganeti", "gwm"],
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 4,
      "created_at": "2015-04-16",
      "deleted_at": null,
      "updated_at": "2015-04-17",
      "users": {
        "members": [
          "patcht"
        ],
        "spectators": [
        ],
        "managers": [
          "tschuy"
        ]
      },
      "parents":
      [
        {
          "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
          "name":"Ganeti Web Manager",
          "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
          "revision": 3,
          "created_at": "2015-04-16",
          "deleted_at": null,
          "updated_at": null,
          "users": {
            "members": [
            ],
            "spectators": [
            ],
            "managers": [
              "tschuy"
            ]
          }
        },
        {...},
        {...}
      ]
    }

GET /times/:uuid?include_revisions=true
+++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    {
      "duration":20,
      "user": "example-user",
      "project": "gwm",
      "activities": ["doc", "research"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked":"2015-04-18",
      "created_at":"2014-06-12",
      "updated_at":"2015-04-18",
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "deleted_at": null,
      "revision":2,
      "parents":
        [
          {
            "duration":20,
            "user": "example-user",
            "project": "gwm",
            "activities": ["doc", "research"],
            "notes":"Worked on documentation toward settings configuration.",
            "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/40",
            "date_worked":"2015-04-17",
            "created_at":"2014-06-12",
            "updated_at":null,
            "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
            "deleted_at": null,
            "revision":1
          }
        ]
    }

GET /activities/:slug?include_revisions=true
++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    {
      "name":"Testing Infra",
      "slug":"test",
      "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": "2014-04-18",
      "revision":2,
      "parents":
        [
          {
            "name":"Testing Infrastructure",
            "created_at": "2014-04-17",
            "deleted_at": null,
            "updated_at": null,
            "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
            "revision":1
          }
        ]
    }

GET /activities?include_revisions=true
++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    [
      {
        "name":"Testing Infra",
        "slug":"test",
        "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": "2014-04-18",
        "revision":2,
        "parents":
          [
            {
              "name":"Testing Infrastructure",
              "created_at": "2014-04-17",
              "deleted_at": null,
              "updated_at": null,
              "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
              "revision":1
            }
          ]
      },
      {
        "name":"Build Infra",
        "slug":"build",
        "uuid": "e81e45ef-e7a7-4da2-88cd-9ede610c5896",
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": "2014-04-23",
        "revision":2,
        "parents":
          [
            {
              "name":"Testing Infrastructure",
              "created_at": "2014-04-17",
              "deleted_at": null,
              "updated_at": null,
              "uuid": "e81e45ef-e7a7-4da2-88cd-9ede610c5896",
              "revision":1
            }
          ]
      }
    ]

Retrieving Deleted Objects (include_deleted)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Alongside revision history, you can also view objects that have been
soft-deleted. To view an object that has been soft deleted, send a GET request
with the ``?include_deleted`` parameter set to true. Doing so will return all
objects matching the query, both current and deleted.

.. note::

    When passing the ``include_deleted`` parameter to your request, note that
    you cannot specify a project/activity by their slug. This is because slugs
    are permanently deleted from activities and projects when they are deleted,
    in order to allow slug re-use.

GET /projects?include_deleted=true
++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    [
      {
        "uri":"https://code.osuosl.org/projects/ganeti-webmgr",
        "name":"Ganeti Web Manager",
        "slugs":["ganeti", "gwm"],
        "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
        "revision": 4,
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": null
      },
      {...},
      {...},
      {
        "uri":"https:://github.com/osuosl/timesync",
        "name":"Timesync",
        "slugs":["ts", "timesync"],
        "uuid": "1f8788bd-0909-4397-be2c-79047f90c575",
        "revision": 1,
        "created_at": "2014-04-17",
        "deleted_at": "2015-10-01",
        "updated_at": null
      }
    ]

GET /activities?include_deleted=true
++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    [
      {
        "name":"Documentation",
        "slug":"doc",
        "uuid": "adf036f5-3d49-4a84-bef9-062b46380bbf",
        "revision": 5,
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": null
      },
      {...},
      {...},
      {
        "name": "Meetings"
        "slugs": "meeting",
        "uuid": "6552d14e-12eb-4f1f-83d5-147f8452614c",
        "revision": 1,
        "created_at": "2014-04-17",
        "deleted_at": "2015-05-01",
        "updated_at": null
      }
    ]

--------------

POST Endpoints
--------------

To add a new object, POST to */:object-name/* with a JSON body. The response
body will contain the object in the same manner as the GET endpoints above.

POST /projects/
~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
       "uri":"https://code.osuosl.org/projects/timesync",
       "name":"TimeSync API",
       "slugs":["timesync", "time"],
       "owner": "example-2",
       "users": {
         "members": [
           "patcht"
         ],
         "spectators": [
         ],
         "managers": [
           "tschuy"
         ]
       }
    }

Response body:

.. code-block:: javascript

    {
       "uri":"https://code.osuosl.org/projects/timesync",
       "name":"TimeSync API",
       "slugs":["timesync", "time"],
       "uuid":"b35f9531-517f-47bd-aab4-14298bb19555",
       "created_at":"2014-04-17",
       "updated_at":null,
       "deleted_at":null,
       "revision":1,
       "users": {
         "members": [
           "patcht"
         ],
         "spectators": [
         ],
         "managers": [
           "tschuy"
         ]
       }
    }

Note that this endpoint, when called, will automatically set the currently
authenticated user as a member, spectator, and manager of the project, allowing
them to update and delete the project, add members to it, and promote/demote
user roles on the project.

POST /activities/
~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
       "name":"Quality Assurance/Testing",
       "slug":"qa"
    }

Response body:

.. code-block:: javascript

    {
       "name":"Quality Assurance/Testing",
       "slug":"qa",
       "uuid": "cfa07a4f-d446-4078-8d73-2f77560c35c0",
       "created_at": "2014-04-17",
       "updated_at": null,
       "deleted_at": null,
       "revision":2
    }


POST /times/
~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-2",
      "project": "ganeti_web_manager",
      "activities": ["docs"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osu-cass/whats-fresh-api/issues/56",
      "date_worked":"2014-04-17"
    }

Response body:

.. code-block:: javascript

    {
      "duration":12,
      "user": "example-2",
      "project": "ganeti_web_manager",
      "activities": ["docs"],
      "notes":"Worked on documentation toward settings configuration.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/56",
      "date_worked":"2014-04-17",
      "created_at":"2014-04-17",
      "updated_at": null,
      "deleted_at": null,
      "uuid": "838853e3-3635-4076-a26f-7efe4e60981f",
      "revision":1
    }

Likewise, if you'd like to edit an existing object, POST to
``/:object-name/:slug`` (or for time objects, ``/times/:uuid``) with a JSON
body.  The object only needs to contain the part that is being updated. The
response body will contain the saved object, as shown above.


POST /projects/:slug
~~~~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
       "uri":"https://code.osuosl.org/projects/timesync",
       "name":"TimeSync API",
       "slugs":["timesync", "time"]
    }

Response body:

.. code-block:: javascript

    {
      "uri":"https://code.osuosl.org/projects/timesync",
      "name":"TimeSync API",
      "slugs":["timesync", "time"],
      "created_at": "2014-04-16",
      "updated_at": "2014-04-18",
      "deleted_at": null,
      "uuid": "309eae69-21dc-4538-9fdc-e6892a9c4dd4",
      "revision":2,
      "users": {
        "members": [
          "patcht"
        ],
        "spectators": [
        ],
        "managers": [
          "tschuy"
        ]
      }
    }

If a value of ``""`` (an empty string) or ``[]`` (an empty array) are passed as
values for a string or array optional field (check the :ref:`model docs<draft_model>`),
the value will be set to the empty string/array. If a value of undefined is provided (i.e.
the value is not provided), the current value of the object will be used.

POST /activities/:slug
~~~~~~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

    {
      "slug":"testing"
    }

Response body:

.. code-block:: javascript

    {
      "name":"Testing Infra",
      "slug":"testing",
      "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
      "created_at": "2014-04-16",
      "updated_at": "2014-04-17",
      "deleted_at": null,
      "revision":2
    }

POST /times/:uuid
~~~~~~~~~~~~~~~~~

Original object:


.. code-block:: javascript

    {
      "duration":12,
      "user": "example-2",
      "activities": ["qa"],
      "project": ["gwm", "ganeti"],
      "notes":"",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/56",
      "date_worked":"2015-07-29",
      "created_at":"2014-06-12",
      "updated_at":null,
      "deleted_at":null,
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "revision":1
    }

Request body:

.. code-block:: javascript

    {
      "duration":18,
      "notes":"Initial duration was inaccurate. Date worked also updated.",
      "date_worked":"2015-08-07"
    }

The response body will be:

.. code-block:: javascript

    {
      "duration":18,
      "user": "example-2",
      "activities": ["qa"],
      "project": ["gwm", "ganeti"],
      "notes":"Initial duration was inaccurate. Date worked also updated.",
      "issue_uri":"https://github.com/osuosl/ganeti_webmgr/issues/56",
      "date_worked":"2015-08-07",
      "created_at":"2014-06-12",
      "updated_at":"2015-10-18",
      "deleted_at":null,
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "revision":2
    }

If a slugs field is passed to ``/project/:slug``, it is assumed to overwrite
the existing slugs for the object. Any slugs which already exist on the object
but are not in the request are dropped, and the slugs field on the request
becomes canonical, assuming all of the slugs do not already belong to another
project.

In the case of a foreign key (such as project on a time) that does not point to
a valid object or a malformed object sent in the request, an Object Not Found
or Malformed Object error (respectively) will be returned, validation will
return immediately, and the object will not be saved.

The following content is checked by the API for validity:

* Time/Date must be a valid ISO 8601 Date/Time.
* URI must be a valid URI.
* Activities must exist in the database.
* The Project must exist in the database.
* Project slugs must not already belong to another project.

----------------

DELETE Endpoints
----------------

The single object endpoints (e.g. ``/times/:uuid``, ``/projects/:slug``) support
DELETE requests; these remove an object from the records.

If the object is successfully deleted, an empty response body is sent, with a 200 OK
status. If the deletion fails for any reason, an error object is returned.

These objects must always be soft-deleted; that is, the object will still exist within the
database. Nonetheless, requests for lists of objects (e.g. ``GET /projects``) will exclude
the object from the results, and requests for single objects (e.g.
``GET /times/:uuid``) will return a 404. The parameter ``?include_deleted``
circumvents this requirement and allows deleted objects to be returned as well.

An object's deleted status is indicated by setting its ``deleted_at`` to the time of
deletion; if the value is null, the object is not deleted. Only the most recent revision
is set. In addition, activities and projects have their ``slugs`` removed, in order
to allow these slugs to be reused by future objects.

Unfortunately, this means that it is impossible to request or update a project or activity
after it is deleted, even using the ``?include_deleted`` parameter. Instead, a new project
or activity must be made; because the original slugs were deleted, the new object can
share any or all of the original project's values.

When attempting to delete a project or activity, it must not be referenced by a current
time (i.e. one which is neither deleted nor updated). If it is referenced by a current
time, a Request Failure error is returned.

-----------------------

Authorization and Roles
-----------------------

Each TimeSync user can be of one of two roles: user, and admin. Admins have
special permissions, including adding, updating, and deleting activities and
projects, creating and promoting users, as well as acting as automatic
managers/spectators of all projects.

In addition, each user has a role within each project to which they belong:

* member
* spectator
* project manager

These roles exist independently (for example, a user may be only a spectator,
or may be a member and manager but not spectator), and are defined by their
permissions:

* a member has permission to write time entries
* a spectator may view time entries
* a project manager may update the project information

A user may be a member, spectator, and/or manager of multiple projects, and a project
may have multiple members, spectators, and managers.

If a user attempts to access an endpoint which they are not authorized for, the
server will return an Authorization Failure.

GET Endpoints
~~~~~~~~~~~~~

GET endpoints do not have authorization at this time, and so any user can
request data from a GET endpoint.

POST and DELETE Endpoints
~~~~~~~~~~~~~~~~~~~~~~~~~

POST /activities, POST /activities/:slug, and DELETE /activities/:slug are all
only accessible to admin users.

POST /projects and DELETE /projects/:slug are only accessible to admin users.
POST /projects/:slug is accessible to that project's manager(s).

POST /times is accessible to that project's member(s), given that the 'user'
field of the posted time is the user authenticating.
