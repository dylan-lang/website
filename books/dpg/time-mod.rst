Four Complete Libraries
=======================

In this chapter, we show all the files that the complete ``time``,
``angle``, ``sixty-unit``, and ``say`` libraries comprise.

The ``sixty-unit`` library
--------------------------

The ``sixty-unit`` library is an example of a shared substrate library.
Both the ``time`` and ``angle`` libraries use the ``sixty-unit`` library to
create more specialized classes that build on a common substrate.

The ``sixty-unit`` library comprises two Dylan interchange-format files: a
library file, containing the library and module definitions; and an
implementation file, containing a single source record, defining the
generic function that is the ``say`` protocol. For completeness, we also
show the LID file that describes the library and its component files.

The ``sixty-unit-library`` file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``sixty-unit-library`` file: ``sixty-unit-library.dylan``.

.. code-block:: dylan

    Module: dylan-user

    // Library definition
    define library sixty-unit
      // Interface module
      export sixty-unit;
      // Substrate library
      use dylan;
    end library sixty-unit;

    // Interface module
    define module sixty-unit
      // Classes
      create <sixty-unit>;
      // Generics
      create total-seconds, encode-total-seconds, decode-total-seconds;
    end module sixty-unit;

    // Implementation module
    define module sixty-unit-implementation
      // External interface
      use sixty-unit;
      // Substrate module
      use dylan;
    end module sixty-unit-implementation;

The ``sixty-unit`` implementation file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``sixty-unit`` implementation file: ``sixty-unit.dylan``.

.. code-block:: dylan

    Module: sixty-unit-implementation

    define open abstract class <sixty-unit> (<object>)
      slot total-seconds :: <integer>, required-init-keyword: total-seconds:;
    end class <sixty-unit>;

    define method encode-total-seconds
        (max-unit :: <integer>, minutes :: <integer>, seconds :: <integer>)
     => (total-seconds :: <integer>)
      ((max-unit * 60) + minutes) * 60 + seconds;
    end method encode-total-seconds;

    define method decode-total-seconds
        (sixty-unit :: <sixty-unit>)
     => (max-unit :: <integer>, minutes :: <integer>, seconds :: <integer>)
      decode-total-seconds(sixty-unit.total-seconds);
    end method decode-total-seconds;

    define method decode-total-seconds
        (total-seconds :: <integer>)
     => (max-unit :: <integer>, minutes :: <integer>, seconds :: <integer>)
      let (total-minutes, seconds) = truncate/(abs(total-seconds), 60);
      let (max-unit, minutes) = truncate/(total-minutes, 60);
      values(max-unit, minutes, seconds);
    end method decode-total-seconds;

The ``sixty-unit`` LID file
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The LID file: ``sixty-unit.lid``.

.. code-block:: dylan

    library: sixty-unit
    files: sixty-unit-library
           sixty-unit

The ``say`` library
-------------------

The ``say`` library is an example of a library that defines a shared
protocol. All our other libraries use the ``say`` library, so that they
can add to the ``say`` generic function methods that appropriately display
the objects of the classes that they define.

The ``say`` library comprises two Dylan interchange-format files: a
library file, containing the library and module definitions; and an
implementation file, containing a single source record, defining the
generic function that is the ``say`` protocol. For completeness, we also
show the LID file that describes the library and its component files.

The ``say-library`` file
~~~~~~~~~~~~~~~~~~~~~~~~

The ``say-library`` file: ``say-library.dylan``.

.. code-block:: dylan

    Module: dylan-user

    // Library definition
    define library say
      // Interface modules
      export say, say-implementor;
      // Substrate libraries
      use format-out;
      use dylan;
    end library say;

    // Protocol interface
    define module say
      create say;
    end module say;

    // Implementor interface
    define module say-implementor
      use say, export: all;
      use format-out, export: all;
    end module say-implementor;

    // Implementation module
    define module say-implementation
      use say;
      use dylan;
    end module say-implementation;

The ``say`` implementation file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``say`` implementation file: ``say.dylan``.

.. code-block:: dylan

    Module: say-implementation

    define open generic say (object :: <object>) => ();

The ``say`` LID file
~~~~~~~~~~~~~~~~~~~~

The LID file: ``say.lid``.

.. code-block:: dylan

    library: say
    files: say-library
           say

The ``time`` library
--------------------

The ``time`` library is a client of the ``sixty-unit`` and ``say`` libraries,
and it will serve as a substrate library for the rest of our
application. Like the previous two libraries, it comprises a library
file and an implementation file; we also show the corresponding LID
file.

The ``time-library`` file
~~~~~~~~~~~~~~~~~~~~~~~~~

The ``time-library`` file: ``time-library.dylan``.

.. code-block:: dylan

    Module: dylan-user

    // Library definition
    define library time
      // Interface module
      export time;
      // Substrate libraries
      use sixty-unit;
      use say;
      use dylan;
    end library time;

    // Interface module
    define module time
      // Classes
      create <time>, <time-of-day>, <time-offset>;
      // Types
      create <nonnegative-integer>;
      // Constants
      create $midnight, $tomorrow;
      // Shared protocol
      use say, export: all;
      use sixty-unit, import: { encode-total-seconds }, export: all;
    end module time;

    // Implementation module
    define module time-implementation
      // External interface
      use time;
      // Substrate modules
      use sixty-unit;
      use say-implementor;
      use dylan;
    end module time-implementation;

.. _time-mod-time-implementation-file:

The ``time`` implementation file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``time`` implementation file: ``time.dylan``.

.. code-block:: dylan

    Module: time-implementation

    // Define nonnegative integers as integers that are >= zero
    define constant <nonnegative-integer> = limited(<integer>, min: 0);

    define abstract class <time> (<sixty-unit>)
    end class <time>;

    define method say (time :: <time>) => ()
      let (hours, minutes) = decode-total-seconds(time);
      format-out("%d:%s%d",
                 hours, if (minutes < 10) "0" else " " end, minutes);
    end method say;

    // A specific time of day from 00:00 (midnight) to before 24:00 (tomorrow)
    define class <time-of-day> (<time>)
    end class <time-of-day>;

    define method total-seconds-setter
        (total-seconds :: <integer>, time :: <time-of-day>)
     => (total-seconds :: <nonnegative-integer>)
      if (total-seconds >= 0)
        next-method();
      else
        error("%d cannot be negative", total-seconds);
      end if;
    end method total-seconds-setter;

    define method initialize (time :: <time-of-day>, #key)
      next-method();
      if (time.total-seconds < 0)
        error("%d cannot be negative", time.total-seconds);
      end if;
    end method initialize;

    // A relative time between -24:00 and +24:00
    define class <time-offset> (<time>)
    end class <time-offset>;

    define method past? (time :: <time-offset>) => (past? :: <boolean>)
      time.total-seconds < 0;
    end method past?;

    define method say (time :: <time-offset>) => ()
      format-out("%s ", if (time.past?) "minus" else "plus" end);
      next-method();
    end method say;

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

    define method \+ (time-of-day :: <time-of-day>, offset :: <time-offset>)
     => (sum :: <time-of-day>)
      offset + time-of-day;
    end method \+;

    define method \< (time1 :: <time-of-day>, time2 :: <time-of-day>)
      time1.total-seconds < time2.total-seconds;
    end method \<;

    define method \< (time1 :: <time-offset>, time2 :: <time-offset>)
      time1.total-seconds < time2.total-seconds;
    end method \<;

    define method \= (time1 :: <time-of-day>, time2 :: <time-of-day>)
      time1.total-seconds = time2.total-seconds;
    end method \=;

    define method \= (time1 :: <time-offset>, time2 :: <time-offset>)
      time1.total-seconds = time2.total-seconds;
    end method \=;

    // Two useful time constants
    define constant $midnight
      = make(<time-of-day>, total-seconds: encode-total-seconds(0, 0, 0));

    define constant $tomorrow
      = make(<time-of-day>,
             total-seconds: encode-total-seconds(24, 0, 0));

The ``time`` LID file
~~~~~~~~~~~~~~~~~~~~~

The LID file: ``time.lid``.

.. code-block:: dylan

    library: time
    files: time-library
           time

The ``angle`` library
---------------------

The ``angle`` library is the second client of the ``sixty-unit`` substrate.
The ``angle`` library extends the ``say`` protocol to handle objects of the
classes that it defines, such as ``<latitude>``, ``<longitude>``, and
``<absolute-position>``. For the time being, we have included positions
with angles, as we do not foresee any benefit to breaking them out into
yet another library, at least for the current application. Nevertheless,
we have defined separate interface and implementation modules for
positions, and we have broken out the position source records into a
separate interchange file.

Like with the ``time`` library, the ``angle`` library file does not have to
specify the use of the ``format-out`` library. It will be transitively
included because it is exported by the ``say`` library. Similarly, clients
of the ``angle`` library do not need to know anything about the ``say`` and
``sixty-unit`` libraries, since those libraries are imported and
re-exported to clients of ``angle``.

Note that the ``position-implementation`` module uses the ``angle`` module —
it is an internal client of the ``angle`` module. This structure means
that we can easily break out positions as a separate library, should the
need arise.

Also note that we have used the ``angle`` interface module to enforce
access control on the ``internal-direction`` slot. It should be accessed
only through the ``direction`` and ``direction-setter`` methods, which
ensure that valid values are used for our ``<latitude>`` and ``<longitude>``
classes. Because only the approved generic functions are created in the
interface module, only they will be accessible to clients of the ``angle``
library. The ``internal-direction`` slot is truly internal to the ``angle``
library — no client library can even determine its existence.

The ``angle-library`` file
~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``angle-library`` file: ``angle-library.dylan``.

.. code-block:: dylan

   Module: dylan-user

   // Library definition
   define library angle
     // Interface module
     export angle, position;
     // Substrate libraries
     use sixty-unit;
     use say;
     use dylan;
   end library angle;

   // Interface module
   define module angle
     // Classes
     create <angle>, <relative-angle>, <directed-angle>, <latitude>, <longitude>;
     // Generics
     create direction, direction-setter;
     // Shared protocol
     use say, export: all;
     use sixty-unit, import: { encode-total-seconds }, export: all;
   end module angle;

   // Interface module
   define module position
     // Classes
     create <position>, <absolute-position>, <relative-position>;
     // Generics
     create distance, angle, latitude, longitude;
     // Shared protocol
     use say, export: all;
   end module position;

   // Implementation module
   define module angle-implementation
     // External interface
     use angle;
     // Substrate modules
     use sixty-unit;
     use say-implementor;
     use dylan;
   end module angle-implementation;

   // Implementation module
   define module position-implementation
     // External interface
     use position;
     // Substrate modules
     use angle;
     use say-implementor;
     use dylan;
   end module position-implementation;

The ``angle`` implementation file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``angle`` implementation file is simply a collection of the source
records that we developed earlier for creating and saying angles,
latitudes, and longitudes.

The ``angle`` implementation file: ``angle.dylan``.

.. code-block:: dylan

    Module: angle-implementation

    define abstract class <angle> (<sixty-unit>)
    end class <angle>;

    define method say (angle :: <angle>) => ()
      let (degrees, minutes, seconds) = decode-total-seconds(angle);
      format-out("%d degrees %d minutes %d seconds",
                 degrees, minutes, seconds);
    end method say;

    define class <relative-angle> (<angle>)
    end class <relative-angle>;

    define method say (angle :: <relative-angle>) => ()
      format-out(" %d degrees", decode-total-seconds(angle));
    end method say;

    define abstract class <directed-angle> (<angle>)
      virtual slot direction :: <symbol>;
      slot internal-direction :: <symbol>;
      keyword direction:;
    end class <directed-angle>;

    define method initialize (angle :: <directed-angle>, #key direction: dir)
      next-method();
      angle.direction := dir;
    end method initialize;

    define method direction (angle :: <directed-angle>) => (dir :: <symbol>)
      angle.internal-direction;
    end method direction;

    define method direction-setter
        (dir :: <symbol>, angle :: <directed-angle>) => (new-dir :: <symbol>)
      angle.internal-direction := dir;
    end method direction-setter;

    define method say (angle :: <directed-angle>) => ()
      next-method();
      format-out(" %s", angle.direction);
    end method say;

    define class <latitude> (<directed-angle>)
    end class <latitude>;

    define method say (latitude :: <latitude>) => ()
      next-method();
      format-out(" latitude\n");
    end method say;

    define method direction-setter
        (dir :: <symbol>, latitude :: <latitude>) => (new-dir :: <symbol>)
      if (dir == #"north" | dir == #"south")
        next-method();
      else
        error("%= is not north or south", dir);
      end if;
    end method direction-setter;

    define class <longitude> (<directed-angle>)
    end class <longitude>;

    define method say (longitude :: <longitude>) => ()
      next-method();
      format-out(" longitude\n");
    end method say;

    define method direction-setter
        (dir :: <symbol>, longitude :: <longitude>) => (new-dir :: <symbol>)
      if (dir == #"east" | dir == #"west")
        next-method();
      else
        error("%= is not east or west", dir);
      end if;
    end method direction-setter;

The ``position`` implementation file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``position`` implementation file is simply a collection of the source
records that we developed earlier for creating and saying absolute and
relative positions.

The ``position`` implementation file: ``position.dylan``.

.. code-block:: dylan

    Module: position-implementation

    define abstract class <position> (<object>)
    end class <position>;

    define class <absolute-position> (<position>)
      slot latitude :: <latitude>, required-init-keyword: latitude:;
      slot longitude :: <longitude>, required-init-keyword: longitude:;
    end class <absolute-position>;

    define method say (position :: <absolute-position>) => ()
      say(position.latitude);
      say(position.longitude);
    end method say;

    define class <relative-position> (<position>)
      // Distance is in miles
      slot distance :: <single-float>, required-init-keyword: distance:;
      // Angle is in degrees
      slot angle :: <angle>, required-init-keyword: angle:;
    end class <relative-position>;

    define method say (position :: <relative-position>) => ()
      format-out("%s miles away at heading ", position.distance);
      say(position.angle);
    end method say;

The ``angle`` LID file
~~~~~~~~~~~~~~~~~~~~~~

Because we have chosen to put the source records for positions in a
separate interchange file, the LID file lists three Dylan files that
make up the ``angle`` library.

The LID file: ``angle.lid``.

.. code-block:: dylan

    library: angle
    files: angle-library
           angle
           position

Summary
-------

The structure of protocol and substrate libraries that we have created
is perhaps overly complex for the simple functionality that we have
implemented here. However, the libraries illustrate the power of the
Dylan module and library system to modularize large projects into easily
manageable sub-projects, and to control the interfaces among those
projects.
