.. _draft_auth:

=============
Authorization
=============

TimeSync allows for various forms of authentication determined by the
implementation. The general scheme of an authentication module looks
like this:

.. code-block:: javascript

    {
        "auth": {
            "type": // type
            ...
        },
        "object": {
           "name": "Ganeti Web Manager",
           ...
        }
    }

In essence, there are two top-level blocks to an authenticated POST request:

* ``auth``, containing any authentication information necessary
* ``object``, containing the data of the POST request

An auth block typically looks like the above example, with a ``type`` field
specifying what kind of authentication the client is connecting with, an other
fields as necessary for that kind of connection.

These other fields might be things like ``password``, ``token``, ``key``, etc.

Password authentication
-----------------------

When used with password-based authentication, TimeSync requires a username field
and a password field:

.. code-block:: javascript

    {
        "auth": {
            "type": "password",
            "username": "tschuy",
            "password": "password"
        },
        ...
    }

This username/password combination is compared to values stored in the local
database for authentication.

LDAP Authentication
-------------------

This form is nearly identical to password-based authentication, using a username
and password:

.. code-block:: javascript

    {
        "auth": {
            "type": "ldap",
            "username": "tschuy",
            "password": "password"
        },
        ...
    }

Instead of comparing the username/password combination to values in a local
database, however, it provides it to a configured LDAP provider for
authentication.
