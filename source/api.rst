.. _api:

=================
API Specification
=================

The current release of the API is v0.1.0.

:ref:`Click here if you would like to skip the pre-amble and go straight to the
endpoints examples<endpoints>`

.. contents::

.. note::

  Variables are indicated with the ``:variable_name`` syntax
  ([colon][variable name]). If you see something like ``:time`` or ``:slug``
  being referenced it is not the literal string ':slug' and ':time' but a
  variable.

----------

Connection
----------

All requests are made via HTTPS. Available methods include:
  * GET to request an object
  * POST to create and/or edit a new object
  * DELETE to remove an object.

------

Format
------

* Responses are returned in standard JSON.
* Multiple results are sent as a list of JSON objects.

  * Order of results is not guaranteed.

* Single results are returned as a single JSON object.


.. note::

  Throughout this API, any form of dates will use a simplified ISO-8601
  format, as `defined by ECMA International.
  <http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15>`_
  This format appears as "YYYY-MM-DD", where Y is the year, M is the month (01-12), and
  DD is the day (01-31).

--------

Versions
--------

The API is versioned with the letter 'v' followed by increasing integers.

For example: ``https://timesync.osuosl.org/v0/projects``

Versions are updated each time there is a significant change to the public API
(not an implementation). A 'significant change' include updates which force a
client to change how it interacts with the API (backward-compatibility breaks,
endpoint renaming, etc).

-----

Slugs
-----

Slugs are used to get objects from the back-end, reference objects from within
other objects, etc. A valid slug follows a very specific format:

#) May only contain numbers and lowercase letters
#) Sets of lowercase letters and numbers can be separated with a single hyphen
#) Must contain at least one letter

For instance:

========== ===============
Not a Slug A Slug
---------- ---------------
--2cool--  e
!ir0ck~    my-username
@username  bossperson
========== ===============

---------

Revisions
---------

When an object is first created, it is assigned a unique tracking ID (UUID).
This UUID will refer to all versions of the same object. For example:

.. code-block:: none

  de305d54-75b4-431b-adb2-eb6b9e546014

When an object is updated, a new revision is created. This allows one to easily
keep track of changes to an object over time (the object's *audit trail*). An
implementation specific backend database key, like an auto-assigned ID (`1`,
`9`, `2001`), would only be used to point to a revision of a given object.

A specific revision of an object can be referred to by its unique compound key
(UUID, revision) where revision is a number which refers to the position of
that version of the object in the audit trail (where 1 is the original version
from object creation, 2 is created after the first update, etc.). This revision
number is re-used between objects.

----------------

Auditing History
----------------

There are three variables in all objects that assist in an audit process
(viewing revisions of an object through its history).

* ``created_at``: the date at which a given object (specified by a UUID) was
  created.
* ``updated_at``: The date at which an object was modified (the day this revision of the
  object was created).
* ``deleted_at``: When the DELETE operation is performed on an object its
  ``deleted_at`` field is set to the date it was deleted. Historical
  (``parents``) copies of an object do not have ``deleted_at`` set unless the
  object was deleted for a given historical copy (and later un-deleted).


**To view the audit trail of an object pass the** ``?include_revisions=true``
**parameter to an endpoint and inspect the** ``parents`` **variable (a list of
object revisions).**

.. note::

    The ``include_revisions`` parameter does not work on all endpoints.

    Check out the :ref:`GET Parameters<query_parameters>` for more
    details.

----------

Pagination
----------

All multiple-item GET endpoints support a pair of query parameters to be used in
pagination: ``?skip`` and ``?limit``.

GET results will always be returned in chronological order by updated_at value
or by created_at where updated_at is NULL.

?limit=n causes the endpoint to return at most *n* objects from its query.
If *n = 0*, the full list of items is returned. The default is 25.

?skip=s causes the first *s* items to be dropped from the result list; they are
not counted in the limit.

To count by an integer, zero-indexed page count *p*, you can request
``?limit=n&skip=(p*n)``; for a one-indexed page count, ``skip=((p+1)*n)``.

-------------

.. _endpoints:

GET Endpoints
-------------

GET /projects
~~~~~~~~~~~~~

.. code-block:: javascript

  [
    {
      "uri": "https://code.osuosl.org/projects/ganeti-webmgr",
      "name": "Ganeti Web Manager",
      "slugs": ["gwm", "ganeti"],
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": "2014-04-19",
      "revision": 2,
      "users": {
        "user1": {
          "member": true,
          "spectator": false,
          "manager": false
        },
        "user2": {
          "member": true,
          "spectator": true,
          "manager": true
        },
        // ...
      }
    },
    {
      // ...
    }
  ]

GET /projects/:slug
~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

  {
    "uri": "https://code.osuosl.org/projects/ganeti-webmgr",
    "name": "Ganeti Web Manager",
    "slugs": ["ganeti", "gwm"],
    "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
    "revision": 4,
    "created_at": "2014-07-17",
    "deleted_at": null,
    "updated_at": "2014-07-20",
    "users": {
      "user1": {
        "member": true,
        "spectator": false,
        "manager": false
      },
      "user2": {
        "member": true,
        "spectator": true,
        "manager": true
      },
      // ...
    }
  }

GET /activities
~~~~~~~~~~~~~~~

.. code-block:: javascript

  [
    {
      "name": "Documentation",
      "slugs": ["docs", "doc"],
      "uuid": "adf036f5-3d49-4a84-bef9-062b46380bbf",
      "revision": 1,
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": null
    },
    {
      // ...
    }
  ]

GET /activities/:slug
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

  {
    "name": "Documentation",
    "slugs": ["doc", "docs"],
    "uuid": "adf036f5-3d49-4a84-bef9-062b46380bbf",
    "revision": 5,
    "created_at": "2014-04-17",
    "deleted_at": null,
    "updated_at": "2014-04-26"
  }

GET /times
~~~~~~~~~~

.. code-block:: javascript

  [
    {
      "duration": 12000,
      "user": "example-user",
      "project": ["ganeti", "gwm"],
      "activities": ["docs", "planning"],
      "notes": "Worked on documentation toward settings configuration.",
      "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked": "2014-04-17",
      "revision": 1,
      "created_at": "2014-04-17",
      "updated_at": null,
      "deleted_at": null,
      "uuid": "c3706e79-1c9a-4765-8d7f-89b4544cad56"
    },
    {
      //...
    }
  ]

.. caution::

  Be aware that this endpoint will return different values depending on the permissions
  of the caller. For more information, see `Authorization and Permissions`_, below.

GET /times/:time-entry-uuid
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

  {
    "duration": 12000,
    "user": "example-user",
    "project": ["gwm", "ganeti"],
    "activities": ["doc", "research"],
    "notes": "Worked on documentation toward settings configuration.",
    "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
    "date_worked": "2014-04-17",
    "created_at": "2014-04-17",
    "updated_at": "2014-04-21",
    "deleted_at": null,
    "uuid": "c3706e79-1c9a-4765-8d7f-89b4544cad56",
    "revision": 3
  }

----------------------------

.. _query_parameters:

GET Request Query Parameters
----------------------------

TimeSync's response data can be narrowed even further than the /:endpoints
return statements by adding parameters:

* user
* project
* activity
* date range
* object revisions
* deleted objects

Reference Table
~~~~~~~~~~~~~~~

=================== ======================= =======================
Parameter           Value(s)                Endpoint(s)
=================== ======================= =======================
?user=              :username               - /times
                                            - /projects
?project=           :project-slug           /times
?activity=          :activity-slug          /times
?start=             :date (ISO format)      /times
?end=               :date (ISO format)      /times
?include_revisions= :bool                   - /activities/
                                            - /activities/:slug
                                            - /projects/
                                            - /projects/:slug
                                            - /times
                                            - /times/:uuid
?include_deleted=   :bool                   - /activities
                                            - /projects
                                            - /times
                                            - /times/:uuid
                                            - /users
                                            - /users/:username
=================== ======================= =======================

.. note::

   A query parameter may only be used once in a given query. Duplicate instance
   of the same query parameter will be discarded.

?user=:username
~~~~~~~~~~~~~~~

``/times?user=:username``
  Filters results to a set of time submitted entries by a specified user.

``/projects?user=:username``
  Filters results to a set of projects on which a specified user is a member.

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
  Filters results to a set of time entries on or after a specified date.

``/times?end=:date&start=:date``
  Can be combined with ?end to create a date range.

?end=:date
~~~~~~~~~~

``/times?end=:date``
  Filters results to a set of time entries on or before a specified date.

``/times?start=:date&end=:date``
  Can be combined with ?start to create a date range.

?include_revisions=:bool
~~~~~~~~~~~~~~~~~~~~~~~~

The 'parents' field is added to the specified object when this parameter is
included and not set to ``false``.

This field is a list of all previous revisions of the object in descending
order by revision number (i.e. ``time.parents[0]`` will be the previous
revision, and ``time.parents[n-1]`` will be the first revision).

?include_deleted=:bool
~~~~~~~~~~~~~~~~~~~~~~

Deleted entries are included in the returned results when this parameter is
included and not set to ``false``.

These are objects which have the 'deleted_at' parameter set to an ISO date
(i.e., a non-null value).

Multiple Parameters Per Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When multiple parameters are used, they narrow down the result set

.. code-block:: none

  $ GET /times?user=example-user&activity=dev&token=...
  # This will return all time entries which were entered by example-user AND
  # which were spent doing development.

Date ranges are inclusive on both ends.

Malformed or Exceptional Parameter Usage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a query parameter is provided with a bad value (e.g. invalid slug, or date
not in ISO-8601 format), a Bad Query Value error is returned.

Any query parameter other than those specified in this document will be
ignored.

For more information about errors, check the :ref:`errors<errors>`
docs.

If multiple ``start``, ``end``, ``include_deleted``, or ``include_revisions`` parameters
are provided, the first one sent is used. If a query parameter is not provided, it
defaults to 'all values'.

Including Revisions of Objects (include_revisions)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

GET /projects/:slug?include_revisions=true
++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  {
    "uri": "https://code.osuosl.org/projects/ganeti-webmgr",
    "name": "Ganeti Web Manager",
    "slugs": ["ganeti", "gwm"],
    "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
    "revision": 4,
    "created_at": "2015-04-16",
    "deleted_at": null,
    "updated_at": "2015-04-23",
    "parents": [
      {
      "uri": "https://code.osuosl.org/projects/old-ganeti-webmgr",
      "name": "Old Ganeti Web Manager",
      "slugs": ["ganeti", "gwm"],
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 3,
      "created_at": "2015-04-16",
      "deleted_at": null,
      "updated_at": "2015-04-21",
      },
      {
        // ...
      },
      // ...
    ],
    "users": {
      "user1": {
        "member": true,
        "spectator": false,
        "manager": false
      },
      "user2": {
        "member": true,
        "spectator": true,
        "manager": true
      },
      // ...
    }
  }

.. note::

  Member lists are not stored for old revisions, so when requesting projects with
  ?include_revisions, the parents will not have "users" fields.

GET /projects?include_revisions=true
++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  [
    {
      "uri": "https://code.osuosl.org/projects/ganeti-webmgr",
      "name": "Ganeti Web Manager",
      "slugs": ["ganeti", "gwm"],
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 4,
      "created_at": "2015-04-16",
      "deleted_at": null,
      "updated_at": "2015-04-23",
      "parents": [
        {
        "uri": "https://code.osuosl.org/projects/old-ganeti-webmgr",
        "name": "Old Ganeti Web Manager",
        "slugs": ["ganeti", "gwm"],
        "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
        "revision": 3,
        "created_at": "2015-04-16",
        "deleted_at": null,
        "updated_at": "2015-04-21",
        },
        {
          // ...
        },
        // ...
      ],
      "users": {
        "user1": {
          "member": true,
          "spectator": false,
          "manager": false
        },
        "user2": {
          "member": true,
          "spectator": true,
          "manager": true
        },
        // ...
      }
    },
    {
      // ...
    },
    // ...
  ]

GET /times/:uuid?include_revisions=true
+++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  {
    "duration": 2000,
    "user": "example-user",
    "project": ["ganeti", "gwm"],
    "activities": ["doc", "research"],
    "notes": "Worked on documentation toward settings configuration.",
    "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
    "date_worked": "2015-04-12",
    "created_at": "2015-04-12",
    "updated_at": "2015-04-18",
    "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
    "deleted_at": null,
    "revision": 2,
    "parents": [
      {
        "duration": 20,
        "user": "example-user",
        "project": ["ganeti", "gwm"],
        "activities": ["doc", "research"],
        "notes": "Worked on documentation toward settings configuration.",
        "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
        "date_worked": "2015-04-12",
        "created_at": "2015-04-12",
        "updated_at": null,
        "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
        "deleted_at": null,
        "revision": 1
      }
    ]
  }

GET /times?include_revisions=true
+++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  [
    {
      "duration": 2000,
      "user": "example-user",
      "project": ["ganeti", "gwm"],
      "activities": ["doc", "research"],
      "notes": "Worked on documentation toward settings configuration.",
      "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked": "2015-04-12",
      "created_at": "2015-04-12",
      "updated_at": "2015-04-18",
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "deleted_at": null,
      "revision": 2,
      "parents": [
        {
          "duration": 20,
          "user": "example-user",
          "project": ["ganeti", "gwm"],
          "activities": ["doc", "research"],
          "notes": "Worked on documentation toward settings configuration.",
          "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
          "date_worked": "2015-04-12",
          "created_at": "2015-04-12",
          "updated_at": null,
          "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
          "deleted_at": null,
          "revision": 1
        }
      ]
    },
    {
      "duration": 12000,
      "user": "example-user",
      "project": ["timesync", "ts"],
      "activities": ["doc"],
      "notes": "Improved readability of API documentation.",
      "issue_uri": "https://github.com/osuosl/timesync/issues/66",
      "date_worked": "2016-03-23",
      "created_at": "2016-03-23",
      "updated_at": "2016-03-25",
      "uuid": "941a39b1-2507-48a6-8530-a83419661300",
      "deleted_at": null,
      "revision": 1
    }
  ]

GET /activities/:slug?include_revisions=true
++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  {
    "name": "Testing Infra",
    "slug": "test",
    "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
    "created_at": "2014-04-17",
    "deleted_at": null,
    "updated_at": "2014-04-18",
    "revision": 2,
    "parents": [
      {
        "name": "Testing Infrastructure",
        "created_at": "2014-04-17",
        "deleted_at": null,
        "updated_at": null,
        "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
        "revision": 1
      }
    ]
  }

GET /activities?include_revisions=true
++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  [
    {
      "name": "Testing Infra",
      "slug": "test",
      "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": "2014-04-18",
      "revision": 2,
      "parents": [
        {
          "name": "Testing Infrastructure",
          "slug": "test",
          "created_at": "2014-04-17",
          "deleted_at": null,
          "updated_at": null,
          "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
          "revision": 1
        }
      ]
    },
    {
      "name": "Build Infra",
      "slug": "build",
      "uuid": "e81e45ef-e7a7-4da2-88cd-9ede610c5896",
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": "2014-04-23",
      "revision": 2,
      "parents": [
        {
          "name": "Testing Infrastructure",
          "slug": "build",
          "created_at": "2014-04-17",
          "deleted_at": null,
          "updated_at": null,
          "uuid": "e81e45ef-e7a7-4da2-88cd-9ede610c5896",
          "revision": 1
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
      "uri": "https://code.osuosl.org/projects/ganeti-webmgr",
      "name": "Ganeti Web Manager",
      "slugs": ["ganeti", "gwm"],
      "uuid": "a034806c-00db-4fe1-8de8-514575f31bfb",
      "revision": 4,
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": null
    },
    {
      // ...
    },
    {
      // ...
    },
    {
      "uri": "https:://github.com/osuosl/timesync",
      "name": "Timesync",
      "slugs": ["ganeti", "gwm"],
      "uuid": "1f8788bd-0909-4397-be2c-79047f90c575",
      "revision": 1,
      "created_at": "2014-04-17",
      "deleted_at": "2015-10-01",
      "updated_at": null
    }
  ]

.. note::

    Note that this now includes the Timesync project, which had previously been deleted.

GET /activities?include_deleted=true
++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  [
    {
      "name": "Documentation",
      "slug": "doc",
      "uuid": "adf036f5-3d49-4a84-bef9-062b46380bbf",
      "revision": 5,
      "created_at": "2014-04-17",
      "deleted_at": null,
      "updated_at": "2014-05-23"
    },
    {
      // ...
    },
    {
      // ...
    },
    {
      "name": "Meetings"
      "slug": "doc",
      "uuid": "6552d14e-12eb-4f1f-83d5-147f8452614c",
      "revision": 1,
      "created_at": "2014-04-17",
      "deleted_at": "2015-05-01",
      "updated_at": null
    }
  ]

.. note::

  Note that this now includes the Meetings activity, which had previously been deleted.

GET /times?include_deleted=true
+++++++++++++++++++++++++++++++

.. code-block:: javascript

  [
    {
      "duration": 2000,
      "user": "example-user",
      "project": ["ganeti", "gwm"],
      "activities": ["doc", "research"],
      "notes": "Worked on documentation toward settings configuration.",
      "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/40",
      "date_worked": "2015-04-12",
      "created_at": "2015-04-12",
      "updated_at": "2015-04-18",
      "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
      "deleted_at": null,
      "revision": 2
    },
    {
      "duration": 3000,
      "user": "example-user",
      "project": ["timesync", "ts"],
      "activities": ["doc"],
      "notes": "Worked on documentation toward include_deleted parameter.",
      "issue_uri": "https://github.com/osuosl/timesync/issues/52",
      "date_worked": "2015-08-18",
      "created_at": "2015-08-18",
      "updated_at": "2015-09-10",
      "deleted_at": "2015-10-12",
      "uuid": "e283a2cd-39c6-4133-95ec-5bc10dd9a9ef",
      "revision": 2
    }
  ]

.. note::

  Note that this now includes the second time, which had previously been deleted.

GET /times/:uuid?include_deleted=true
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

  {
    "duration": 30,
    "user": "example-user",
    "project": ["timesync", "ts"],
    "activities": ["doc"],
    "notes": "Worked on documentation toward include_deleted parameter.",
    "issue_uri": "https://github.com/osuosl/timesync/issues/52",
    "date_worked": "2015-08-18",
    "created_at": "2015-08-18",
    "updated_at": "2015-09-10",
    "deleted_at": "2015-10-12",
    "uuid": "e283a2cd-39c6-4133-95ec-5bc10dd9a9ef",
    "revision": 2
  }

.. note::

    As above, this time is deleted (note the deleted_at field), but instead of
    a 404, it returns the object.

--------------

POST Endpoints
--------------

To add a new object, POST to */:object-name/* with a JSON body. The response
body will contain the object in a similar manner as the GET endpoints above.

POST /projects/
~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

  {
    "uri": "https://code.osuosl.org/projects/timesync",
    "name": "TimeSync API",
    "slugs": ["timesync", "time"],
    "users": {
      "user1": {
        "member": true,
        "spectator": false,
        "manager": false
      },
      "user2": {
        "member": true,
        "spectator": true,
        "manager": true
      },
      // ...
    }
  }

Response body:

.. code-block:: javascript

  {
    "uri": "https://code.osuosl.org/projects/timesync",
    "name": "TimeSync API",
    "slugs": ["timesync", "time"],
    "uuid": "b35f9531-517f-47bd-aab4-14298bb19555",
    "created_at": "2014-04-17",
    "updated_at": null,
    "deleted_at": null,
    "revision": 1,
    "users": {
      "user1": {
        "member": true,
        "spectator": false,
        "manager": false
      },
      "user2": {
        "member": true,
        "spectator": true,
        "manager": true
      },
      // ...
    }
  }

.. note::

  Because of sitewide manager and admin permissions, no users are automatically added to
  a project, unless a ``users`` field is passed to add them.

POST /activities/
~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

  {
   "name": "Quality Assurance/Testing",
   "slug": "qa"
  }

Response body:

.. code-block:: javascript

  {
    "name": "Quality Assurance/Testing",
    "slug": "qa",
    "uuid": "cfa07a4f-d446-4078-8d73-2f77560c35c0",
    "created_at": "2014-04-17",
    "updated_at": null,
    "deleted_at": null,
    "revision": 1
  }


POST /times/
~~~~~~~~~~~~

Request body:

.. code-block:: javascript

  {
    "duration": 12,
    "user": "example-2",
    "project": "ganeti_web_manager",
    "activities": ["docs"],
    "notes": "Worked on documentation toward settings configuration.",
    "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/56",
    "date_worked": "2014-04-17"
  }

Response body:

.. code-block:: javascript

  {
    "duration": 12,
    "user": "example-2",
    "project": "ganeti_web_manager",
    "activities": ["docs"],
    "notes": "Worked on documentation toward settings configuration.",
    "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/56",
    "date_worked": "2014-04-17",
    "created_at": "2014-04-17",
    "updated_at": null,
    "deleted_at": null,
    "uuid": "838853e3-3635-4076-a26f-7efe4e60981f",
    "revision": 1
  }


POST /users/
~~~~~~~~~~~~

User documentation can be found in the :ref:`User Documentation<users>`

~~~~

Likewise, if you'd like to edit an existing object, POST to
``/projects/:slug``, ``/activities/:slug``, or ``/times/:uuid`` with a JSON
body.  The object only needs to contain the part that is being updated. The
response body will contain the saved object, as shown above.

.. note::

  If a deleted time or user is updated using these endpoints, the new revision is no
  longer deleted; the old revision still has its deleted_at set, but the new revision
  does not, allowing it to appear in GET responses, etc. Note that this does not apply
  to activities or projects; because their slugs are deleted, they cannot be referenced
  by these endpoints, and thus must be recreated.

POST /projects/:slug
~~~~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

  {
    "uri": "https://code.osuosl.org/projects/timesync",
    "name": "TimeSync API",
    "slugs": ["timesync", "ts"]
  }

Response body:

.. code-block:: javascript

  {
    "uri": "https://code.osuosl.org/projects/timesync",
    "name": "TimeSync API",
    "slugs": ["timesync", "ts"],
    "created_at": "2014-04-16",
    "updated_at": "2014-04-18",
    "deleted_at": null,
    "uuid": "309eae69-21dc-4538-9fdc-e6892a9c4dd4",
    "revision": 2,
    "users": {
      "user1": {
        "member": true,
        "spectator": false,
        "manager": false
      },
      "user2": {
        "member": true,
        "spectator": true,
        "manager": true
      },
      // ...
    }
  }

.. note::

  If a slugs field is passed to ``/projects/:slug``, it is assumed to overwrite
  the existing slugs for the object. Any slugs which already exist on the object
  but are not in the request are dropped, and the slugs field on the request
  becomes canonical.

  If any of the slugs provided belong to any other projects, a
  :ref:`Slug Already Exists<slug-already-exists>` error is returned
  listing all slugs already associated with other projects, and no changes are made.


POST /activities/:slug
~~~~~~~~~~~~~~~~~~~~~~

Request body:

.. code-block:: javascript

  {
    "slug": "testing"
  }

Response body:

.. code-block:: javascript

  {
    "name": "Testing Infra",
    "slug": "testing",
    "uuid": "3cf78d25-411c-4d1f-80c8-a09e5e12cae3",
    "created_at": "2014-04-16",
    "updated_at": "2014-04-17",
    "deleted_at": null,
    "revision": 2
  }

POST /times/:uuid
~~~~~~~~~~~~~~~~~

Original object:

.. code-block:: javascript

  {
    "duration": 12000,
    "user": "example-2",
    "activities": ["qa"],
    "project": ["gwm", "ganeti"],
    "notes": "",
    "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/56",
    "date_worked": "2014-06-10",
    "created_at": "2014-06-12",
    "updated_at": null,
    "deleted_at": null,
    "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
    "revision": 1
  }

Request body:

.. code-block:: javascript

  {
    "duration": 18000,
    "notes": "Initial duration was inaccurate. Date worked also updated.",
    "date_worked": "2014-06-07"
  }

The response body will be:

.. code-block:: javascript

  {
    "duration": 18000,
    "user": "example-2",
    "activities": ["qa"],
    "project": ["gwm", "ganeti"],
    "notes": "Initial duration was inaccurate. Date worked also updated.",
    "issue_uri": "https://github.com/osuosl/ganeti_webmgr/issues/56",
    "date_worked": "2014-06-07",
    "created_at": "2014-06-12",
    "updated_at": "2014-07-02",
    "deleted_at": null,
    "uuid": "aa800862-e852-4a40-8882-9b4a79aa3015",
    "revision": 2
  }

----

POST /users/:username
~~~~~~~~~~~~~~~~~~~~~

User documentation can be found in the :ref:`User Documentation<users>`

----

.. note::

    If a value of ``""`` (an empty string) or ``[]`` (an empty array) are
    passed as values for a string or array optional field (see the
    :ref:`model docs<model>`), the value will be set to the empty string/array.
    If a value of undefined is provided (i.e.  the value is not provided), the
    current value of the object will be used.

.. note::

    In the case of a malformed object sent in the request, or a foreign key
    (such as project on a time) that does not point to a valid object, a
    Malformed Object, Object Not Found or error (respectively) will be
    returned, validation will return immediately, and the object will not be
    saved.

----

The following content is checked by the API for validity:

* Time/Date must be a valid ISO 8601 Date/Time.
* URI must be a valid URI.
* Activities must exist in the database.
* The Project must exist in the database.
* Project and activity slugs must not already belong to another
  project/activity.

----------------

DELETE Endpoints
----------------

The single object endpoints (e.g. ``/times/:uuid``, ``/projects/:slug``)
support DELETE requests; these remove an object from the records.

If the object is successfully deleted, an empty response body is sent, with a
200 OK status. If the deletion fails for any reason, an error object is
returned.

These objects will always be soft-deleted; that is, the object will still
exist, but will not be returned for a normal GET request. Requests for lists of
objects (e.g. ``GET /projects``) will exclude the object from the results, and
requests for single objects (e.g.  ``GET /times/:uuid``) will return a 404. The
parameter ``?include_deleted`` circumvents this requirement and allows deleted
objects to be included in the returned set of objects.

An object's deleted status is indicated by setting its ``deleted_at`` field to
the date of deletion; if the value is null, the object is not deleted. Only
the most recent revision is set. In addition, activities and projects have
their ``slugs`` removed in order to allow these slugs to be reused by future
objects.

This means that it is impossible to request or update a project or activity
after it is deleted, even when using the ``?include_deleted`` parameter.
Instead, a new project or activity must be made; because the original slugs
were deleted, the new object can share any or all of the original project's
user-defined metadata.

When deleting a project or activity it must not be referenced by a current time
entry (i.e. one which is neither deleted nor updated). If it is referenced by a
current time, then a Request Failure error is returned.

-----------------------------

Authorization and Permissions
-----------------------------

There are two classes of permissions in TimeSync: project roles and site roles.
Each user may be any combination of the following:

* site_spectator
* site_manager
* site_admin.

In addition, each user may be any combination of the following:

* project_member
* project_spectator
* project_manager

for an individual project.

These project permissions exist independently. A user may only be a
site_spectator, or may be a project_member and project_manager but not
project_spectator; sitewide permissions override those of projects.
Permissions are defined here:

==================  =================================================================
    Permission                                  Allowed to
==================  =================================================================
Project member      Create time entries
Project spectator   View time entries for that project (see ``GET Endpoints``, below)
Project manager     Update projects and members
------------------  -----------------------------------------------------------------
Sitewide spectator  View all time entries
Sitewide manager    Create projects and activities, create users
Sitewide admin      Any action, including promote users to managers and admins
==================  =================================================================

A user may be a member, spectator, and/or manager of multiple projects, and a
project may have multiple members, spectators, and managers.

If a user attempts to access an endpoint which they are not authorized for, the
server will return an Authorization Failure.

.. note::

    It is recommended that the site have one admin user which belongs to no one
    in particular, similarly to the Linux ``root`` user, which may add other
    users/admins.

GET Endpoints
~~~~~~~~~~~~~

GET /activities, GET /activities/:slug, GET /projects, and GET /projects/:slug
are accessible to anyone who has successfully authenticated.

GET /times will return:

* The authenticated user's times
* All times in projects for which a user is a spectator or manager
* All times if the user is a sitewide spectator or manager

GET /times/:uuid follows the same rules (i.e. it will return the time if that
time would be in the results of /times, or Authentication Failure otherwise).

User documentation can be found in the :ref:`User Documentation<users>`

POST Endpoints
~~~~~~~~~~~~~~

POST /activities and POST /activities/:slug can be accessed by sitewide
managers.

POST /projects is accessible to sitewide managers.

POST /projects/:slug is accessible to the project's manager(s) and sitewide
managers.  In addition, note that both project managers and sitewide managers may promote
another user to manager and demote other managers. As well, note that a project manager
may in this way demote themselves or remove themselves from the project.

POST /times is accessible to members of the project for which they intend to
create a time.

POST /times/:slug is accessible to the user who created the time originally.

User documentation can be found in the :ref:`User Documentation<users>`

DELETE Endpoints
~~~~~~~~~~~~~~~~

DELETE /activities/:slug is accessible to sitewide managers.

DELETE /projects/:slug is accessible to the project's manager(s) and sitewide
managers.

DELETE /times/:uuid is accessible to the user who created the time and sitewide
managers.

User documentation can be found in the :ref:`User Documentation<users>`
