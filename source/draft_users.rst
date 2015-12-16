.. _draft_users:

=====================
Draft User Management
=====================

Users are managed primarily through the API, by admin users.

.. contents::

-----------

Users Model
-----------

For reference of the structure of this table, see the :ref:`draft_model<draft_model>` docs.

===========  ========  =============  ==============================  =============================================
   Name        Type    POST Required           Description                                Notes
===========  ========  =============  ==============================  =============================================
username     string        true       Username the user logs in with  Used for display if displayname is null
password     string        true       Password for user login         Stored as bcrypt hash; not returned on get
displayname  string        false      Name used for public display
active       boolean       false      Whether user is can log in      Used by admins to invalidate users
admin        boolean       false      Whether user has admin rights   Can only be set by another admin
email        string        false      Email address of user
created_at   ISO date      false      The date the user was created   Automatically set by the server, unchangeable
deleted_at   ISO date      false      The date the user was deleted   Automatically set by the server upon delete
===========  ========  =============  ==============================  =============================================

It is worth noting that, because users are updated "in-place" (i.e. without an audit trail
or revision system), there are no ``uuid``, ``revision``, or ``updated_at`` fields.

-----------

Admin Users
-----------

An admin user is defined by the boolean ``admin`` field. Admin users are able to access
any endpoint, and manage all users (including being the only users who can add users or
promote them to admin).

---------------

/users Endpoint
---------------

User management involves a new endpoint, at ``/users``. In most respects it is similar to
the other endpoints, with certain key differences.

GET /users
~~~~~~~~~~

Returns a list of all users in the database:

.. code-block:: javascript

    [
      {
        "username": "example",
        "displayname": "X. Ample User",
        "email": "example@example.com",
        "active": true,
        "admin": false,
        "created_at": "2015-02-29",
        "deleted_at": null
      },
      {
        ...
      },
      ...
    ]

This endpoint can be accessed by any authenticated user.

GET /users/:username
~~~~~~~~~~~~~~~~~~~~

Returns a specific user from the database.

.. code-block:: javascript

    {
      "username": "example",
      "displayname": "X. Ample User",
      "email": "example@example.com",
      "active": true,
      "admin": false,
      "created_at": "2015-02-29",
      "deleted_at": null
    }

This endpoint can be accessed by any authenticated user, and may be used to look up any
user.

POST /users
~~~~~~~~~~~

Create a new user.

Request:

.. code-block:: javascript

    {
      "username": "example",
      "password": "password",
      "displayname": "X. Ample User",
      "email": "example@example.com"
    }

Response:

.. code-block:: javascript

    {
      "username": "example",
      "displayname": "X. Ample User",
      "email": "example@example.com",
      "active": true,
      "admin": false,
      "created_at": "2015-02-29",
      "deleted_at": null
    }

This endpoint may only be accessed by admins. It is therefore recommended that admins
provide the user with a temporary password and have the user change the password when
they log in.

POST /users/:username
~~~~~~~~~~~~~~~~~~~~~

Update a user's information.

Request:

.. code-block:: javascript

    {
      "username": "example-user",
      "password": "new-password",
      "displayname": "Mr. Example",
      "email": "examplej@example.com"
    }

Response:

.. code-block:: javascript

    {
      "username": "example-user",
      "displayname": "Mr. Example",
      "email": "examplej@example.com",
      "active": true,
      "admin": false,
      "created_at": "2015-02-29",
      "deleted_at": null
    }

This endpoint may be accessed by admins or the user who is being updated. However, the
``admin`` field may only be set by an admin.

DELETE /users/:username
~~~~~~~~~~~~~~~~~~~~~~~

Delete a user. Returns a 200 OK with empty response body on success, or an
:ref:`error<draft_errors>` on failure. Only accessible to admins.

---------------

Role Management
---------------

Role management is handled through the ``projects`` endpoints. The projects model contains
a ``users`` object, which contains three lists: ``members``, ``spectators``, and
``managers``, each lists of usernames. An admin or project manager may set these at any
time, adding to or removing from any of the lists.
