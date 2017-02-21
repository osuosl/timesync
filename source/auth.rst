.. _auth:

================================
Authentication and Authorization
================================

TimeSync allows for various forms of authentication determined by the
implementation.

.. contents::

The general scheme of an authentication module looks
like this:

.. code-block:: javascript

  {
    "auth": {
      "type": // type
      // ...
    },
    "object": {
      "name": "Ganeti Web Manager",
      // ...
    }
  }

There are two top-level blocks to an authenticated POST request:

* ``auth``, containing any authentication information necessary
* ``object``, containing the data of the POST request (see the :ref:`API docs<api>`)

An auth block typically looks like the above example, with a ``type`` field
specifying what kind of authentication the client is connecting with, and other
fields as necessary for that kind of connection.

These other fields might be things like ``password``, ``token``, ``key``, etc.

Token-based authentication
--------------------------

Token-based is the primary and default authentication method for TimeSync.
TimeSync provides an endpoint at ``/login``, which should be sent a POST
request, with the body fitting one of the other available authentication
schemes (e.g. Password, LDAP). The endpoint returns a `JWT token
<http://jwt.io/>`_ string as its response body.  This response body is
used to access the other endpoints:

POST endpoints:

.. code-block:: javascript

  {
    "auth": {
      "type": "token",
      "token": // ...
    },
    // ...
  }

GET and DELETE endpoints: Use the query key ``token``, as in ``GET /times?token=:token``
or ``DELETE /activities/example?token=:token``

For example, the workflow may occur as follows:

``POST /login``

.. code-block:: javascript

  {
    "auth": {
      "type": "password",
      "username": "example-user",
      "password": "pass"
    }
  }

Response:

.. code-block:: none

  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ0aW1lc3luYyIsInN1YiI6ImV4Y
    W1wbGUtdXNlciIsImV4cCI6MTQ0NjQ5NzQyNzEzMywiaWF0IjoxNDQ2NDk1NTc5OTY3fQ.k8ij2cXBRs5tUe
    _cq2RDePCYMpFjVkKqnpU11Q1XEnk"
  }

``POST /projects``

.. code-block:: javascript

  {
    "auth": {
      "type": "token",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ0aW1lc3luYyIsInN1YiI6ImV
          4YW1wbGUtdXNlciIsImV4cCI6MTQ0NjQ5NzQyNzEzMywiaWF0IjoxNDQ2NDk1NTc5OTY3fQ.k8ij2c
          XBRs5tUe_cq2RDePCYMpFjVkKqnpU11Q1XEnk"
    },
    "object": {
      "name": "Example Project",
      "owner": "example-user",
      "uri": "http://example.com/",
      "slugs": ["example", "example-project"]
    }
  }

Response:

.. code-block:: javascript

  {
    "name": "Example Project",
    "slugs": ["example", "example-project"],
    "uri": "http://example.com/",
    "owner": "example-user",
    "uuid": "9ac95604-28dd-44e0-9ba5-ff9c5e2b2212",
    "revision": 1,
    "created_at": "2015-11-02",
    "updated_at": null,
    "deleted_at": null
  }

To later get this object back:

``GET /projects/example?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ0aW1lc3luYyI
sInN1YiI6ImV4YW1wbGUtdXNlciIsImV4cCI6MTQ0NjQ5NzQyNzEzMywiaWF0IjoxNDQ2NDk1NTc5OTY3fQ.k8ij2c
XBRs5tUe_cq2RDePCYMpFjVkKqnpU11Q1XEnk``

Response:

.. code-block:: javascript

  {
    "name": "Example Project",
    "slugs": ["example", "example-project"],
    "uri": "http://example.com/",
    "owner": "example-user",
    "uuid": "9ac95604-28dd-44e0-9ba5-ff9c5e2b2212",
    "revision": 1,
    "created_at": "2015-11-02",
    "updated_at": null,
    "deleted_at": null
  }

API tokens have a life of 30 minutes, and must be used on the same TimeSync
instance as they are created.

Password authentication
-----------------------

When used with password-based authentication, TimeSync requires a username
field and a password field:

.. code-block:: javascript

  {
    "auth": {
      "type": "password",
      "username": "tschuy",
      "password": "password"
    }
  }

This username/password combination is compared to values stored in the local
database for authentication.

LDAP Authentication
-------------------

This form is nearly identical to password-based authentication, using a
username and password:

.. code-block:: javascript

  {
    "auth": {
      "type": "ldap",
      "username": "tschuy",
      "password": "password"
    }
  }

Instead of comparing the username/password combination to values in a local
database, however, the combination is provided to a configured LDAP provider for
authentication.
