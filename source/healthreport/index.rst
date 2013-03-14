.. healthreport_index:

=====================
Firefox Health Report
=====================

The Firefox Health Report is a daily client *ping* that uploads basic
product usage information.

Payload Format
==============

Currently, the Firefox Health Report is submitted as a compressed JSON
document. The root JSON element is an object. A *version* field defines
the version of the payload which in turn defines the expected contents
the object.

Version 2
---------

Version 2 is the same as version 1 with the exception that it has an additional
top-level field, *geckoAppInfo*, which contains basic application info.

geckoAppInfo
^^^^^^^^^^^^

This field is an object that is a simple map of string keys and values
describing basic application metadata.

It's keys are as follows:

vendor
    The value of nsXREAppData.vendor. Can be empty an empty string. For
    official Mozilla builds, this will be "Mozilla".

name
    The value of nsXREAppData.name. For official Firefox builds, this
    will be "Firefox".

id
    The value of nsXREAppData.ID.

version
    The value of nsXREAppData.version. This is the application's version. e.g.
    "21.0.0".

appBuildID
    The build ID/date of the application. e.g. "20130314113542".

platformVersion
    The version of the Gecko platform (as opposed to the app version). For
    Firefox, this is almost certainly equivalent to the *version* field.

platformBuildID
    The build ID/date of the Gecko platfor (as opposed to the app version).
    This is commonly equivalent to *appBuildID*.

os
    The name of the operating system the application is running on.

xpcomabi
    The binary architecture of the build.

Version 1
---------

Top-level Properties
^^^^^^^^^^^^^^^^^^^^

The main JSON object contains the following properties:

lastPingDate
    UTC date of the last upload. If this is the first upload from this client,
    this will not be present.

thisPingDate
    UTC date when this payload was constructed.

version
    Integer version of this payload format. Currently only 1 is defined.

data
    Object holding data constituting health report.

Data Properties
^^^^^^^^^^^^^^^

The bulk of the health report is contained within the *data* object. This
object has the following keys:

days
   Object mapping UTC days to measurements from that day. Keys are in the
   *YYYY-MM-DD* format. e.g. "2013-03-14"

last
   Object mapping measurement names to their values.


The value of *days* and *last* are objects mapping measurement names to that
measurement's values. The values are always objects. Each object contains
a *_v* property. This property defines the version of this measurement.
Additional non-underscore-prefixed properties are defined by the measurement
itself (see sections below).

Example
^^^^^^^

Here is an example JSON document for version 1::

    {
      "version": 1,
      "thisPingDate": "2013-03-11",
      "lastPingDate": "2013-03-10",
      "data": {
        "last": {
          "org.mozilla.addons.active": {
            "masspasswordreset@johnathan.nightingale": {
              "userDisabled": false,
              "appDisabled": false,
              "version": "1.05",
              "type": "extension",
              "scope": 1,
              "foreignInstall": false,
              "hasBinaryComponents": false,
              "installDay": 14973,
              "updateDay": 15317
            },
            "places-maintenance@bonardo.net": {
              "userDisabled": false,
              "appDisabled": false,
              "version": "1.3",
              "type": "extension",
              "scope": 1,
              "foreignInstall": false,
              "hasBinaryComponents": false,
              "installDay": 15268,
              "updateDay": 15379
            },
            "_v": 1
          },
          "org.mozilla.appInfo.appinfo": {
            "_v": 1,
            "appBuildID": "20130309030841",
            "distributionID": "",
            "distributionVersion": "",
            "hotfixVersion": "",
            "id": "{ec8030f7-c20a-464f-9b0e-13a3a9e97384}",
            "locale": "en-US",
            "name": "Firefox",
            "os": "Darwin",
            "platformBuildID": "20130309030841",
            "platformVersion": "22.0a1",
            "updateChannel": "nightly",
            "vendor": "Mozilla",
            "version": "22.0a1",
            "xpcomabi": "x86_64-gcc3"
          },
          "org.mozilla.profile.age": {
            "_v": 1,
            "profileCreation": 12444
          },
          "org.mozilla.appSessions.current": {
            "_v": 3,
            "startDay": 15773,
            "activeTicks": 522,
            "totalTime": 70858,
            "main": 1245,
            "firstPaint": 2695,
            "sessionRestored": 3436
          },
          "org.mozilla.sysinfo.sysinfo": {
            "_v": 1,
            "cpuCount": 8,
            "memoryMB": 16384,
            "architecture": "x86-64",
            "name": "Darwin",
            "version": "12.2.1"
          }
        },
        "days": {
          "2013-03-11": {
            "org.mozilla.addons.counts": {
              "_v": 1,
              "extension": 15,
              "plugin": 12,
              "theme": 1
            },
            "org.mozilla.places.places": {
              "_v": 1,
              "bookmarks": 757,
              "pages": 104858
            },
            "org.mozilla.appInfo.appinfo": {
              "_v": 1,
              "isDefaultBrowser": 1
            }
          },
          "2013-03-10": {
            "org.mozilla.addons.counts": {
              "_v": 1,
              "extension": 15,
              "plugin": 12,
              "theme": 1
            },
            "org.mozilla.places.places": {
              "_v": 1,
              "bookmarks": 757,
              "pages": 104857
            },
            "org.mozilla.searches.counts": {
              "_v": 1,
              "google.urlbar": 4
            },
            "org.mozilla.appInfo.appinfo": {
              "_v": 1,
              "isDefaultBrowser": 1
            }
          }
        }
      }
    }

Measurements
============

The bulk of payloads consists of measurement data. An individual measurement
is merely a collection of related values e.g. *statistics about the Places
database* or *system information*.

Each measurement has an integer version number attached. When the fields in
a measurement or the semantics of data within that measurement change, the
version number is incremented.

All measurements are defined alphabetically in the sections below.

org.mozilla.addons.active
-------------------------

This measurement contains information about the currently-installed add-ons.

Version 1
^^^^^^^^^

The measurement object is a mapping of add-on IDs to objects containing
add-on metadata.

Each add-on contains the following properties:

* userDisabled
* appDisabled
* version
* type
* scope
* foreignInstall
* hasBinaryComponents
* installDay
* updateDay

With the exception of *installDay* and *updateDay*, all these properties
come direct from the Addon instance. See https://developer.mozilla.org/en-US/docs/Addons/Add-on_Manager/Addon.
*installDay* and *updateDay* are the number of days since UNIX epoch of
the add-ons *installDate* and *updateDate* properties, respectively.

Notes
^^^^^

Add-ons that have opted out of AMO updates are not included in the list.

Example
^^^^^^^
::

    "org.mozilla.addons.active": {
      "_v": 1,
      "SQLiteManager@mrinalkant.blogspot.com": {
        "userDisabled": false,
        "appDisabled": false,
        "version": "0.7.7",
        "type": "extension",
        "scope": 1,
        "foreignInstall": false,
        "hasBinaryComponents": false,
        "installDay": 15196,
        "updateDay": 15307
      },
      "testpilot@labs.mozilla.com": {
        "userDisabled": false,
        "appDisabled": false,
        "version": "1.2.2",
        "type": "extension",
        "scope": 1,
        "foreignInstall": false,
        "hasBinaryComponents": false,
        "installDay": 15176,
        "updateDay": 15595
      }
    }


org.mozilla.addons.counts
-------------------------

This measurement contains information about historical add-on counts.

Version 1
^^^^^^^^^

The measurement object consists of counts of different add-on types. The
properties are:

extension
    Integer count of installed extensions.
plugin
    Integer count of installed plugins.
theme
    Integer count of installed themes.
lwtheme
    Integer count of installed lightweigh themes.

Notes
^^^^^

Add-ons opted out of AMO updates are included in the counts. This differs from
the behavior of the active add-ons measurement.

If no add-ons of a particular type are installed, the property for that type
will not be present (as opposed to an explicit property with value of 0).

Example
^^^^^^^

::

    "2013-03-14": {
      "org.mozilla.addons.counts": {
        "_v": 1,
        "extension": 21,
        "plugin": 4,
        "theme": 1
      }
    }



org.mozilla.appInfo.appinfo
---------------------------

This measurement contains basic XUL application and Gecko platform
information. It is reported in the *last* section.

Version 1
^^^^^^^^^

The measurement object contains mostly string values describing the
current application and build. The properties are:

* vendor
* name
* id
* version
* appBuildID
* platformVersion
* platformBuildID
* os
* xpcomabi
* updateChannel
* distributionID
* distributionVersion
* hotfixVersion
* locale
* isDefaultBrowser

Notes
^^^^^

All of the properties appear in the *last* section except for
*isDefaultBrowser*, which appears under *days*.

Example
^^^^^^^

This example comes from an official OS X Nightly build::

    "org.mozilla.appInfo.appinfo": {
      "_v": 1,
      "appBuildID": "20130311030946",
      "distributionID": "",
      "distributionVersion": "",
      "hotfixVersion": "",
      "id": "{ec8030f7-c20a-464f-9b0e-13a3a9e97384}",
      "locale": "en-US",
      "name": "Firefox",
      "os": "Darwin",
      "platformBuildID": "20130311030946",
      "platformVersion": "22.0a1",
      "updateChannel": "nightly",
      "vendor": "Mozilla",
      "version": "22.0a1",
      "xpcomabi": "x86_64-gcc3"
    },

org.mozilla.appInfo.versions
----------------------------

This measurement contains a history of application version numbers.

When the application version (*version* from *org.mozilla.appinfo.appinfo*)
changes, we record the new version on the day the change was seen. The new
versions for a day are recorded in an array under the *version* property.

Example
^^^^^^^

::

    "2013-02-20": {
      "org.mozilla.appInfo.versions": {
        "_v": 1,
        "version": [
          "22.0a1"
        ]
      }
    }

org.mozilla.appSessions.current
-------------------------------

This measurement contains information about the currently running XUL
application's session.

Version 3
^^^^^^^^^

This measurement has the following properties:

startDay
    Integer days since UNIX epoch when this session began.
activeTicks
    Integer count of *ticks* the session was active for. Gecko periodically
    sends out a signal when the session is active. Session activity
    involves keyboard or mouse interaction with the application. Each tick
    represents a window of 5 seconds where there was interaction.
totalTime
    Integer seconds the session has been alive.
main
    Integer milliseconds it took for the Gecko process to start up.
firstPaint
    Integer milliseconds from process start to first paint.
sessionRestored
    Integer milliseconds from process start to session restore.

Example
^^^^^^^

::

    "org.mozilla.appSessions.current": {
      "_v": 3,
      "startDay": 15775,
      "activeTicks": 4282,
      "totalTime": 249422,
      "main": 851,
      "firstPaint": 3271,
      "sessionRestored": 5998
    }

org.mozilla.appSessions.previous
--------------------------------

This measurement contains information about previous XUL application sessions.

Version 3
^^^^^^^^^

This measurement contains per-day lists of all the sessions started on that
day. The following properties may be present:

cleanActiveTicks
    Active ticks of sessions that were properly shut down.
cleanTotalTime
    Total number of seconds for sessions that were properly shut down.
abortedActiveTicks
    Active ticks of sessions that were not properly shut down.
abortedTotalTime
    Total number of seconds for sessions that were not properly shut down.
main
    Time in milliseconds from process start to main process initialization.
firstPaint
    Time in milliseconds from process start to first paint.
sessionRestored
    Time in milliseconds from process start to session restore.

Notes
^^^^^

Sessions are recorded on the date on which they began.

If a session was aborted/crashed, the total time may be less than the actual
total time. This is because we don't always update total time during periods
of inactivity and the abort/crash could occur after a long period of idle,
before we've updated the total time.

The lengths of the arrays for {cleanActiveTicks, cleanTotalTime},
{abortedActiveTicks, abortedTotalTime}, and {main, firstPaint, sessionRestored}
should all be identical.

The length of the clean sessions plus the length of the aborted sessions should
be equal to the length of the {main, firstPaint, sessionRestored} properties.

It is not possible to distinguish the main, firstPaint, and sessionRestored
values from a clean vs aborted session: they are all lumped together.

For sessions spanning multiple UTC days, it's not possible to know which
days the session was active for. It's possible a week long session only
had activity for 2 days and there's no way for us to tell which days.

Example
^^^^^^^

::

    "org.mozilla.appSessions.previous": {
      "_v": 3,
      "cleanActiveTicks": [
        78,
        1785
      ],
      "cleanTotalTime": [
        4472,
        88908
      ],
      "main": [
        32,
        952
      ],
      "firstPaint": [
        2755,
        3497
      ],
      "sessionRestored": [
        5149,
        5520
      ]
    }

org.mozilla.crashes.crashes
---------------------------

This measurement contains a historical record of application crashes.

Version 1
^^^^^^^^^

This measurement will be reported on each day there was a crash. The
following properties are reported:

pending
    The number of crash reports that haven't been submitted.
submitted
    The number of crash reports that were submitted.

Notes
^^^^^

Crashes are typically submitted immediately after they occur (by checking
a box in the crash reporter, which should appear automatically after a
crash). If the crash reporter submits the crash successfully, we get a
submitted crash. Else, we leave it as pending.

A pending crash does not mean it will eventually be submitted.

Pending crash reports can be submitted post-crash by going to
about:crashes.

If a pending crash is submitted via about:crashes, the submitted count
increments but the pending count does not decrement. This is because FHR
does not know which pending crash was just submitted and therefore it does
not know which day's pending crash to decrement.

Example
^^^^^^^

::

    "org.mozilla.crashes.crashes": {
      "_v": 1,
      "pending": 1,
      "submitted": 2
    }

org.mozilla.places.places
-------------------------

This measurement contains information about the Places database (where Firefox
stores its history and bookmarks).

Version 1
^^^^^^^^^

Daily counts of items in the database are reported in the following properties:

bookmarks
    Integer count of bookmarks present.
pages
    Integer count of pages in the history database.

Example
^^^^^^^

::

    "org.mozilla.places.places": {
      "_v": 1,
      "bookmarks": 388,
      "pages": 94870
    }

org.mozilla.profile.age
-----------------------

This measurement contains information about the current profile's age.

Version 1
^^^^^^^^^

A single *profileCreation* property is present. It defines the integer
days since UNIX epoch that the current profile was created.

Notes
^^^^^

It is somewhat difficult to obtain a reliable *profile born date* due to a
number of factors.

Example
^^^^^^^

::

    "org.mozilla.profile.age": {
      "_v": 1,
      "profileCreation": 15176
    }

org.mozilla.searches.counts
---------------------------

This measurement contains information about searches performed in the
application.

Version 1
^^^^^^^^^

We record counts of performed searches grouped by search engine and search
origin. Only search engines with which Mozilla has a business relationship
are explicitly counted. All other search engines are grouped into an
*other* bucket.

The following search engines are explicitly counted:

* Amazon.com
* Bing
* Google
* Yahoo
* Other

The following search origins are distinguished:

about:home
    Searches initiated from the search text box on about:home.
context menu
    Searches initiated from the context menu (highlight text, right click,
    and select "search for...")
search bar
    Searches initiated from the search bar (the text field next to the
    Awesomebar)
url bar
    Searches initiated from the awesomebar/url bar.

Notes
^^^^^

Due to the localization of search engine names, non en-US locales may wrongly
attribute searches to the *other* bucket. Bug 841554 tracks.

Example
^^^^^^^

::

    "org.mozilla.searches.counts": {
      "_v": 1,
      "google.searchbar": 3,
      "google.urlbar": 7
    },

org.mozilla.sysinfo.sysinfo
---------------------------

This measurement contains basic information about the system the application
is running on.

Version 1
^^^^^^^^^

The following properties may be available:

cpuCount
    Integer number of CPUs/cores in the machine.
memoryMB
    Integer megabytes of memory in the machine.
manufacturer
    The manufacturer of the device.
device
    The name of the device (like model number).
hardware
    Unknown.
name
    OS name.
version
    OS version.
architecture
    OS architecture.

Example
^^^^^^^

::

    "org.mozilla.sysinfo.sysinfo": {
      "_v": 1,
      "cpuCount": 8,
      "memoryMB": 8192,
      "architecture": "x86-64",
      "name": "Darwin",
      "version": "12.2.0"
    }

