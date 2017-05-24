.. _users:

===============
User Management
===============

Users are managed primarily through the API, by admin users.

.. contents::

------------------

Organization Roles
------------------

The users model provides the concept of organization roles. These are string
values which can be associated with users in order to help define what purpose
the user serves within the organization (e.g. "Intern", "Project Manager",
"Executive"). A user may belong to one or more organization roles, which are
defined by the ``/users/org-roles`` endpoints.

An organization role consists of a machine-readable slug, which shall be passed
to and from the API as references to roles, and a human-readable name, which
shall be returned from the ``GET /users/org-roles/:slug`` endpoint.

Slugs follow the same rules as in the rest of the :ref:`API<api>`.

The list of users may be filtered by roles, in order to get a list only of
members of that role.

Roles should not be confused with the authorization levels (e.g. site_manager),
which configure what data a user is allowed to access or modify within the
TimeSync system. Roles exist largely as metadata for use within the
organization itself.

-----------

Users Model
-----------

For reference of the user fields, see the :ref:`model<model>` docs.

Here is an example user object:

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "org-roles": ["intern"],
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
    period, underscore, and tilde characters.

.. note::

    Usernames are considered case-insensitive: they will be displayed using the
    case they were created with, but both the ``/users/:username`` endpoints
    and the ``/login`` endpoint will match on any capitalization, i.e. if I
    have a user named 'User1', ``GET /users/User1``, ``GET /users/user1``, and
    ``GET /users/uSeR1`` will all return that user.

-----------

Admin Users
-----------

A site-wide admin user is defined by the boolean ``site_admin`` field. Admins
are able to access any endpoint, and manage all users (including being the only
users which can promote others to admin status).

---------------

/users Endpoint
---------------

User management involves a new endpoint, at ``/users``. In most respects it is
similar to the other endpoints, with certain key differences.

GET /users
~~~~~~~~~~

Returns a list of all user objects.

.. code-block:: javascript

  [
    {
      "display_name": "User One",
      "username": "user1",
      "email": "user1@example.org",
      "org-roles": ["intern"],
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "active": true,
      "meta": "extra metadata about user"
    },
    {
      // ...
    },
    // ...
  ]

.. note::

  The /users endpoint also includes the ability to ?include_deleted objects.

.. note::

  Usernames are permanent, even upon deletion.

GET /users?role=:role
~~~~~~~~~~~~~~~~~~~~~

Similar to the base ``GET /users`` endpoint, this endpoint filters only for
users matching the role slug provided. If the query is repeated, it functions as
an "OR" statement.

That is, ``GET /users?role=intern`` returns all interns in the system.

``GET /users?role=intern&role=mentor`` returns everyone who is either an intern
OR a mentor.

.. code-block:: javascript

  [
    {
      "display_name": "User One",
      "username": "user1",
      "email": "user1@example.org",
      "org-roles": ["intern"],
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "active": true,
      "meta": "extra metadata about user"
    },
    {
      // ...
    },
    // ...
  ]

GET /users/:username
~~~~~~~~~~~~~~~~~~~~

Returns a single user object.

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "org-roles": ["intern"],
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
      "org-roles": ["intern"],
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": "2017-06-21",
      "active": false,
      "meta": "extra metadata about user"
    },
    {
      // ...
    },
    // ...
  ]

GET /users/:username?include_deleted=true
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "org-roles": ["intern"],
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
    "org-roles": ["intern"],
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
    "org-roles": ["intern"],
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

.. caution::

  It is the client's responsibility to hash the password before sending it to
  this endpoint, unlike the /login endpoint (see :ref:`auth<auth>`). The
  password should be hashed with bcrypt, using 10 rounds. The bcrypt prefix
  should be "2a" (different implementations may use different prefixes, but the
  API requires consistency for authentication).

.. note::

  This endpoint may only be accessed by admins and sitewide managers.

.. note::

  It is recommended that admins provide the user with a temporary password
  and have the user change the password when they log in.

.. note::

  If an organizational role which does not exist in the system is provided to
  this endpoint, a :ref:`Invalid Foreign Key error<errors>` will be returned.

~~~~~~~~~~~~~~~~~~~~~

POST /users/:username
~~~~~~~~~~~~~~~~~~~~~

Original object:

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "org-roles": ["intern"],
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
    "org-roles": ["developer"],
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
    "org-roles": ["developer"],
    "site_spectator": true,
    "site_manager": true,
    "site_admin": false,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-18",
    "deleted_at": null,
    "meta": "Different metadata about user1"
  }

.. caution::

  It is the client's responsibility to hash the password before sending it to
  this endpoint, unlike the /login endpoint (see :ref:`auth<auth>`). The
  password should be hashed with bcrypt, using 10 rounds. The bcrypt prefix
  should be "2a" (different implementations may use different prefixes, but the
  API requires consistency for authentication).

.. note::

  Site-wide admins can modify other users' site_spectator, site_manager, and site_admin
  fields.

  Site-wide managers can modify other users' site_spectator fields.

.. note::

  If an organizational role which does not exist in the system is provided to
  this endpoint, a :ref:`Invalid Foreign Key error<errors>` will be returned.

.. note::

  The ``org-roles`` field, when passed to this endpoint, overwrites the existing
  value, including if ``[]`` (an empty array) or ``null`` (which is treated like
  an empty array) is passed as the value. To maintain the current role list, the
  existing list must be passed as-is, or else the field must be omitted entirely
  from the request.

This endpoint may be accessed by admins, sitewide managers, or the user who is being
updated. However, users may not set their own permissions unless they are an admin, and
managers may *only* set the ``site_spectator`` field; thus the ``site_admin`` and
``site_manager`` fields may only be set by an admin.

DELETE /users/:username
~~~~~~~~~~~~~~~~~~~~~~~

Soft-delete a user. Returns a 200 OK with empty response body on success, or an
:ref:`error<errors>` on failure. Only accessible to admins.

For more information on deletion, see the DELETE section of the :ref:`API<api>` docs.

---------------

Roles Endpoints
---------------

The following endpoints retrieve or modify the list of roles to which users
may belong.

GET /users/org-roles
~~~~~~~~~~~~~~~~

This endpoint returns the list of roles in the system to which users may belong.

.. code-block:: javascript

  [
    {
      "name": "Summer Intern",
      "slug": "intern"
    },
    {
      "name": "Software Developer",
      "slug": "developer"
    },
    ...
  ]

GET /usrs/org-roles/:slug
~~~~~~~~~~~~~~~~~~~~~~~~~

This endpoint allows one to request the name of an organization role by its
slug.

.. code-block:: javascript

  {
    "name": "Summer Intern",
    "slug": "intern"
  }

POST /users/org-roles
~~~~~~~~~~~~~~~~~

This endpoint creates a new organization role to which users may later be added.

Both role names and slugs must be unique; if they are not, an error will be
returned.

Request body:

.. code-block:: javascript

  {
    "name": "C-Level Executive",
    "slug": "executive"
  }

Response will be identical to the request in case of success.

POST /users/org-roles/:slug
~~~~~~~~~~~~~~~~~~~~~~~~~~~

This endpoint edits the name and/or slug of an existing role.

As with the creation endpoint, both the new role name and slug must not already
exist.

All users who currently have this role will return the new slug after this
request.

Original object:

.. code-block:: javascript

  {
    "name": "Summer Intern",
    "slug": "intern"
  }

Request body:

.. code-block:: javascript

  {
    "slug": "summer"
  }

Response body:

.. code-block:: javascript

  {
    "name": "Summer Intern",
    "slug": "summer"
  }

DELETE /users/org-roles/:slug
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This endpoint deletes a role from the organization, preventing any new users
from being given the role.

Only roles to which no users belong may be deleted. If a role is passed to which
users still belong, a :ref:`Request Failure error<errors>` will be returned.
Edit users with this role to another role to delete the error.

Unlike other objects in TimeSync, roles are permanently deleted by this request.
This means that there is no way to retrieve them after this (and that there is
no ``include_deleted`` query for roles). This also means that the name and slug
previously taken by this role are freed, and a new role with the same name
and/or slug may be created in the future.

------------------------

Authorization Management
------------------------

Authorization management is handled through the ``projects`` and ``users``
endpoints.

The user object contains the ``site_spectator``, ``site_manager``, and
``site_admin`` fields, which are booleans designating those permissions. As
stated above, a sitewide manager may promote a user to sitewide spectator or
demote sitewide spectators; a sitewide admin may also promote a user to
sitewide manager or to admin, or demote sitewide managers or other admins (including
themselves).

The project object contains a ``users`` object, which map users (by username)
to their permissions on the project. An admin, sitewide manager, or project
manager may set these at any time, adding to or removing from any of the lists.
A project may have zero or more of members, spectators, and managers; if a
project has no managers, sitewide managers and admins may still manage the
project.
