*******
Tracing
*******

.. current-library:: tracing

This is a tracing library based on the `Dapper`_ work by Google.
It is also similar to Twitter's `Zipkin`_ and `HTrace`_ from Cloudera
which were also inspired by the `Dapper`_ work.

Tracing is different from simply recording metrics about a service
or logging text strings to a log file. Tracing is intended to capture
information about the units of work that are being performed, how they
relate to one another, and how long the pieces of the units of work
take to execute.

Each unit of work at the top level is captured by a trace which consists
of a tree of spans (:class:`<span>`). Each unit of work is tracked by
a span in the tree.

Each span can have key/value data associated with it via either
:func:`trace-add-data` or :func:`span-add-data`. This data typically
represents data associated with the computation or action. Examples
might include the user name making a request, the SQL being executed,
the table being queried, or the IP address associated with a request.

Each span can also have timestamped annotations provided via either
:func:`trace-annotate` or :func:`span-annotate`. This associates a
text description of an event with a timestamp and the span. This might
be used to indicate progress through a task, unusual events, or
anything interesting.

Other important details are discussed below, such as `Writers`_
and `Sampling`_.

.. _Dapper: http://research.google.com/pubs/pub36356.html
.. _Zipkin: http://twitter.github.io/zipkin/
.. _HTrace: https://github.com/cloudera/htrace/

Instrumenting Your Code
=======================

Instrumenting your code with tracing should be pretty easy.

At application startup, call :func:`trace-set-host` with a
string identifier that is unique among the machines running
the software.

To start tracing, either call :func:`trace-push` and
:func:`trace-pop` or use :macro:`with-tracing`:

.. code-block:: dylan

   with-tracing ("http-request")
     trace-add-data("client-ip-address", ...);
     trace-add-data("requested-path", ...);

     ... do work ...
   end;

Setting up Sampling
-------------------

Sampling of traces is left up to the application. A few
basic primitives are supplied in :func:`always-sample`,
:func:`never-sample`, :func:`if-tracing-sample`, and
related functions. See `Sampling`_ for more information.

The TRACING-CORE module
=======================

.. current-module:: tracing-core

.. contents::
   :local:

Tracing
-------

The tracing functions in this section represent the high level
interface to the tracing library and are what would typically
be used, rather than the span-specific functions.

There may be times though when using the lower level,
span-specific functions is appropriate, such as when you have
multiple units of work executing asynchronously. The asynchronous
tasks may find it easier to track their own spans separately.

.. function:: trace-push

   :signature: trace-push (description #key sampler) => (span?)

   :parameter description: An instance of :drm:`<string>`.
   :parameter #key sampler: An instance of :drm:`<function>`.
   :value span?: An instance of ``false-or(<span>)``.

   :description:

     Create a new :class:`<span>` and make it the current tracing
     span. If there is already a span, the new span will use the
     existing span as the parent.

     ``sampler`` defaults to :func:`if-tracing-sample`.

   :seealso:

     * :func:`trace-pop`

.. function:: trace-add-data

   :signature: trace-add-data (key data) => ()

   :parameter key: An instance of :drm:`<string>`.
   :parameter data: An instance of :drm:`<string>`.

   :description:

     Adds key / value data to the current trace span (if any),
     using :gf:`span-add-data`.

   :seealso:

     * :gf:`span-add-data`

.. function:: trace-annotate

   :signature: trace-annotate (description) => ()

   :parameter description: An instance of :drm:`<string>`.

   :description:

     Adds an annotation to the current trace span (if any), using
     :gf:`span-annotate`.

   :seealso:

     * :gf:`span-annotate`

.. function:: trace-pop

   :signature: trace-pop (span?) => ()

   :parameter span?: An instance of ``false-or(<span>)``.

   :description:

     Stops the current span and pops it from the stack, returning
     the previous span to the current slot.

   :seealso:

     * :func:`trace-push`

.. function:: trace-set-host

   Sets the global host identifier that will be set on all spans created by this
   process.

   :signature: trace-set-host (host) => ()

   :parameter host: An instance of :drm:`<string>`.

   :description:

      Sets the global host identifier that will be set on all spans created
      by this process.

      This may just be a hostname, but if multiple processes are running on
      the same host, then it should include additional identifying data.
      Ideally, this identifier will be unique per process within a cluster
      in a distributed system.

      .. note:: This function should be called early in the application
         startup, before any tracing is performed.

   :seealso:

     * :gf:`span-host`

.. macro:: with-tracing

   :macrocall:

     .. code-block:: dylan

       with-tracing ("Span description")
         trace-add-data("Table", "users");
         ...
       end with-tracing;

       with-tracing ("Span description", sampler: never-sample)
         ...
       end with-tracing;

   :description:

     The ``with-tracing`` macro simplifies the process of calling :gf:`trace-push`
     and :gf:`trace-pop`. It can also take any keywords that :gf:`trace-push`
     takes and will pass them along.


Spans
-----

.. class:: <span>

   :superclasses: <object>

   :keyword description:
   :keyword parent-id:
   :keyword trace-id:

   :description:

     A span tracks a period of time associated with a computation
     or action, along with annotations and key / value data. Spans
     exist within a tree of spans all of which share the same
     ``trace-id``.

.. generic-function:: span-add-data

   :signature: span-add-data (span key data) => ()

   :parameter span: An instance of :class:`<span>`.
   :parameter key: An instance of :drm:`<string>`.
   :parameter data: An instance of :drm:`<string>`.

   :description:

      Key / value pairs may be stored on a span to provide better
      context. This might include the query being executed, address
      or host information or whatever is relevant to the application
      being traced.

   :seealso:

     * :gf:`span-data`

.. generic-function:: span-annotate

   :signature: span-annotate (span description) => ()

   :parameter span: An instance of :class:`<span>`.
   :parameter description: An instance of :drm:`<string>`.

   :description:

      Annotations are to record an occurrence of an event
      during a span. They have a specific timestamp associated
      with them that is automatically set to the time when
      the annotation is created.

   :seealso:

     * :gf:`span-annotations`
     * :class:`<span-annotation>`
     * :gf:`annotation-description`
     * :gf:`annotation-timestamp`

.. generic-function:: span-annotations

   Returns the collection of :class:`<span-annotation>` associated with
   this span.

   :signature: span-annotations (span) => (annotations)

   :parameter span: An instance of :class:`<span>`.
   :value annotations: An instance of :drm:`<vector>`.

   :seealso:

     * :gf:`span-annotate`
     * :class:`<span-annotation>`
     * :gf:`annotation-description`
     * :gf:`annotation-timestamp`

.. generic-function:: span-data

   Returns the property list of data associated with this span.

   :signature: span-data (span) => (data)

   :parameter span: An instance of :class:`<span>`.
   :value data: An instance of :drm:`<vector>`.

   :seealso:

     * :gf:`span-add-data`

.. generic-function:: span-description

   Returns the description of the span.

   :signature: span-description (span) => (description)

   :parameter span: An instance of :class:`<span>`.
   :value description: An instance of :drm:`<string>`.

.. generic-function:: span-duration

   :signature: span-duration (span) => (time?)

   :parameter span: An instance of :class:`<span>`.
   :value time?: An instance of ``false-or(<duration>)``.

   :description:

     If the span has not yet been stopped, this returns ``#f``. Once
     the span has been stopped, the duration that the span was running
     will be returned.

   :seealso:

     * :gf:`span-start-time`
     * :class:`<duration>`
     * :gf:`duration-microseconds`
     * :gf:`duration-seconds`

.. generic-function:: span-host

   Returns the host identifier for the process which
   created this span.

   :signature: span-host (span) => (host)

   :parameter span: An instance of :class:`<span>`.
   :value host: An instance of :drm:`<string>`.

.. generic-function:: span-id

   Returns the unique ID associated with this span.

   :signature: span-id (span) => (id)

   :parameter span: An instance of :class:`<span>`.
   :value id: An instance of ``<object>``.

.. generic-function:: span-parent-id

   :signature: span-parent-id (span) => (id)

   :parameter span: An instance of :class:`<span>`.
   :value id: An instance of ``<object>``.

.. generic-function:: span-process-id

   :signature: span-process-id (span) => (process-id)

   :parameter span: An instance of :class:`<span>`.
   :value process-id: An instance of :drm:`<integer>`.

.. generic-function:: span-start-time

   Returns the time that the span was created.

   :signature: span-start-time (span) => (timestamp)

   :parameter span: An instance of :class:`<span>`.
   :value timestamp: An instance of :class:`<timestamp>`.

   Like other time values in Dylan, this is the time since
   January 1, 1900.

   :seealso:

     * :gf:`span-duration`
     * :class:`<timestamp>`
     * :gf:`timestamp-days`
     * :gf:`timestamp-microseconds`
     * :gf:`timestamp-seconds`

.. generic-function:: span-stop

   Stops a span and sends it to the current registered
   :class:`<span-writer>` instances.

   :signature: span-stop (span) => ()

   :parameter span: An instance of :class:`<span>`.

   :seealso:

     * :gf:`span-stopped?`
     * :func:`store-span`

.. generic-function:: span-stopped?

   Has the span been stopped yet?

   :signature: span-stopped? (span) => (stopped?)

   :parameter span: An instance of :class:`<span>`.
   :value stopped?: An instance of :drm:`<boolean>`.

   :seealso:

     * :gf:`span-stop`

.. generic-function:: span-thread-id

   :signature: span-thread-id (span) => (thread-id)

   :parameter span: An instance of :class:`<span>`.
   :value thread-id: An instance of :drm:`<integer>`.

.. generic-function:: span-trace-id

   Return the trace-id for a span.

   :signature: span-trace-id (span) => (id)

   :parameter span: An instance of :class:`<span>`.
   :value id: An instance of ``<object>``.

   :description:

     Returns the trace-id for a span. This ID is the same for all
     spans within a single trace.

Annotations
-----------

Annotations let you attach events that happened at a point in time
(noted by a timestamp) to a span.

.. class:: <span-annotation>

   :superclasses: <object>

   :keyword description:
   :keyword timestamp:

.. generic-function:: annotation-description

   Return the description of an annotation.

   :signature: annotation-description (annotation) => (description)

   :parameter annotation: An instance of :class:`<span-annotation>`.
   :value description: An instance of :drm:`<string>`.

.. generic-function:: annotation-timestamp

   Return the timestamp at which the annotation was created and attached.

   :signature: annotation-timestamp (annotation) => (timestamp)

   :parameter annotation: An instance of :class:`<span-annotation>`.
   :value timestamp: An instance of :class:`<timestamp>`.

Sampling
--------

Samplers allow for collecting a subset of the data, making the
usage of this tracing framework in a heavily loaded production
scenario more realistic.

Samplers are simply functions that return a boolean value
indicating whether or not an actual trace should be generated
and recorded. They should be called at the appropriate point
within the application being traced. Some applications may
wish to limit which traces or parts of traces are collected
in ways that are not readily representable within this framework.

.. function:: always-sample

   Always returns true, so that the trace is sampled.

   :signature: always-sample () => #t

   :value record-sample?: Always ``#t``.

.. function:: if-tracing-sample

   Returns true if tracing is enabled, otherwise ``#f``.

   :signature: if-tracing-sample () => (record-sample?)

   :value record-sample?: An instance of :drm:`<boolean>`.

   :seealso:

     * :func:`disable-tracing`
     * :func:`enable-tracing`

.. function:: never-sample

   Always returns false, so that the trace isn't sampled.

   :signature: never-sample () => #f

   :value record-sample?: Always ``#f``.

.. function:: disable-tracing

   :signature: disable-tracing () => ()

   :description:

     This function only modifies the return value of :func:`if-tracing-sample`
     and does not globally disable tracing.

   :seealso:

     * :func:`enable-tracing`

.. function:: enable-tracing

   :signature: enable-tracing () => ()

   :description:

     This function only modifies the return value of :func:`if-tracing-sample`
     and does not globally enable tracing.

   :seealso:

     * :func:`disable-tracing`

Writers
-------

Spans are stored by using instances of :class:`<span-writer>` which
have been registered using :func:`register-span-writer`. Spans are
stored when they are stopped (:func:`trace-pop`, :func:`span-stop`).
Spans are also stored when they are finalized without having been
stopped previously. This finalization is only present to prevent
data from being lost and should not be a default mode of operation.

.. class:: <span-writer>

   :superclasses: <object>

   :seealso:

     * :func:`register-span-writer`
     * :func:`registered-span-writers`
     * :func:`unregister-span-writer`

.. function:: register-span-writer

   :signature: register-span-writer (span-writer) => ()

   :parameter span-writer: An instance of :class:`<span-writer>`.

   :seealso:

     * :class:`<span-writer>`
     * :func:`registered-span-writers`
     * :func:`unregister-span-writer`

.. function:: registered-span-writers

   :signature: registered-span-writers () => (span-writers)

   :value span-writers: An instance of ``<span-writer-vector>``.

   :seealso:

     * :class:`<span-writer>`
     * :func:`register-span-writer`
     * :func:`unregister-span-writer`

.. function:: store-span

   :signature: store-span (span) => ()

   :parameter span: An instance of :class:`<span>`.

   :seealso:

     * :func:`registered-span-writers`

.. function:: unregister-span-writer

   :signature: unregister-span-writer (span-writer) => ()

   :parameter span-writer: An instance of :class:`<span-writer>`.

   :seealso:

     * :class:`<span-writer>`
     * :func:`register-span-writer`
     * :func:`registered-span-writers`

Writer Implementation
---------------------

To add a new storage class, subclass :class:`<span-writer>` and
implement the :gf:`span-writer-add-span` method. Then, call
:func:`register-span-writer` with an instance of your span
writer and all subsequent spans completed will be written to it.

.. generic-function:: span-writer-add-span

   :signature: span-writer-add-span (span span-writer) => ()

   :parameter span: An instance of :class:`<span>`.
   :parameter span-writer: An instance of :class:`<span-writer>`.

   :description:

      This method is specialized for each subclass of
      :class:`<span-writer>`. It is called whenever a span
      needs to be processed by a span writer.

Time Utilities
--------------

.. class:: <duration>

   Measure of time elapsed.

   :superclasses: <object>

   :keyword microseconds:
   :keyword seconds:

   :seealso:

     * :gf:`duration-microseconds`
     * :gf:`duration-seconds`

.. generic-function:: duration-microseconds

   :signature: duration-microseconds (duration) => (microseconds)

   :parameter duration: An instance of :class:`<duration>`.
   :value microseconds: An instance of :drm:`<integer>`.

   :seealso:

     * :gf:`duration-seconds`

.. generic-function:: duration-seconds

   :signature: duration-seconds (duration) => (seconds)

   :parameter duration: An instance of :class:`<duration>`.
   :value seconds: An instance of :drm:`<integer>`.

   :seealso:

     * :gf:`duration-microseconds`

.. class:: <timestamp>

   A point in time.

   :superclasses: <object>

   :keyword microseconds:
   :keyword seconds:

   :seealso:

     * :gf:`timestamp-days`
     * :gf:`timestamp-microseconds`
     * :gf:`timestamp-seconds`

.. generic-function:: timestamp-days

   :signature: timestamp-days (timestamp) => (days)

   :parameter timestamp: An instance of :class:`<timestamp>`.
   :value days: An instance of :drm:`<integer>`.

   :seealso:

     * :gf:`timestamp-microseconds`
     * :gf:`timestamp-seconds`

.. generic-function:: timestamp-microseconds

   :signature: timestamp-microseconds (timestamp) => (microseconds)

   :parameter timestamp: An instance of :class:`<timestamp>`.
   :value microseconds: An instance of :drm:`<integer>`.

   :seealso:

     * :gf:`timestamp-days`
     * :gf:`timestamp-seconds`

.. generic-function:: timestamp-seconds

   :signature: timestamp-seconds (timestamp) => (seconds)

   :parameter timestamp: An instance of :class:`<timestamp>`.
   :value seconds: An instance of :drm:`<integer>`.

   :seealso:

     * :gf:`timestamp-days`
     * :gf:`timestamp-microseconds`

Tags
----

These constants are available to help standardize tracing across
applications.

.. constant:: $tag/peer/host-name

.. constant:: $tag/peer/ip

.. constant:: $tag/peer/port

.. constant:: $tag/http/uri

.. constant:: $tag/http/response/size

.. constant:: $tag/http/status


Miscellaneous
-------------

.. function:: get-unique-id

   :signature: get-unique-id () => (id)

   :value id: An instance of ``<unique-id>``.

