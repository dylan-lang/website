Testworks Usage
***************

.. current-library:: testworks
.. current-module:: testworks

Testworks is a Dylan unit testing library.

See also: :doc:`reference`

Quick Start
===========

For the impatient (i.e., most of us), this section summarizes most of what you
need to know to use Testworks.

Add ``use testworks;`` to both your test library and test module.

Tests contain arbitrary code and at least one assertion or expectation.

**Terminology:** An assertion failure terminates the test and skips any further
assertions or expectations.  (This is normally preferred since it prevents
cascading failures.)  An expectation failure does not terminate the
test. Assertions are Testworks macros that begin with ``assert-`` and
expectations begin with ``expect-``. When this documentation refers to
"assertions" it should be understood to include both kinds of macros.

Example test:

.. code-block:: dylan

   define test test-my-function ()
     let v = do-something();
     assert-equal("expected-value", my-function(v));
     assert-condition(<error>, my-function(v, key: 7), "regression test for bug 12345");
   end test;

If there are no assertions in a test it is considered "NOT IMPLEMENTED", which
is displayed in the output (as a reminder to implement it) but is not
considered a failure.

See also: `Assertions`_ for other assertion macros.

Benchmarks do not require any assertions and are automatically given the
"benchmark" tag:

.. code-block:: dylan

   define benchmark fn1-benchmark ()
     fn1()
   end;

See also, :macro:`benchmark-repeat`.

If you have a large or complex test library, "suites" may be used to organize
tests into groups (for example one suite per module) and may be nested
arbitrarily. This does impose a small maintenance burden of adding each test to
a suite, and is not required. See `Suites`_ for details.

.. code-block:: dylan

   define suite my-library-suite ()
     suite module1-suite;
     suite module2-suite;
     test some-other-test;
   end;

.. note:: Suites must be defined textually *after* the other suites and tests
          they contain.

To run your tests of course you need an executable and there are two ways to
accomplish this:

#.  Have your library call :func:`run-test-application` and compile it as an
    executable. With no arguments :func:`run-test-application` runs all tests
    and benchmarks, as filtered by the Testworks command-line options.

    If you prefer to manually organize your tests with suites, pass your
    top-level suite to :func:`run-test-application` and that determines the
    initial set of tests that are filtered by the command line.

    .. note:: If you forget to add a test to any suite, the test will not be
              run.

#.  Compile your test library as a shared library and run it with the
    ``testworks-run`` application. For example, for the `foo-test` library::

      _build/bin/testworks-run --load libfoo-test.so

In both cases :func:`run-test-application` parses the command line so the
options are the same. Use ``--help`` to see all options.

Defining Tests
==============

Assertions
----------

Assertions come in two forms: ``assert-*`` macros and ``expect-*`` macros.
When an ``assert-*`` macro fails, it causes the test to fail and the remainder
of the test to be skipped. This kind of assertion is generally preferred
because it prevents "cascading failures". In contrast, when an ``expect-*``
macro fails, it causes a test failure but the test continues running, so there
may be multiple assertion failures recorded for the test.

.. note:: You may also find ``check-*`` macros in existing test suites.  These
          are a deprecated form of assertion, replaced by ``expect-*``.

The benefit of the ``expect-*`` macros is that when you're initially debugging
the test it may require fewer iterations if you can see multiple failures in
one pass.

An assertion accepts an expression to evaluate and report back on, saying if
the expression passed, failed, or crashed (i.e., signaled an error).  As an
example, in

.. code-block:: dylan

    assert-true(foo > bar)
    expect(foo > bar)

the expression ``foo > bar`` is compared to ``#f``, and the result is recorded
by the test harness.

See the :doc:`reference` for detailed documentation on the
available assertion macros.

Each assertion macro accepts an optional description (a format string and
optional format arguments), after the required arguments, which is displayed if
the assertion fails.  If the description isn't provided, Testworks makes one
from the expressions passed to the assertion macro. For example,
``assert-true(2 > 3)`` produces this failure message::

  FAILED: (2 > 3)
    expression "(2 > 3)" evaluates to #f

In general, Testworks should be pretty good at reporting the actual values that
caused the failure so it shouldn't be necessary to include them in the
description all the time. Usually if your test iterates over various inputs
it's a good idea to provide a description so the failing input can be easily
identified.

If you do provide a description it may either be a single value to display, as
with ``format-to-string("%s", v)``, or a format string and corresponding format
arguments. These are all valid:

.. code-block:: dylan

   assert-equal(want, got);       // auto-generated description
   assert-equal(want, got, foo);  // foo used as description
   assert-equal(want, got, "does %= = %=?", a, b);  // formatted description

Tests
-----

Tests contain assertions and arbitrary code needed to support those
assertions. Each test may be part of a suite.  Use the
:macro:`test-definer` macro to define a test:

.. code-block:: dylan

    define test NAME (#key EXPECTED-FAILURE?, TAGS)
      BODY
    end;

For example:

.. code-block:: dylan

    define test my-test ()
      assert-equal(2, 3);
      assert-equal(#f, #f);
      assert-true(identity(#t), "Check identity function");
    end;

The result looks like this::

    $ _build/bin/my-test-suite
    Running suite my-test-suite:
    Running test my-test:
      2 = 3: [2 and 3 are not =.]
       FAILED in 0.000257s and 17KiB

    my-test failed
      #f = #f passed
      2 = 3 failed [2 and 3 are not =.]
    Ran 2 checks: FAILED (1 failed)
    Ran 1 test: FAILED (1 failed)
    FAILED in 0.000257 seconds

Note that the third assertion was not executed since the second one failed and
terminated ``my-test``.

Tests may be tagged with arbitrary strings, providing a way to select
or filter out tests to run:

.. code-block:: dylan

    define test my-test-2 (tags: #["huge"])
      ...huge test that takes a long time...
    end test;

    define test my-test-3 (tags: #["huge", "verbose"])
      ...test with lots of output...
    end test;

Tags can then be passed on the Testworks command-line.  For example,
this skips both of the above tests::

    $ _build/bin/my-test-suite-app --tag=-huge --tag=-verbose

Negative tags take precedence, so ``--tag=huge --tag=-verbose`` runs
``my-test-2`` and skips ``my-test-3``.

If the test is expected to fail, or fails under some conditions, Testworks
can be made aware of this:

.. code-block:: dylan

    define test failing-test
        (expected-to-fail-reason: "bug 1234")
      assert-true(#f);
    end test;

    define test fails-on-windows
        (expected-to-fail?: method () $os-name = #"win32" end,
         expected-to-fail-reason: "blah is not implemented for WIN32 platform")
      if ($os-name = #"win32")
        assert-false(#t);
      else
        assert-true(#t);
      end if;
    end test;

A test that is expected to fail and then fails is considered to be a passing
test. If the test succeeds unexpectedly, it is considered a failing test. When
marking a test as expected to fail, ``expected-to-fail-reason:`` is
**required** and ``expected-to-fail?:`` is optional, and normally
unnecessary. An example of a good reason is a bug URL or other bug reference.

.. note:: When providing a value for ``expected-to-fail?:`` always provide a
          method of no arguments. For example, instead of ``expected-to-fail?:
          $os-name == #"win32"`` use ``expected-to-fail?: method () $os-name ==
          #"win32" end``. The former is equivalent to ``expected-to-fail?: #f``
          on non-Windows platforms and results in an ``UNEXPECTED SUCCESS``
          result. This is because the (required) reason string is used as
          shorthand to indicate that failure is expected even when
          ``expected-to-fail?:`` is ``#f``.

Test setup and teardown is accomplished with normal Dylan code using
``block () ... cleanup ... end;``...

.. code-block:: dylan

   define test foo ()
     block ()
       do-setup-stuff();
       assert-equal(...);
       assert-equal(...);
     cleanup
       do-teardown-stuff()
     end
   end;

Benchmarks
----------

Benchmarks are like tests except for:

* They do not require any assertions. (They pass unless they signal an error.)
* They are automatically assigned the "benchmark" tag.

The :macro:`benchmark-definer` macro is like :macro:`test-definer`:

.. code-block:: dylan

   define benchmark my-benchmark ()
     ...body...
   end;

Benchmarks may be added to suites:

.. code-block:: dylan

   define suite my-benchmarks-suite ()
     benchmark my-benchmark;
   end;

Benchmarks and tests may be combined in the same suite, but this is
discouraged. It is preferable to have separate libraries for the two since
benchmarks often take longer to run and may not necessarily need to be run for
every commit.

See also, :macro:`benchmark-repeat`.

Suites
------

Suites are an optional feature that may be used to organize your tests
into a hierarchy.  Suites contain tests, benchmarks, and other
suites. A suite is defined with the :macro:`suite-definer` macro.  The
format is:

.. code-block:: dylan

    define suite NAME (#key setup-function, cleanup-function)
        test TEST-NAME;
        benchmark BENCHMARK-NAME;
        suite SUITE-NAME;
    end;

For example:

.. code-block:: dylan

    define suite first-suite ()
      test my-test;
      test example-test;
      test my-test-2;
    end;

    define suite second-suite ()
      suite first-suite;
      test my-test;
    end;

Suites can specify setup and cleanup functions via the keyword
arguments ``setup-function`` and ``cleanup-function``. These can be
used for things like establishing database connections, initializing
sockets and so on.

A simple example of doing this can be seen in the http-server test
suite:

.. code-block:: dylan

    define suite http-test-suite (setup-function: start-sockets)
      suite http-server-test-suite;
      suite http-client-test-suite;
    end;

Interface Specification Suites
------------------------------

The :macro:`interface-specification-suite-definer` macro creates a normal test
suite, much like ``define suite`` does, but based on an interface
specification. For example,

.. code-block:: dylan

   define interface-specification-suite time-specification-suite ()
     sealed instantiable class <time> (<object>);
     constant $utc :: <zone>;
     variable *zone* :: <zone>;
     sealed generic function in-zone (<time>, <zone>) => (<time>);
     function now (#"key", #"zone") => (<time>);
     ...
   end;

The specification usually has one clause, or "spec", for each name exported
from your public interface module. Each spec creates a test named
``test-{name}-specification`` to verify that the implementation matches the
spec for ``{name}``. For example, by checking that the names are bound, that
their bindings have the correct types, that functions accept the right number
and types of arguments, etc.

Specification suites are otherwise just normal suites. They may include other
arbitrary tests and child suites if desired:

.. code-block:: dylan

   define interface-specification-suite time-suite ()
     ...
     test test-time-still-moving-forward;
     suite time-travel-test-suite;
   end;

This also means that if your interface is large you may use multiple
:macro:`interface-specification-suite-definer` forms and then group them
together.

See :macro:`interface-specification-suite-definer` for more details on the
various kinds of specs.


Organizing Tests for One Library
================================

If you don't use suites, the only organization you need is to name
your tests and benchmarks uniquely, and you can safely skip the rest
of this section.  If you do use suites, read on....

Tests are used to combine related assertions into a unit, and suites
further organize related tests and benchmarks.  Suites may also
contain other suites.

It is common for the test suite for library xxx to export a single
test suite named xxx-test-suite, which is further subdivided into
sub-suites, tests, and benchmarks as appropriate for that library.
Some suites may be exported so that they can be included as a
component suite in combined test suites that cover multiple related
libraries. (The alternative to this approach is running each library's
tests as a separate executable.)

.. note:: It is an error for a test to be included in a suite multiple times,
          even transitively. Doing so would result in a misleading pass/fail
          ratio, and it is more likely to be a mistake than to be intentional.

The overall structure of a test library that is intended to be
included in a combined test library may look something like this:

.. code-block:: dylan

    // --- library.dylan ---

    define library xxx-tests
      use common-dylan;
      use testworks;
      use xxx;                 // the library you are testing
      export xxx-tests;        // so other test libs can include it
    end;

    define module xxx-tests
      use common-dylan;
      use testworks;
      use xxx;                 // the module you are testing
      export xxx-test-suite;   // so other suites can include it
    end;

    // --- main.dylan ---

    define test my-awesome-test ()
      assert-true(...);
      assert-equal(...);
      ...
    end;

    define benchmark my-awesome-benchmark ()
      awesomely-slow-function();
    end;

    define suite xxx-test-suite ()
      test my-awesome-test;
      benchmark my-awesome-benchmark;
      suite my-awesome-other-suite;
      ...
    end;

Running Tests As A Stand-alone Application
==========================================

If you don't need to export any suites so they can be included in a
higher-level combined test suite library (i.e., if you're happy
running your test suite library as an executable) then you can simply
call :func:`run-test-application` to parse the standard testworks
command-line options and run the specified tests::

  run-test-application();

and you can skip the rest of this section.

If you need to export a suite for use by another library, then you must also
define a separate executable library, traditionally named "xxx-test-suite-app",
which calls ``run-test-application()``.

Here's an example of such an application library:

1. The file ``library.dylan`` which must use at least the library that
exports the test suite, and ``testworks``:

.. code-block:: dylan

    Module:    dylan-user
    Synopsis:  An application library for xxx-test-suite

    define library xxx-test-suite-app
      use xxx-test-suite;
      use testworks;
    end;

    define module xxx-test-suite-app
      use xxx-test-suite;
      use testworks;
    end;

2. The file ``xxx-test-suite-app.dylan`` which simply contains a call to the
   method :func:`run-test-application`:

.. code-block:: dylan

    Module: xxx-test-suite-app

    run-test-application();

3. The file ``xxx-test-suite-app.lid`` which specifies the names of the source
   files:

.. code-block:: dylan

    Library: xxx-test-suite-app
    Target-type: executable
    Files: library.dylan
           xxx-test-suite-app.dylan

Once a library has been defined in this fashion it can be compiled
into an executable with ``dylan-compiler -build
xxx-test-suite-app.lid`` and run with ``xxx-test-suite-app --help``.


Reports
=======

The ``--report`` and ``--report-file`` options can be used to write a full
report of test run results so that those results can be compared with
subsequent test runs, for example to find regressions. These are the available
report types:

failures (the default)
  Prints out only the list of failures and a summary, in readable text format.

full
  Like ``failures`` but prints results whether passing or failing.

json
  Outputs JSON objects that match the suite/test/assertion tree structure,
  with full detail.

summary
  Prints out only a summary of how many assertions, tests and suites
  were executed, passed, failed or crashed.

surefire
  Outputs XML in Surefire format.  This elides information about specific
  assertions.  This format is supported by various tools such as Jenkins.

xml
  Outputs XML that directly matches the suite/test/assertion tree structure,
  with full detail.


Comparing Test Results
======================

*** To be filled in ***

Quick version:

*  (master branch)$ my-test-suite --report json --report-file out1.json
*  (your branch)$ my-test-suite --report json --report-file out2.json
*  $ testworks-report out1.json out2.json
