.. highlight:: dylan

****************
The TIME library
****************

.. current-library:: time
.. current-module:: time

This time library is intended as a modernization of Open Dylan's "date" module
in the "system" library. It has a nanosecond precision and more efficient time
representation, and a more complete API.

Current Status
==============

* This is **alpha** software.
* It works only on Linux (possibly on macOS but it hasn't been tried).
* It works only on 64-bit machines.
* It can get the current time, to nanosecond precision.
* It can format times and durations.
* Fixed-offset timezones work.
* Support for "aware" timezones is underway. Support for TZif format is about
  half done.
* The API should be considered provisional.

You have been warned. Enjoy!

.. note:: This documentation contains little more than the stubs auto-generated
          by :command:`export -format rst interface-reference`. Use the sauce,
          Luke.


Reference
=========

Each of the following constants is bound to the corresponding instance of
:class:`<month>`.

.. constant:: $january
.. constant:: $february
.. constant:: $march
.. constant:: $april
.. constant:: $may
.. constant:: $june
.. constant:: $july
.. constant:: $august
.. constant:: $september
.. constant:: $october
.. constant:: $november
.. constant:: $december

Each of the following constants is bound to the corresponding length of a
:class:`<duration>`. (Caveat emptor etc etc)

.. constant:: $week
.. constant:: $day
.. constant:: $hour
.. constant:: $minute
.. constant:: $second
.. constant:: $millisecond
.. constant:: $microsecond
.. constant:: $nanosecond

Each of the following constants is bound to the corresponding instance of
:class:`<day>`.

.. constant:: $monday
.. constant:: $tuesday
.. constant:: $wednesday
.. constant:: $thursday
.. constant:: $friday
.. constant:: $saturday
.. constant:: $sunday

Each of the following constants is an instance of :class:`<time-format>`.

.. constant:: $rfc3339
.. constant:: $rfc3339-microseconds
.. constant:: $rfc3339-milliseconds

Each of the following constants is bound to the corresponding instance of
:class:`<time>`.

.. constant:: $epoch
.. constant:: $minimum-time
.. constant:: $maximum-time

.. constant:: $utc

   An instance of :class:`<zone>` representing Coordinated Universal Time
   (UTC).

.. class:: <day>

   :superclasses: :drm:`<object>`

   :keyword required long-name: An instance of :drm:`<string>`.
   :keyword required short-name: An instance of :drm:`<string>`.

.. class:: <duration>

   :superclasses: :drm:`<object>`

   :keyword nanoseconds: An instance of :drm:`<integer>`.

.. class:: <month>

   :superclasses: :drm:`<object>`

   :keyword required days: An instance of :drm:`<integer>`.
   :keyword required long-name: An instance of :drm:`<string>`.
   :keyword required number: An instance of :drm:`<integer>`.
   :keyword required short-name: An instance of :drm:`<string>`.

.. class:: <time-error>

   :superclasses: :drm:`<simple-error>`


.. class:: <time-format>

   :superclasses: :drm:`<object>`

   :keyword parsed: An instance of :drm:`<sequence>`.
   :keyword required string: An instance of :drm:`<string>`.

.. class:: <time>
   :primary:

   :superclasses: :drm:`<object>`

   :keyword required days: An instance of :drm:`<integer>`.
   :keyword required nanoseconds: An instance of :drm:`<integer>`.
   :keyword zone: An instance of :class:`<zone>`.

.. class:: <zone>
   :abstract:

   :superclasses: :drm:`<object>`

   :keyword required name: An instance of :drm:`<string>`.

.. generic-function:: compose-time

   :signature: compose-time (y mon d h min sec nano zone) => (_)

   :parameter y: An instance of :drm:`<integer>`.
   :parameter mon: An instance of :class:`<month>`.
   :parameter d: An instance of :drm:`<integer>`.
   :parameter h: An instance of :drm:`<integer>`.
   :parameter min: An instance of :drm:`<integer>`.
   :parameter sec: An instance of :drm:`<integer>`.
   :parameter nano: An instance of :drm:`<integer>`.
   :parameter zone: An instance of :class:`<zone>`.
   :value _: An instance of :class:`<time>`.

.. generic-function:: day-long-name

   :signature: day-long-name (object) => (value)

   :parameter object: An instance of ``{<day> in time}``.
   :value value: An instance of :drm:`<string>`.

.. generic-function:: day-short-name

   :signature: day-short-name (object) => (value)

   :parameter object: An instance of ``{<day> in time}``.
   :value value: An instance of :drm:`<string>`.

.. generic-function:: duration-nanoseconds

   :signature: duration-nanoseconds (object) => (value)

   :parameter object: An instance of ``{<duration> in time}``.
   :value value: An instance of :drm:`<integer>`.

.. function:: find-zone

   :signature: find-zone (name #key zones) => (zone)

   :parameter name: An instance of :drm:`<string>`.
   :parameter #key zones: An instance of :drm:`<object>`.
   :value zone: An instance of :const:`<zone>?`.

.. generic-function:: format-duration

   :signature: format-duration (stream duration #key long?) => ()

   :parameter stream: An instance of ``<stream>``.
   :parameter duration: An instance of :class:`<duration>`.
   :parameter #key long?: An instance of :drm:`<boolean>`.

.. generic-function:: format-time

   :signature: format-time (stream format time #key zone) => ()

   :parameter stream: An instance of ``<stream>``.
   :parameter format: An instance of :drm:`<object>`.
   :parameter time: An instance of :class:`<time>`.
   :parameter #key zone: An instance of :drm:`<object>`.

.. method:: format-time
   :specializer: <stream>, <time-format>, <time>

.. method:: format-time
   :specializer: <stream>, <string>, <time>

.. method:: format-time
   :specializer: <stream>, <sequence>, <time>

.. generic-function:: local-time-zone

   :signature: local-time-zone () => (zone)

   :value zone: An instance of :class:`<zone>`.

.. generic-function:: month-days

   :signature: month-days (object) => (value)

   :parameter object: An instance of ``{<month> in time}``.
   :value value: An instance of :drm:`<integer>`.

.. generic-function:: month-long-name

   :signature: month-long-name (object) => (value)

   :parameter object: An instance of ``{<month> in time}``.
   :value value: An instance of :drm:`<string>`.

.. generic-function:: month-number

   :signature: month-number (object) => (value)

   :parameter object: An instance of ``{<month> in time}``.
   :value value: An instance of :drm:`<integer>`.

.. generic-function:: month-short-name

   :signature: month-short-name (object) => (value)

   :parameter object: An instance of ``{<month> in time}``.
   :value value: An instance of :drm:`<string>`.

.. generic-function:: parse-day

   :signature: parse-day (name) => (d)

   :parameter name: An instance of :drm:`<string>`.
   :value d: An instance of :class:`<day>`.

.. generic-function:: parse-duration

   :signature: parse-duration (s #key start end) => (d end-position)

   :parameter s: An instance of :drm:`<string>`.
   :parameter #key start: An instance of :drm:`<integer>`.
   :parameter #key end: An instance of :drm:`<integer>`.
   :value d: An instance of :class:`<duration>`.
   :value end-position: An instance of :drm:`<integer>`.

.. generic-function:: parse-time

   :signature: parse-time (input #key format zone) => (time)

   :parameter input: An instance of :drm:`<string>`.
   :parameter #key format: An instance of :class:`<time-format>`.
   :parameter #key zone: An instance of :class:`<zone>`.
   :value time: An instance of :class:`<time>`.

.. generic-function:: time-components

   :signature: time-components (t #key zone) => (y mon d h min sec nano zone dow)

   :parameter t: An instance of :class:`<time>`.
   :parameter #key zone: An instance of :const:`<zone>?`.
   :value y: An instance of :drm:`<integer>`.
   :value mon: An instance of :class:`<month>`.
   :value d: An instance of :drm:`<integer>`.
   :value h: An instance of :drm:`<integer>`.
   :value min: An instance of :drm:`<integer>`.
   :value sec: An instance of :drm:`<integer>`.
   :value nano: An instance of :drm:`<integer>`.
   :value zone: An instance of :class:`<zone>`.
   :value dow: An instance of :class:`<day>`.

.. generic-function:: time-day-of-month

   :signature: time-day-of-month (t) => (day-of-month)

   :parameter t: An instance of :class:`<time>`.
   :value day-of-month: An instance of :drm:`<integer>`.

.. generic-function:: time-day-of-week

   :signature: time-day-of-week (t) => (_)

   :parameter t: An instance of :class:`<time>`.
   :value _: An instance of :class:`<day>`.

.. generic-function:: time-hour

   :signature: time-hour (t) => (hour)

   :parameter t: An instance of :class:`<time>`.
   :value hour: An instance of :drm:`<integer>`.

.. generic-function:: time-in-zone

   :signature: time-in-zone (t zone) => (t2)

   :parameter t: An instance of :class:`<time>`.
   :parameter zone: An instance of :class:`<zone>`.
   :value t2: An instance of :class:`<time>`.

.. generic-function:: time-minute

   :signature: time-minute (t) => (minute)

   :parameter t: An instance of :class:`<time>`.
   :value minute: An instance of :drm:`<integer>`.

.. generic-function:: time-month

   :signature: time-month (t) => (month)

   :parameter t: An instance of :class:`<time>`.
   :value month: An instance of :class:`<month>`.

.. generic-function:: time-nanosecond

   :signature: time-nanosecond (t) => (nanosecond)

   :parameter t: An instance of :class:`<time>`.
   :value nanosecond: An instance of :drm:`<integer>`.

.. generic-function:: time-now

   :signature: time-now (#key zone) => (t)

   :parameter #key zone: An instance of :class:`<zone>`.
   :value t: An instance of :class:`<time>`.

.. generic-function:: time-second

   :signature: time-second (t) => (second)

   :parameter t: An instance of :class:`<time>`.
   :value second: An instance of :drm:`<integer>`.

.. generic-function:: time-year

   :signature: time-year (t) => (year)

   :parameter t: An instance of :class:`<time>`.
   :value year: An instance of :drm:`<integer>`.

.. generic-function:: time-zone

   :signature: time-zone (t) => (zone)

   :parameter t: An instance of :class:`<time>`.
   :value zone: An instance of :class:`<zone>`.

.. generic-function:: zone-abbreviation

   :signature: zone-abbreviation (zone #key time) => (abbrev)

   :parameter zone: An instance of :class:`<zone>`.
   :parameter #key time: An instance of :drm:`<object>`.
   :value abbrev: An instance of :drm:`<string>`.

.. method:: zone-abbreviation
   :specializer: <naive-zone>

.. method:: zone-abbreviation
   :specializer: <aware-zone>

.. generic-function:: zone-daylight-savings?

   :signature: zone-daylight-savings? (zone #key time) => (dst?)

   :parameter zone: An instance of :class:`<zone>`.
   :parameter #key time: An instance of :drm:`<object>`.
   :value dst?: An instance of :drm:`<boolean>`.

.. method:: zone-daylight-savings?
   :specializer: <naive-zone>

.. method:: zone-daylight-savings?
   :specializer: <aware-zone>

.. generic-function:: zone-name

   :signature: zone-name (object) => (value)

   :parameter object: An instance of ``{<zone> in time}``.
   :value value: An instance of :drm:`<string>`.

.. generic-function:: zone-offset-seconds

   :signature: zone-offset-seconds (zone #key time) => (seconds)

   :parameter zone: An instance of :class:`<zone>`.
   :parameter #key time: An instance of :drm:`<object>`.
   :value seconds: An instance of :drm:`<integer>`.

.. method:: zone-offset-seconds
   :specializer: <naive-zone>

.. method:: zone-offset-seconds
   :specializer: <aware-zone>

.. generic-function:: zone-offset-string

   :signature: zone-offset-string (zone #key time) => (offset)

   :parameter zone: An instance of :class:`<zone>`.
   :parameter #key time: An instance of :drm:`<object>`.
   :value offset: An instance of :drm:`<string>`.

.. method:: zone-offset-string
   :specializer: <naive-zone>

.. method:: zone-offset-string
   :specializer: <aware-zone>


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
