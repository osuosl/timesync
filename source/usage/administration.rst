.. _administration:

==============
Administration
==============

A TimeSync administrator is a user whose responsibilities include creating
projects, creating activities, and reporting. In general, TimeSync is designed
to be as straightforward as possible; however, there are still a few things to
keep in mind as a TimeSync admin.

Creating new projects and activities
------------------------------------

Projects
````````

A project is something that a TimeSync user works on -- a website, product, etc.
In TimeSync, a project contains a few pieces of information to make user's lives
easier:

* a ``uri``: a uri linking to the project's homepage.
* a ``name``: this is the full name of the project that should be displayed
  to the user.
* an ``owner``: this should be the user responsible for the project, generally
  an admin user. This owner is the only person that can modify the project.
* a list of ``slugs``: these are generally short things that people can refer
  to the project with. If your project is named Protein Geometry Database,
  it makes users' lives easier to only need to type in ``pgd`` when submitting
  new entries.

Slugs can't refer to multiple projects at the same time -- so if you have Fine
Software Project and the Fancy Scalable Project, you can't refer to both of them
as ``fsp``.

Activities
``````````

An activity is something that a TimeSync user spends their time doing --
software development, documentation, code reviewing, etc. An activity contains
only two pieces of information:

* a ``name``: this is a human-readable name of the activity, like Documentation.
* a ``slug``: this is a human-typeable version of the activity. It should be
  memorable and quick to type, like ``dev`` for Development and ``docs`` for
  Documentation.

Activities can only have one slug, so choose wisely.

Getting a subset of time entries
--------------------------------

TimeSync lets you choose a subset of time entries based on things like the user
that did the work, the project worked on, the date range, etc. All parameters
can be done together, to limit a query to only a certain user's time entries on
a certain project in a certain week.

Projects
````````

The list of times can be set to the exact projects you want reports on. For
instance, if you're only interested in two projects, ``gwm`` and ``pgd``, out of
a large number of projects , you can easily choose to only view the time entries
associated with one, the other, or both.

Activities
``````````

If you're only interested in the time users spent on one particular activity,
that's easy as well. Just as with projects, you can elect to view an individual
activity, or some subset of all activities.

Users
`````

Just as with projects and activities, the time entries can be selected by the
users that worked on them.

Date worked
```````````

If you are only interested in a certain date range worked, you can set the
start, end, or both to limit the timespan of the time entries selected.

Example
```````

An administrator can opt to view only entries by ``alice`` and ``bob`` on the
``awesome-proj`` and ``boring-proj`` from ``2015-02-27`` to ``2015-03-14`` by
setting the parameters on their client to the above values.
