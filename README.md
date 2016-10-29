TimeSync
========

<img align="right" style="padding: 5px;" src="/source/_static/timesync.png?raw=true" />

TimeSync is the OSU Open Source Lab's time tracking system. It's designed to be
simple, have a sane API, and make sense while allowing users to track their
time spent on various projects and activities.

This repository contains the source code for the TimeSync API specification
document. The documentation can also be found on [Read the Docs](http://timesync.readthedocs.io/en/latest/).

Our reference implementation of TimeSync is here:

[timesync-node repository](https://github.com/osuosl/timesync-node)

Related projects:

* [Pymsync: Python TimeSync module](https://github.com/osuosl/pymesync)
* [Rimesync: Ruby TimeSync gem](https://github.com/osuosl/rimesync)
* [TimeSync Web: Web front-end](https://github.com/osuosl/timesync-frontend-flask)
* [Climesync: TimeSync CLI](https://github.com/osuosl/timesync-node)
* [Timevis: TimeSync visualizations](https://github.com/osuosl/timevis)
* [GitSync: Github integration](https://github.com/osuosl/gitsync)
* [Timesync-RT: RT integration](https://github.com/osuosl/rt-timesync)

Usage
-----

To build the HTML rendered documentation, run the following commands:

```
$ pip install -r requirements.txt
$ make html
$ <browser> build/html/index.html
```
