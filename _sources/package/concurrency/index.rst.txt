***********
Concurrency
***********

.. current-library:: concurrency
.. current-module:: concurrency

This library provides various concurrency utilities for use with Dylan
programs.

Basic Abstractions
==================

The abstractions in this library are somewhat inspired by ``javax.concurrency``.

Executors
---------

Executors perform work that is requested from them asynchronously.

Currently, all executors use their own private threads.

See: :class:`<executor>`, :class:`<fixed-thread-executor>`, :class:`<thread-executor>`,
and :class:`<single-thread-executor>`.

Queues
------

Queues are job-streams that can have items enqueued and subsequently dequeued.

These form the synchronization mechanism for thread executors.

See: :class:`<queue>`, :class:`<locked-queue>`.

Work
----

Work objects represent something to be done.

See: :class:`<work>`, :class:`<locked-work>`.

Library Reference
=================

Executors
---------

.. class:: <executor>
   :abstract:

   :superclasses: :drm:`<object>`

   :keyword name:

   :operations:

     * :gf:`executor-name`
     * :gf:`executor-request`

.. class:: <thread-executor>
   :abstract:

   :superclasses: :class:`<executor>`

   :keyword queue:

   :operations:

     * :gf:`executor-shutdown`

.. class:: <fixed-thread-executor>

   :superclasses: :class:`<thread-executor>`

   :keyword thread-count:

.. class:: <single-thread-executor>

   :superclasses: :class:`<thread-executor>`

.. generic-function:: executor-name

   :signature: executor-name (executor) => (name)

   :parameter executor: An instance of :class:`<executor>`.
   :value name: An instance of :drm:`<string>`.

.. generic-function:: executor-request

   Request that this executor do some work.

   :signature: executor-request (executor work) => ()

   :parameter executor: An instance of :class:`<executor>`.
   :parameter work: An instance of :drm:`<object>`.

.. method:: executor-request
   :specializer: <function>

   A convenience method that converts the given function into a
   :class:`<work>` object.  The function must not have any required
   arguments.

   :signature: executor-request (executor function) => ()

   :parameter executor: An instance of :class:`<executor>`.
   :parameter work: An instance of :drm:`<function>`.

.. method:: executor-request
   :specializer: <work>

   :signature: executor-request (executor work) => ()

   :parameter executor: An instance of :class:`<executor>`.
   :parameter work: An instance of :class:`<work>`.

.. generic-function:: executor-shutdown

   :signature: executor-shutdown (executor #key join? drain?) => ()

   :parameter executor: An instance of :class:`<thread-executor>`.
   :parameter #key join?: An instance of :drm:`<boolean>`.
   :parameter #key drain?: An instance of :drm:`<boolean>`.


Queues
------

.. class:: <queue>
   :abstract:

   :superclasses: :drm:`<object>`

   :keyword name:

   :discussion:

     This is a base class for specific implementations
     that modify queueing behaviour.

   :operations:

     * :gf:`dequeue`
     * :gf:`enqueue`
     * :gf:`queue-name`

.. class:: <locked-queue>

   Locked multi-reader multi-writer queue

   :superclasses: :class:`<queue>`

   :discussion:

     Locked multi-reader multi-writer queue

     A notification is used for synchronization.
     The associated lock is used for all queue state.

     Locked queues can be :gf:`STOPPED <stop-queue>` so
     that no further work will be accepted and processing
     will end once all previously submitted work has been
     finished.

     After stopping, all further enqueue operations will
     signal :class:`<queue-stopped>`.

     Dequeue operations will continue until the queue has
     been drained, whereupon they will also be signalled.

     Locked queues can be :gf:`INTERRUPTED <interrupt-queue>`
     so that no further work will be accepted or begun. Work
     that has already been started will continue.

     Interrupting implies stopping, so :gf:`enqueue` operations
     will be signalled :class:`<queue-stopped>`.

     :gf:`Dequeue <dequeue>` operations will signal
     :class:`<queue-interrupt>`.

   :operations:

     * :gf:`interrupt-queue`
     * :gf:`stop-queue`

.. generic-function:: dequeue

   Dequeue the next available item from the queue.

   :signature: dequeue (queue) => (object)

   :parameter queue: An instance of :class:`<queue>`.
   :value object: An instance of :drm:`<object>`.

   :discussion:

     Dequeue the next available item from the queue.

     May signal :class:`<queue-interrupt>` or
     :class:`<queue-stopped>` when the queue has
     reached the respective state.

.. generic-function:: enqueue

   Enqueue a work item onto the queue.

   :signature: enqueue (queue object) => ()

   :parameter queue: An instance of :class:`<queue>`.
   :parameter object: An instance of :drm:`<object>`.

   :discussion:

     Enqueue a work item onto the queue.

     May signal :class:`<queue-stopped>` when
     the queue no longer accepts work.

.. generic-function:: queue-name

   Returns the name of the queue.

   :signature: queue-name (queue) => (name?)

   :parameter queue: An instance of :class:`<queue>`.
   :value name?: An instance of ``false-or(<string>)``.

.. generic-function:: interrupt-queue

   Interrupts the queue, abandoning submitted work.

   :signature: interrupt-queue (queue) => ()

   :parameter queue: An instance of :class:`<locked-queue>`.

   :discussion:

     Interrupts the queue, abandoning submitted work.

     Submitters will be signalled :class:`<queue-stopped>`
     in :gf:`enqueue` if they try to submit further work.

     Receivers will be signalled :class:`<queue-interrupt>`
     at the first :gf:`dequeue` operation they perform.

.. generic-function:: stop-queue

   Stops the queue so that submitted work can still continue.

   :signature: stop-queue (queue) => ()

   :parameter queue: An instance of :class:`<locked-queue>`.

   :discussion:

     Stops the queue so that submitted work can still continue.

     Submitters will be signalled :class:`<queue-stopped>`
     in :gf:`enqueue` if they try to submit further work.

     Receivers will be signalled :class:`<queue-stopped>`
     in :gf:`dequeue` once the queue has been drained.

.. class:: <queue-condition>
   :abstract:

   Conditions related to <locked-queue> operations.

   :superclasses: :drm:`<condition>`

   :keyword queue:
   :keyword thread:

.. class:: <queue-interrupt>

   Signalled when the queue has been interrupted.

   :superclasses: :class:`<queue-condition>`

.. class:: <queue-stopped>

   Signalled when the queue has been stopped.

   :superclasses: :class:`<queue-condition>`

.. generic-function:: queue-condition-queue

   :signature: queue-condition-queue (condition) => (queue)

   :parameter condition: An instance of :class:`<queue-condition>`.
   :value queue: An instance of :class:`<queue>`.

.. generic-function:: queue-condition-thread

   :signature: queue-condition-thread (condition) => (thread)

   :parameter condition: An instance of :class:`<queue-condition>`.
   :value thread: An instance of :drm:`<thread>`.


Work
----

.. class:: <work>

   :superclasses: :drm:`<object>`

   :keyword function: A function to perform some work.  The function must not have any required arguments.

   :operations:

   * :gf:`work-finished?`
   * :gf:`work-perform`
   * :gf:`work-started?`
   * :gf:`work-thread`

.. class:: <locked-work>

   :superclasses: :class:`<work>`

   :operations:

   * :gf:`work-wait`

.. generic-function:: work-finished?

   :signature: work-finished? (work) => (finished?)

   :parameter work: An instance of :class:`<work>`.
   :value finished?: An instance of :drm:`<boolean>`.

.. generic-function:: work-perform

   :signature: work-perform (work) => ()

   :parameter work: An instance of :class:`<work>`.

.. generic-function:: work-started?

   :signature: work-started? (work) => (started?)

   :parameter work: An instance of :class:`<work>`.
   :value started?: An instance of :drm:`<boolean>`.

.. generic-function:: work-thread

   Return the thread on which the work was executed.

   :signature: work-thread (work) => (thread)

   :parameter work: An instance of :class:`<work>`.
   :value thread: An instance of :class:`<thread>`.

.. generic-function:: work-wait

   Wait for a work item to reach the given state.  Valid states are
   :const:`$work-started` and :const:`$work-finished`.


   :signature: work-wait (work state) => ()

   :parameter work: An instance of :class:`<locked-work>`.
   :parameter state: An instance of :class:`<work-state>`. One of
      :const:`$work-started` or :const:`$work-finished`.

.. constant:: $work-started

   Used with :func:`work-wait` to indicate that you want to wait until
   work has started executing.

   :type: :class:`<work-state>`

   See also: :const:`$work-finished`

.. constant:: $work-finished

   Used with :func:`work-wait` to indicate that you want to wait until
   work has finished executing.

   :type: :class:`<work-state>`

   See also: :const:`$work-finished`
