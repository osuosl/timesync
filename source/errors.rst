.. _errors:

======
Errors
======

Error types from the TimeSync API.

.. contents::

Errors consist of:

#) a descriptive HTTP status code
#) a standardized error name
#) informational text
#) a ``values`` array containing variables relevant to the error, if any

The existence of an ``error`` key indicates an error.

In the following docs string literals are indicated by double quotes (as in the
JSON standard), but use the ECMAScript 2015 string interpolation specification
to represent variables like slugs and object types.

TimeSync uses the following error codes:

-------------------

1. Object Not Found
-------------------

To be returned if the server receives a valid key (e.g. Time UUID or Activity
Slug) which does not match an object in the database.

.. code-block:: javascript

  {
    "status": 404,
    "error": "Object not found",
    "text": "Nonexistent ${object}"
  }

---------------

2. Server Error
---------------

A generic catch-all for when there is a server error outside of the client's
control. This may be the result of an uncaught exception, a database error, or
any other condition which renders the server unable to process a valid request.
Note that in a production environment, ``text`` may be empty to avoid
disclosing sensitive information.

.. code-block:: javascript

  {
    "status": 500,
    "error": "Server error",
    "text": server_error // (e.g. exception text or sql error)
  }

----------------------

3. Invalid Foreign Key
----------------------

If a client attempts to make a POST request to create or update an object, but
the new object sent by the client contains a foreign key parameter which does
not point to a valid object in the database. (E.g. the client sends a new time
which does not have a valid project.)

.. code-block:: javascript

  {
    "status": 409,
    "error": "Invalid foreign key",
    "text": "The ${object_type} does not contain a valid ${foreign_key} reference"
  }

-------------

4. Bad Object
-------------

A client attempts to make a POST request to create or update an object, but the
new object sent by the client contains a non-existent key, lacks a necessary
key, or contains an invalid value for a key (e.g. a time with a string in the
duration field, or a project without a name.)

.. code-block:: javascript

  unknown_field_error = {
    "status": 400,
    "error": "Bad object",
    "text": "${object_type} does not have a ${field_name} field"
  }
  missing_field_error = {
    "status": 400,
    "error": "Bad object",
    "text": "The ${object_type} is missing a ${field_name}"
  }
  invalid_field_error = {
    "status": 400,
    "error": "Bad object",
    "text": "Field ${field_name} of ${object_type} should be ${expected_type} but was sent as ${received_type}"
  }

---------------------

5. Invalid Identifier
---------------------

This error would be returned when an identifier field (e.g. time UUID or activity
slug) is malformed or otherwise not valid for use; an identifier is considered invalid if
it does not match the expected format (e.g. a slug with special characters or a
non-numeric ID field).

This is to be distinguished from Object Not Found: Object Not Found occurs when a
perfectly valid, well-formed identifier is supplied, but no object matching the identifier
could be found.

Object Not Found is therefore considered to be a temporary error (making an
identical request later may return an object instead of an error), while Invalid
Identifier is considered a permanent error (the request will always return this
error, pending changes to the specification).

.. code-block:: javascript

  {
    "status": 400,
    "error": "Invalid identifier",
    "text": "Expected ${slug/uuid} but received ${received_identifier}",
    "values": [${received_identifier}]
  }

With multiple invalid identifiers, the error is formatted like so:

.. code-block:: javascript

  {
    "status": 400,
    "error": "Invalid identifier",
    "text": "Expected ${slug/uuid} but received: ${bad}, ${bad}, ${bad}",
    "values": [${bad}, ${bad}, ...]
  }

-------------------

6. Invalid Username
-------------------

This error is returned when creating a new user and a username with invalid
characters is provided.

.. code-block:: javascript

  {
    "status": 401,
    "error": "Invalid username",
    "text": "Invalid username ${username} is not a valid username"
  }

7. Authentication Failure
-------------------------

This error is returned when authentication fails for any reason. The text of
the error may change based on what kind of authentication backend the TimeSync
server is running.

.. code-block:: javascript

  {
    "status": 401,
    "error": "Authentication failure",
    "text": "Invalid username or password" / "Bad oAuth token" / etc
  }

.. _slug-already-exists:

----------------------

8. Slug Already Exists
----------------------

This error is returned when a new object is being created but the slugs provided
contain a slug that already exists.

.. code-block:: javascript

  {
    "status": 409,
    "error": "Slug already exists",
    "text": "Slug ${slug} already exists on another object",
    "values": [${slug}]
  }

If multiple slugs are duplicated:

.. code-block:: javascript

  {
    "status": 409,
    "error": "Slugs already exist",
    "text": "Slugs ${slug}, ${slug} already exist on another object",
    "values": [${slug}, ${slug}, ...]
  }

------------------------

9. Authorization Failure
------------------------

This error is returned when the user is successfully authenticated, but lacks
the authorization to complete the task they are attempting to do. This is used
when a non-administrator user attempts to create time or project entries for
another user.

.. code-block:: javascript

  {
    "status": 401,
    "error": "Authorization failure",
    "text": "${user} is not authorized to ${action}"
  }

-------------------

10. Request Failure
-------------------

This error is returned when a request is sent to an
object and is rejected. This is used mainly in the instances when a user tries to
delete something they are not supposed to. For example, a user may attempt to
delete a project that has associated times.

Allowed methods must be returned along with the error object, which will be listed
in the HTTP Allow header.

.. code-block:: javascript

  {
    "status": 405,
    "error": "Method not allowed",
    "text": "The method specified is not allowed for the ${objectType} identified"
  }

-------------------

11. Bad Query Value
-------------------

This error is returned when a GET request is made with query parameters, but the value
of a parameter is invalid in some way. This includes dates which are not sent in
ISO 8601 format, and slugs and IDs which are not considered valid. This error is not
returned, however, if a query parameter is missing (default values are assumed), or if
an extra query parameter is used (nonexistent keys are ignored).

.. code-block:: javascript

  {
  "status": 400,
  "error": "Bad query value",
  "text": "Parameter ${key} contained invalid value ${value}"
  }

---------------------------

12. Username Already Exists
---------------------------

This error is returned when creating a user and the username provided for the new user
is already used by another user. Compare 8. Slug Already Exists.

.. code-block:: javascript

  {
    "status": 409,
    "error": "Username already exists",
    "text": "username ${username} already exists",
    "values": [${username}]
  }
