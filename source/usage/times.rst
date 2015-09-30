.. _time_entries:

============
Time Entries
============

TimeSync entries have a few main fields:

* ``duration``: the amount of time worked, in seconds
* ``project``: the project you worked on during the time entry
* ``activities``: the activities you worked on. This can be things like ``dev``,
  ``docs``, etc.
* ``user``: your username. Right now, only the account authenticated can only
  make time entries for itself.
* ``date worked``: the date you did the work on. Many TimeSync clients will
  default to the day you're sending the entry in.

There are a few optional fields as well:

* ``issue_uri``: a valid link to the issue you're working on. Generally this
  would be something like a GitHub issue, an RT ticket, etc.
* ``notes``: a text field containing whatever you want. Useful for recording a
  small note on the exact task you completed/worked on with more detail than the
  activities field.

Each time entry will automatically include a created timestamp, and an updated
timestamp if the entry is modified.
