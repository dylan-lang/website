A Simple Library
================

In this chapter, we create a complete library that represents time and
position. The ``timespace`` library provides the ``say`` generic function
for all concrete classes, the mathematical function ``+`` on certain kinds
of time, and the comparison functions ``<`` and ``=``, which enable users
to call all other numerical comparisons, ``>``, ``>=``, ``<=``, and ``~=``.
Our library consists of four files:

- The LID file lists all the files in the ``timespace`` library.
- The library file defines the ``timespace`` library and the ``timespace``
  module.
- The implementation file defines the classes, methods, and generic
  functions.
- The test file creates instances, calls ``say`` on them, and adds time
  instances.

We provide the test file so that you can experiment with the library.
Because the test code has a purpose different from that of the
implementation code, we separate them into two files. Normally, a
finished library would not contain both the implementation and the test
code — the test code would be in a separate library. However, when you
are starting to implement your program or library, it is convenient to
put all the code in one library, as we do here.

We shall continue to develop the time and position library in :doc:`part2`.
The complete version is given in :doc:`time-mod`.

The LID file
------------

The LID file: ``timespace.lid``.

.. code-block:: dylan

    library: timespace
    files: library
           library-implementation
           test

The library file
----------------

The library file: ``library.dylan``.

.. code-block:: dylan

    module: dylan-user

    define library timespace
      use dylan;
      use format-out;
    end library timespace;

    define module timespace
      use dylan;
      use format-out;
    end module timespace;

The implementation file
-----------------------

The implementation file: ``library-implementation.dylan``.

.. code-block:: dylan

    module: timespace

    // The sixty-unit class

    define abstract class <sixty-unit> (<object>)
      slot total-seconds :: <integer>, init-keyword: total-seconds:;
    end class <sixty-unit>;

    // decode-total-seconds

    define method decode-total-seconds
        (sixty-unit :: <sixty-unit>)
     => (max-unit :: <integer>, minutes :: <integer>, seconds :: <integer>)
      decode-total-seconds(abs(sixty-unit.total-seconds));
    end method decode-total-seconds;

    define method decode-total-seconds
        (total-seconds :: <integer>)
     => (hours :: <integer>, minutes :: <integer>, seconds :: <integer>)
      let (total-minutes, seconds) = truncate/(total-seconds, 60);
      let (hours, minutes) = truncate/(total-minutes, 60);
      values(hours, minutes, seconds);
    end method decode-total-seconds;

    // encode-total-seconds

    define method encode-total-seconds
        (max-unit :: <integer>, minutes :: <integer>, seconds :: <integer>)
     => (total-seconds :: <integer>)
      ((max-unit * 60) + minutes) * 60 + seconds;
    end method encode-total-seconds;

    // The say generic function

    // Given an object, print a description of the object
    define generic say (any-object :: <object>) => ();

    // The time classes and methods

    define abstract class <time> (<sixty-unit>)
    end class <time>;

    define method say (time :: <time>) => ()
      let (hours, minutes) = decode-total-seconds(time);
      format-out
        ("%d:%s%d", hours, if (minutes < 10) "0" else "" end, minutes);
    end method say;

    // A specific time of day from 00:00 (midnight) to before 24:00 (tomorrow)
    define class <time-of-day> (<time>)
    end class <time-of-day>;

    // A relative time between -24:00 and +24:00
    define class <time-offset> (<time>)
    end class <time-offset>;

    // Method for determining whether a time offset is in the past
    define method past? (time :: <time-offset>) => (past? :: <boolean>)
      time.total-seconds < 0;
    end method past?;

    define method say (time :: <time-offset>) => ()
      format-out("%s ", if (past?(time)) "minus" else "plus" end);
      next-method();
    end method say;

    // Methods for adding times

    define method \+
        (offset1 :: <time-offset>, offset2 :: <time-offset>)
     => (sum :: <time-offset>)
      let sum = offset1.total-seconds + offset2.total-seconds;
      make(<time-offset>, total-seconds: sum);
    end method \+;

    define method \+
        (offset :: <time-offset>, time-of-day :: <time-of-day>)
     => (sum :: <time-of-day>)
      make(<time-of-day>,
      total-seconds: offset.total-seconds + time-of-day.total-seconds);
    end method \+;

    define method \+
        (time-of-day :: <time-of-day>, offset :: <time-offset>)
     => (sum :: <time-of-day>)
      offset + time-of-day;
    end method \+;

    // Methods for comparing times

    define method \< (time1 :: <time-of-day>, time2 :: <time-of-day>)
     => (boolean :: <boolean>)
      time1.total-seconds < time2.total-seconds;
    end method \<;

    define method \< (time1 :: <time-offset>, time2 :: <time-offset>)
     => (boolean :: <boolean>)
      time1.total-seconds < time2.total-seconds;
    end method \<;

    define method \= (time1 :: <time-of-day>, time2 :: <time-of-day>)
     => (boolean :: <boolean>)
      time1.total-seconds = time2.total-seconds;
    end method \=;

    define method \= (time1 :: <time-offset>, time2 :: <time-offset>)
     => (boolean :: <boolean>)
      time1.total-seconds = time2.total-seconds;
    end method \=;

    // The angle classes and methods

    define abstract class <angle> (<sixty-unit>)
    end class <angle>;

    define method say (angle :: <angle>) => ()
      let (degrees, minutes, seconds) = decode-total-seconds(angle);
      format-out
        ("%d degrees %d minutes %d seconds",
         degrees, minutes, seconds);
    end method say;

    define class <relative-angle> (<angle>)
    end class <relative-angle>;

    // We need to show degrees for <relative-angle> but we do not need to
    // show minutes and seconds, so we override the method on <angle>
    define method say (angle :: <relative-angle>) => ()
      format-out("%d degrees", decode-total-seconds(angle));
    end method say;

    define abstract class <directed-angle> (<angle>)
      slot direction :: <string>, init-keyword: direction:;
    end class <directed-angle>;

    define method say (angle :: <directed-angle>) => ()
      next-method();
      format-out(" %s", angle.direction);
    end method say;

    // The latitude and longitude classes and methods

    define class <latitude> (<directed-angle>)
    end class <latitude>;

    define method say (latitude :: <latitude>) => ()
      next-method();
      format-out(" latitude\n");
    end method say;

    define class <longitude> (<directed-angle>)
    end class <longitude>;

    define method say (longitude :: <longitude>) => ()
      next-method();
      format-out(" longitude\n");
    end method say;

    // The position classes and methods

    define abstract class <position> (<object>)
    end class <position>;

    define class <absolute-position> (<position>)
      slot latitude :: <latitude>, init-keyword: latitude:;
      slot longitude :: <longitude>, init-keyword: longitude:;
    end class <absolute-position>;

    define method say (position :: <absolute-position>) => ()
      say(position.latitude);
      say(position.longitude);
    end method say;

    define class <relative-position> (<position>)
      // Distance is in miles
      slot distance :: <single-float>, init-keyword: distance:;
      slot angle :: <angle>, init-keyword: angle:;
    end class <relative-position>;

    define method say (position :: <relative-position>) => ()
      format-out("%d miles away at heading ", position.distance);
      say(position.angle);
    end method say;

The test file
-------------

The test file: ``test.dylan``.

.. code-block:: dylan

    module: timespace

    format-out("Creating an instance of <absolute-position>:\n");

    define variable *my-absolute-position*
      = make(<absolute-position>,
             latitude: make(<latitude>,
                            total-seconds: encode-total-seconds(42, 19, 34),
                            direction: "North"),
             longitude: make(<longitude>,
                             total-seconds: encode-total-seconds(70, 56, 26),
                             direction: "West"));

    say(*my-absolute-position*);

    format-out("\n");

    format-out("Creating an instance of <relative-position>:\n");

    define variable *her-relative-position*
      = make(<relative-position>,
             distance: 30.0,
             angle: make(<relative-angle>,
                         total-seconds: encode-total-seconds(90, 5, 0)));

    say(*her-relative-position*);

    format-out("\n");

    format-out("Creating an instance of <time-offset> in *minus-2-hours*.\n");

    define variable *minus-2-hours*
      = make(<time-offset>, total-seconds: - encode-total-seconds (2, 0, 0));

    format-out("Creating an instance of <time-offset> in *plus-15-20-45*.\n");

    define variable *plus-15-20-45*
      = make(<time-offset>, total-seconds: encode-total-seconds (15, 20, 45));

    format-out("Creating an instance of <time-of-day> in *8-30-59*.\n");

    define variable *8-30-59*
      = make(<time-of-day>, total-seconds: encode-total-seconds (8, 30, 59));

    format-out("Adding <time-offset> + <time-offset>: *minus-2-hours* + *plus-15-20-45*:\n");

    decode-total-seconds(*minus-2-hours* + *plus-15-20-45*);

    format-out("Adding <time-offset> + <time-of-day>: *minus-2-hours* + *8-30-59*:\n");

    decode-total-seconds(*minus-2-hours* + *8-30-59*);

    format-out("Adding <time-of-day> + <time-offset>: *8-30-59* + *minus-2-hours*:\n");

    decode-total-seconds(*8-30-59* + *minus-2-hours*);

When we run the test file, we see the following output and values::

    Creating an instance of <absolute-position>:
    42 degrees 19 minutes 34 seconds North latitude
    70 degrees 56 minutes 26 seconds West longitude

    Creating an instance of <relative-position>:
    30.000000 miles away at heading 90 degrees

    Creating an instance of <time-offset> in *minus-2-hours*.
    Creating an instance of <time-offset> in *plus-15-20-45*.
    Creating an instance of <time-of-day> in *8-30-59*.
    Adding <time-offset> + <time-offset>: *minus-2-hours* + *plus-15-20-45*:
    13
    20
    45
    Adding <time-offset> + <time-of-day>: *minus-2-hours* + *8-30-59*:
    6
    30
    59
    Adding <time-of-day> + <time-offset>: *8-30-59* + *minus-2-hours*:
    6
    30
    59

Summary
-------

In this chapter, we created the four files that constitute the
``timespace`` library.
