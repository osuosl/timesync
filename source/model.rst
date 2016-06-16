.. _model:

================
API Object Model
================

Below is a listing of the structure of objects in the TimeSync API.

Each object is listed with the fields it consists of. Type represents the type
of data passed; it may represent data types (such as "string" or "array") or
formats (such as "ISO date"). POST Required represents whether "empty" values
are accepted: ``true`` means that values must be complete, while ``false`` means
that values such as an empty string, empty array, or null are valid.

Note that this documentation does not reflect the way the values are stored in
the database, only the way they are passed into and out of the API. For
documentation on storing the objects, see the implementation.

------

Common
------

These fields are shared between all objects except users. Do not pass any of them in a
POST request; they are provided automatically.

==========  ================  =============  ======================================  ===========================================
   Name          Type         POST Required               Description                                    Notes
==========  ================  =============  ======================================  ===========================================
uuid        UUID              false          A tracking ID between object revisions
revision    positive number   false          A version number of an object
created_at  ISO date          false          The date the object (UUID) was created
updated_at  ISO date or null  false          The date this revision was created
deleted_at  ISO date or null  false          The date this revision was deleted      Always null unless ?include_deleted is used
parents     array             false          The previous revisions of this object   Only provided if ?revisions=true is used
==========  ================  =============  ======================================  ===========================================

-----

Times
-----

===========  ===============  =============  ======================================  ===================================================================================
   Name           Type        POST Required               Description                                                        Notes
===========  ===============  =============  ======================================  ===================================================================================
duration     positive number  true           Length of time user worked, in seconds
user         username         true           Username of the user who worked         Must be an existing username
project      mixed            true           The project the user worked on          Array of slugs in GET requests, single slug in POST
activities   array of slugs   sometimes      The activities the user performed       Required on creation if the project has no default activity; not nullable on update
date_worked  ISO date         true           The date this time began on             May be in the future or the past
issue_uri    URI              false          The issue the user worked on            May link to any issue tracker; must be a valid URI
notes        string           false          Other notes the user wishes to provide
===========  ===============  =============  ======================================  ===================================================================================

--------

Projects
--------

================  ==============   =============  ==================================  =============================
      Name             Type        POST Required              Description                         Notes
================  ==============   =============  ==================================  =============================
name              string           true           The name of the project
slugs             array of slugs   true           Slugs to identify the project       Must be unique to the project
uri               URI              false          The URI of the project
default_activity  slug             false          The activity times will default to
================  ==============   =============  ==================================  =============================

----------

Activities
----------

====  ======  =============  =======================================  =====
Name   Type   POST Required               Description                 Notes
====  ======  =============  =======================================  =====
name  string  true           The human-readable name of the activity
slug  string  true           A unique slug to identify the activity
====  ======  =============  =======================================  =====

-----

Users
-----

.. caution::

  User objects do not include the 'Common' fields listed at the top of this
  document.

===============  ======= ===============================  =============================================================
    Field         Type             Description                                        Notes
===============  ======= ===============================  =============================================================
display_name     string  User's public display name.      While username cannot change, display_name can.
username         string  Permanent username.              This username will remain in use even if the user is deleted.
password         string  Password for user login.         Stored as hash; not returned in GET requests.
email            string  Email address of user.
active           bool    Whether the user can login.      Used by admins to invalidate users.
site_spectator   bool    Site-wide spectator flag.        Can be set by a manager or admin.
site_manager     bool    Site-wide manager flag.          Can only be set by an admin.
site_admin       bool    Site-wide admin flag.            Can only be set by another admin.
created_at       date    Date account was created at.     Automatically set, unchangeable.
updated_at       date    Date account was last updated.   Automatically set when updated.
deleted_at       date    Date account was soft-deleted.   Automatically set by server upon DELETE.
meta             string  Miscellaneous user meta-data.
===============  ======= ===============================  =============================================================

.. note::

    Users are updated "in-place" (i.e. without an audit trail or revision
    system), there is no ``uuid`` or ``revision`` field.
