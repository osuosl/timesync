.. _draft_model:

================
API Object Model
================

Below is a listing of the structure of objects in the TimeSync API.

Each object is listed with the fields it consists of. Type represents the type
of data passed; it may represent data types (such as "string" or "array") or
formats (such as "ISO date"). POST Required represents whether "empty" values
are accepted: ``true`` means that values must be complete, while ``false`` means
that values such as an empty string or array are valid.

Note that this documentation does not reflect the way the values are stored in
the database, only the way they are passed into and out of the API. For
documentation on storing the objects, check the implementation.

------

Common
------

These fields are shared between all objects. Do not pass any of them in a POST
request; they are provided automatically.

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

===========  ===============  =============  ======================================  ============================================
   Name           Type        POST Required               Description                                    Notes
===========  ===============  =============  ======================================  ============================================
duration     positive number  true           Length of time user worked, in seconds
user         username         true           Username of the user who worked         Must be an existing username
project      mixed            true           The project the user worked on          Array of slugs in GET requests, slug in POST
activities   array of slugs   true           The activities the user performed
date_worked  ISO date         true           The date this time began on             May be in the future or the past
issue_uri    URI              false          The issue the user worked on            May link to any issue tracker
notes        string           false          Other notes the user wishes to provide
===========  ===============  =============  ======================================  ============================================

--------

Projects
--------

=====  ==============   =============  =============================  =============================
Name        Type        POST Required           Description                       Notes
=====  ==============   =============  =============================  =============================
name   string           true           The name of the project
slugs  array of slugs   true           Slugs to identify the project  Must be unique to the project
uri    URI              false          The URI of the project
=====  ==============   =============  =============================  =============================

----------

Activities
----------

====  ======  =============  =======================================  ===========================================
Name   Type   POST Required               Description                                    Notes
====  ======  =============  =======================================  ===========================================
name  string  true           The human-readable name of the activity
slug  string  true           A slug to identify the activity
====  ======  =============  =======================================  ===========================================
