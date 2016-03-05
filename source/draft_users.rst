.. _draft_users:

=====================
Draft User Management
=====================

Users are managed primarily through the API, by admin users.

.. contents::

-----------

Users Model
-----------

For reference of the structure of this table, see the
:ref:`draft_model<draft_model>` docs.

Here is an example user object:

.. code-block:: javascript

      {
        "display_name": "User One",
        "username": "user1",
        "email": "user1@example.org",
        "site_spectator": true,
        "site_manager": false,
        "site_admin": false,
        "created_at": "2016-02-15",
        "updated_at": "2016-02-15",
        "deleted_at": null,
        "active": true,
        "meta": "extra metadata about user"
      }

.. note::

  Usernames may consist of uppercase and lowercase letters, decimal digits, hyphen,
  period, underscore, and tilde characters. Usernames are considered case-insensitive:
  they will be displayed using the case they were created with, but both the
  ``/users/:username`` endpoints and the ``/login`` endpoint will match on any
  capitalization, i.e. if I have a user named 'User1', ``GET /users/User1``, ``GET
  /users/user1``, and ``GET /users/uSeR1`` will all return that user.

-----------

Admin Users
-----------

A site-wide admin user is defined by the boolean ``admin`` field. Admins
are able to access any endpoint, and manage all users (including being the only
users which can promote others to admin status).

---------------

/users Endpoint
---------------

User management involves a new endpoint, at ``/users``. In most respects it is
similar to the other endpoints, with certain key differences.

GET /users
~~~~~~~~~~

Returns a list of all user objects, sorted alphabetically by username.

.. code-block:: javascript

    [
      {
        "display_name": "User One",
        "username": "user1",
        "email": "user1@example.org",
        "site_spectator": true,
        "site_manager": false,
        "site_admin": false,
        "created_at": "2016-02-15",
        "updated_at": "2016-02-15",
        "deleted_at": null,
        "active": true,
        "meta": "extra metadata about user"
      },
      {...},
      ...
    ]

.. note::

    The /users endpoint also includes the ability to ?include_deleted
    objects.

    Usernames are permanent.

GET /users/:username
~~~~~~~~~~~~~~~~~~~~

Returns a single user object.

.. code-block:: javascript

    {
      "display_name": "User One",
      "username": "user1",
      "email": "user1@example.org",
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "active": true,
      "meta": "extra metadata about user"
    }

GET /users?include_deleted=true
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    [
      {
        "display_name": "User One",
        "username": user1,
        "email": "user1@example.org",
        "site_spectator": true,
        "site_manager": false,
        "site_admin": false,
        "created_at": "2016-02-15",
        "updated_at": "2016-02-15",
        "deleted_at": "2017-06-21",
        "active": false,
        "meta": "extra metadata about user"
      },
      {...},
      ...
    ]

GET /users/:username?include_deleted=true
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

    {
      "display_name": "User One",
      "username": "user1",
      "email": "user1@example.org",
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": "2017-06-21",
      "active": false,
      "meta": "extra metadata about user"
    }

POST /users
~~~~~~~~~~~

Create a new user.

Request:

.. code-block:: javascript

    {
      "displayname": "X. Ample User",
      "username": "example",
      "password": "password",
      "email": "example@example.com"
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "active": true,
      "meta": "Some metadata about the user"
    }

Response:

.. code-block:: javascript

    {
      "displayname": "X. Ample User",
      "username": "example",
      "email": "example@example.com"
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "active": true,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "active": true,
      "meta": "Some metadata about the user"
    }

.. note::

    This endpoint may only be accessed by admins and sitewide managers.

    It is recommended that admins provide the user with a temporary password
    and have the user change the password when they log in.

~~~~~~~~~~~~~~~~~~~~~

POST /users/:username
~~~~~~~~~~~~~~~~~~~~~

Original object:

.. code-block:: javascript

    {
      "display_name": "User One",
      "username": "user1",
      "email": "user1@example.org",
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "active": true,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "active": false,
      "meta": "extra metadata about user"
    }

Request body (made by a ``site_admin`` user):

.. code-block:: javascript

    {
      "display_name": "New Displayname",
      "password": "Battery Staple",
      "email": "user1+new@example.org",
      "meta": "Different metadata about user1",
      "site_spectator": true,
      "site_manager": true,
      "site_admin": false,
    }

The response will be:

.. code-block:: javascript

    {
      "display_name": "New Displayname",
      "username": "user1",
      "email": "user1+new@example.org",
      "site_spectator": true,
      "site_manager": true,
      "site_admin": false,
      },
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "meta": "Different metadata about user1"
    }

.. note::

    Site-wide admins can modify other user's manager field.

    Site-wide managers can modify other user's spectator field.

This endpoint may be accessed by admins or the user who is being updated.
However, the ``admin`` field may only be set by an admin.

DELETE /users/:username
~~~~~~~~~~~~~~~~~~~~~~~

Delete a user. Returns a 200 OK with empty response body on success, or an
:ref:`error<draft_errors>` on failure. Only accessible to admins.

---------------

Role Management
---------------

Role management is handled through the ``projects`` and ``users`` endpoints.

The users model contains the ``site_spectator``, ``site_manager``, and ``site_admin``
fields, which are booleans designating those permissions. As stated above, a sitewide
manager may promote a user to sitewide spectator or demote sitewide spectators; a sitewide
admin may also promote a user to sitewide manager or to admin, or demote sitewide managers
or other admins.

The projects model contains a ``users`` object, which map users (by username) to their
permissions on the project. An admin, sitewide manager, or project manager may set these
at any time, adding to or removing from any of the lists. A project may have zero or more
of members, spectators, and managers; if a project has no managers, sitewide managers and
admins may still manage the project.
