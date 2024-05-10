User-Defined Classes and Methods
================================

In this chapter, we show the most basic techniques for writing
object-oriented code in Dylan. We define a class, make instances of the
class, initialize slots of the instances, and get and set the values of
slots. We define methods, and call them on the instances. One method
returns multiple values — and that is an extremely useful technique.
Another method uses local variables.

In this chapter, we start to develop an example of a library that
represents different kinds of time. A library is a complete unit of code
that can be used by many different clients, where a client can be
another library or an application program. In Chapters
:doc:`offset` and :doc:`multi`, we expand and refine
the example that we begin in this chapter. :doc:`time-code`,
shows the result: a complete and working library.

Requirements of the time and position classes and methods
---------------------------------------------------------

Our eventual goal in this book is to develop a sample application based
on an airport theme. The sample application handles the scheduling of
aircraft that are arriving into and departing from an airport. For more
information, see :doc:`design`.

We know that, for our airport application, we need to represent time.
There are several ways to represent time. We could say that an event
happened 2 hours ago (a time offset). We could say that an event
happened at 21:30 (a time of day). We must represent both kinds of time
in our time library, and we must provide a way to print representations
of both. In this chapter, we define a class named ``<time-of-day>``, and
we define a method that prints a representation of ``<time-of-day>``. In
:doc:`offset`, we define the ``<time-offset>`` class, and a
method that prints a representation of ``<time-offset>``.

The airport application also requires us to represent physical objects
(such as aircraft), and the positions (locations) of physical objects.
In :doc:`modularity`, we define classes that represent physical objects and
positions.

Eventually, we need to be able to add times, to compare times for
similarity, and to determine which of two times is greater than the
other. We implement those operations in :doc:`multi`.

We package the result of all our work into a complete and working
library, in :doc:`time-code`. Later, we refine this library to
achieve greater modularity and extensibility. The final result is given
in :doc:`time-mod`.

User-defined classes
--------------------

A *user-defined class* is like a structure or a record type in other
languages. When you define a class, you specify its name, its direct
superclasses, and its *slots*. A slot has a name and a type. Normally,
each instance stores its own value for the slot. A class inherits the
slots defined by its superclasses, and it can define more slots if it
needs them.

The ``<time-of-day>`` class
~~~~~~~~~~~~~~~~~~~~~~~~~~~

We start by defining a class to represent the concept of a time of day,
such as 21:30. The definition of the ``<time-of-day>`` class is as
follows:

.. code-block:: dylan

    // A specific time of day from 00:00 (midnight) to below 24:00 (tomorrow)
    define class <time-of-day> (<object>) // 1
      slot total-seconds :: <integer>; // 2
    end class <time-of-day>; // 3

The top line is a *comment*. The ``//`` characters begin a comment, which
continues to the end of the line. We also provide comments that number
the lines of code after the first comment. The line numbers are useful
only for discussing the code examples in the book, and would not be used
in source files. You can also have multi-line comments that start with
``/*`` and end with ``*/``.

On line 1, the words ``define class`` start the class definition. The name
of the class is ``<time-of-day>``. The list following the name of the
class is a list of the direct superclasses of this class. The
``<time-of-day>`` class has one direct superclass, which is the class
``<object>``. Each user-defined class must have at least one direct
superclass. If no other class is appropriate, the class must have
``<object>`` as its superclass.

Line 2 contains the only slot definition of this class. This class has
one slot, named ``total-seconds``. The slot’s type constraint is
``<integer>``. The double colon, ``::``, specifies the type constraint of
a slot, just as it specifies the type constraint of a module variable or
of a method’s parameter.

Line 3 is the end of the class definition. The text after the word ``end``
and before the semicolon is an optional part of the definition; it
documents which definition is ending. Any text appearing after the ``end``
must match the definition ending, such as ``end class <time-of-day>``, or
``end class``. You do not need to put any text after the ``end`` — however,
such text is useful for long or complex definitions, where it can be
difficult to see which language construct is ending.

.. index::
   single: type constraint; of slots

The type constraint of a slot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The type constraint of the ``total-seconds`` slot is ``<integer>``. This
slot can hold instances of ``<integer>``, and cannot hold any other kind
of object.

The type constraint of a slot is optional. Specifying a slot with no
explicit type constraint is equivalent to specifying ``<object>`` as the
type constraint. A slot whose type constraint is ``<object>`` can hold any
object. The ability to have slots with the type constraint ``<object>``
provides flexibility that can be valuable; for more information, see
:doc:`perform`.

.. index:: make

Use of ``make`` to create an instance
-------------------------------------

We want to make an instance of ``<time-of-day>``, but first we need a
place to store it. We define a module variable called
``*my-time-of-day*``, and initialize it to contain a new instance of
``<time-of-day>``:

.. code-block:: dylan-console

    ? define variable *my-time-of-day* = make(<time-of-day>);

The ``make`` function creates an instance of ``<time-of-day>``. The
argument to ``make`` is the class to create. The ``make`` function returns
the new instance.

The instance stored in ``*my-time-of-day*`` has a ``total-seconds`` slot
with no value. The next logical step is to store a value in that slot.

.. _usr-class-getters-setters:

Getters and setters of slot values
----------------------------------

We can store a value in the ``total-seconds`` slot of the ``<time-of-day>``
instance by using the assignment operator, ``:=``, as follows:

.. code-block:: dylan-console

    ? *my-time-of-day*.total-seconds := 180;
    => 180

We can examine the value of the slot in the instance:

.. code-block:: dylan-console

    ? *my-time-of-day*.total-seconds;
    => 180

Although these expressions may look like they are accessing the slots
directly, they are not. They are abbreviations for function calls to a
getter and a setter. A *getter* is a method that retrieves the current
value of a slot in an object. A *setter* is a method that stores a value
in a slot. Each slot in a class automatically has a getter and a setter
defined for it. You can see the function-call syntax, and other
information about getters and setters, in :doc:`slots`.

Initialization of slots when instances are made
-----------------------------------------------

So far, we have made an instance and set the value of its slot. We might
like to combine those two steps and to set the slot’s value while making
the instance — in other words, to *initialize* the slot when we make the
instance. One way to do that is to provide a *keyword argument* to
``make``. (Dylan offers several techniques for initializing slots; see
:doc:`slots`.)

Keyword arguments in function calls
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We would like to be able to call ``make`` as follows:

.. code-block:: dylan-console

    ? make(<time-of-day>, total-seconds: 120);

We will be able to make this call after we have done a bit of homework,
as we shall show in `Init keywords: Keywords that initialize slots`_.
In the preceding call to ``make``, we provided a keyword argument,
consisting of a keyword, ``total-seconds:``, followed by a value, ``120``.
The ``<time-of-day>`` instance returned by ``make`` has its
``total-seconds`` slot set to ``120``.

A *keyword argument* consists of a keyword followed by the keyword’s
value. A *keyword* is a name followed by a colon, such as
``total-seconds:``. The colon after a keyword is not a convention; it is
a required part of the keyword. There must be no space between the name
and the colon.

You can define functions to accept keyword arguments. When a function
accepts keyword arguments, you can provide them in any order. Keyword
arguments can be useful for functions that take many arguments — when
you call the function, you do not need to remember the order of the
arguments. Keyword arguments are optional arguments, so they are useful
for parameters that have a default value that you may want to override
at times. For more information about keyword arguments, see
:ref:`func-parameter-lists`.

How does ``make`` know that the value of the ``total-seconds:`` keyword
should be used to initialize the ``total-seconds`` slot? The keyword and
the slot happen to have the same name, but that is not how it knows.
Before you can use the ``total-seconds:`` keyword argument to ``make``, you
must associate that keyword with the ``total-seconds`` slot in the class
definition.

Init keywords: Keywords that initialize slots
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``total-seconds:`` keyword is an *init keyword* — a keyword that we
can give to ``make`` to provide an initial value for a slot. To make it
possible to give an init keyword to ``make``, we need to use the
``init-keyword:`` slot option when we define the class. A *slot option*
lets us specify a characteristic of a slot. Slot options appear after
the optional type specifier of a slot.

Here, we redefine the ``<time-of-day>`` class to use the ``init-keyword:``
slot option:

.. code-block:: dylan
   :linenos:

    // A specific time of day from 00:00 (midnight) to below 24:00 (tomorrow)
    define class <time-of-day> (<object>)
      slot total-seconds :: <integer>, init-keyword: total-seconds:;
    end class <time-of-day>;

The preceding definition *redefines* the class ``<time-of-day>``. That
is, this new definition of ``<time-of-day>`` replaces the old definition
of ``<time-of-day>``.

In line 3, the ``init-keyword:`` slot option defines ``total-seconds:`` as a
keyword parameter that we can give to ``make`` when we make an instance of
this class. Now that we have defined ``total-seconds:`` as an init
keyword, we can provide the keyword argument as follows:

.. code-block:: dylan-console

    ? *my-time-of-day* := make(<time-of-day>, total-seconds: 120);
    => {instance of <time-of-day>}

The preceding expression creates a new instance of ``<time-of-day>``, and
stores that instance in the variable ``*my-time-of-day*``. The value of
the ``total-seconds`` slot of this instance is initialized to ``120``. The
assignment operator returns the new value stored; in the preceding call,
the new value is the newly created instance of ``<time-of-day>``, which
the listener displays as ``{instance of <time-of-day>}``.

We can use the getter to verify that the slot has an initial value:

.. code-block:: dylan-console

    ? *my-time-of-day*.total-seconds;
    => 120

If you call ``make`` and provide a keyword that has not been declared as a
valid keyword for the class, you get an error; for example,

.. code-block:: dylan-console

    ? make(<time-of-day>, seconds: 120);
    => ERROR: seconds: is not a valid keyword argument to make for {class <time-of-day>}

.. topic:: Automatic storage-management note:

   Dylan provides automatic storage management (also called garbage
   collection). Thus, you do not need to deallocate memory explicitly.
   When an object becomes inaccessible, Dylan’s automatic storage
   management will recycle the storage used by that object. In this
   section, there are two examples of objects that become inaccessible:

   -  We redefined the ``<time-of-day>`` class. The storage used by the old
      class definition can be recycled.
   -  We stored a new instance in ``*my-time-of-day*``. The storage used
      by the instance previously stored in that variable can be recycled.

   Although redefinition is not part of the Dylan language, most Dylan
   development environments support redefinition.

.. topic:: Comparison with Java:

   Java recognizes that manual memory management can be the source of
   program errors and often can be exploited to breach security measures.
   Like Dylan, Java has an automatic garbage collector that correctly and
   efficiently recovers unused objects in a program — freeing the
   programmer of that mundane but difficult chore.

Methods for handling time
-------------------------

We decided to represent the time of day with a single slot named
``total-seconds``. An alternate choice would be to give the class three
slots, named ``hours``, ``minutes``, and ``seconds``. People naturally
think of time in terms of hours, minutes, and seconds. We chose to store
the total seconds instead, because we envisioned needing to operate on
times, such as adding a time of day to a time offset. For example, if it
is 9:00 now, and a meeting is to be held 2.5 hours from now, then the
meeting will be held at 11:30. It is easier to operate on a single
value, rather than on three values of hours, minutes, and seconds. On
the other hand, it is convenient to see times expressed as hours,
minutes, and seconds. We can represent the instances with a single slot,
and can provide methods that let users create and see ``<time-of-day>``
instances as being hours, minutes, and seconds.

Method for ``encode-total-seconds``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can provide a method that converts from hours, minutes, and seconds
to total seconds:

.. code-block:: dylan
   :linenos:

    define method encode-total-seconds
        (hours :: <integer>, minutes :: <integer>, seconds :: <integer>)
     => (total-seconds :: <integer>)
      ((hours * 60) + minutes) * 60 + seconds;
    end method encode-total-seconds;

Line 2 contains the parameter list of the method ``encode-total-seconds``.
The method has three required parameters, named ``hours``, ``minutes``,
and ``seconds``, each of type ``<integer>``. This method is invoked when
``encode-total-seconds`` is called with three integer arguments.

Line 3 contains the *value declaration*, which starts with the
characters ``=>``. It is a list declaring the values returned by the
method. Each element of the list contains a descriptive name of the
return value and the type of the value (if the type is omitted, it is
``<object>``). In this case, there is one value returned, named
``total-seconds``, which is of the type ``<integer>``. The name of a
return value is used purely for documentation purposes. Although methods
are not required to have value declarations, there are advantages to
supplying those declarations. When you provide a value declaration for a
method, the compiler signals an error if the method tries to return a
value of the wrong type, can check receivers of the results of the
method for correct type, and can usually produce more efficient code.
These advantages are significant, so we use value declarations
throughout the rest of this book. For more information about value
declarations, see :ref:`func-value-declarations`.

Line 4 is the only expression in the body. It uses arithmetic functions
to convert the hours, minutes, and seconds into total seconds. All
methods return the value of the expression executed last in the body.
This method returns the result of the arithmetic expression in line 4.

In line 5, we could have simply used ``end;``. We provided ``end method
decode-total-seconds;`` for documentation purposes. Throughout the rest
of this book, we provide the extra words after the ``end`` of a
definition.

We can call ``encode-total-seconds`` with arguments representing 8 hours,
30 minutes, and 59 seconds:

.. code-block:: dylan-console

    ? encode-total-seconds(8, 30, 59);
    => 30659

We find it convenient to call ``encode-total-seconds`` to initialize the
``total-seconds`` slot when we create an instance of ``<time-of-day>``, or when
we store a new value in that slot. Here, for example, we create a new instance:

.. code-block:: dylan-console

    ? define variable *your-time-of-day*
      = make(<time-of-day>, total-seconds: encode-total-seconds(8, 30, 59));

We examine the value of the ``total-seconds`` slot:

.. code-block:: dylan-console

    ? *your-time-of-day*.total-seconds;
    => 30659

The result reminds us that it would be useful to convert in the other
direction as well — from total seconds to hours, minutes, and seconds.

.. _method-for-decode-total-seconds:

Method for ``decode-total-seconds``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We define ``decode-total-seconds`` to convert in the other direction —
from total seconds to hours, minutes, and seconds:

.. code-block:: dylan
    :linenos:

    define method decode-total-seconds
        (total-seconds :: <integer>)
     => (hours :: <integer>, minutes :: <integer>, seconds :: <integer>)
      let (total-minutes, seconds) = truncate/(total-seconds, 60);
      let (hours, minutes) = truncate/(total-minutes, 60);
      values(hours, minutes, seconds);
    end method decode-total-seconds;

We can use ``decode-total-seconds`` to see the value of the
``total-seconds`` slot:

.. code-block:: dylan-console

    ? decode-total-seconds(*your-time-of-day*.total-seconds);
    => 8
    => 30
    => 59

The value declaration on line 3 specifies that ``decode-total-seconds``
returns three separate values: the hours, minutes, and seconds. This
method illustrates how to return multiple values, and how to use *let*
to initialize multiple local variables. We describe these techniques in
Sections `Multiple return values`_ and `Use of let to declare local variables`_.

Multiple return values
~~~~~~~~~~~~~~~~~~~~~~

The method for ``decode-total-seconds`` returns three values: the hours,
the minutes, and the seconds. To return the three values, the method
uses the ``values`` function as the expression executed last in the body.
The ``values`` function simply returns all its arguments as separate
values. The ability to return multiple values allows a natural symmetry
between ``encode-total-seconds`` and ``decode-total-seconds``, as shown in
`symmetry-of-encode-decode`_.

.. _symmetry-of-encode-decode:

.. table:: Symmetry of ``encode-total-seconds`` and ``decode-total-seconds``.

    +--------------------------+-----------------------------+-----------------------------+
    | Method                   | Parameter(s)                | Return value(s)             |
    +==========================+=============================+=============================+
    | ``encode-total-seconds`` | ``hours, minutes, seconds`` | ``total-seconds``           |
    +--------------------------+-----------------------------+-----------------------------+
    | ``decode-total-seconds`` | ``total-seconds``           | ``hours, minutes, seconds`` |
    +--------------------------+-----------------------------+-----------------------------+

Lines 4 and 5 of the ``decode-total-seconds`` method contain calls to
``truncate/``. The ``truncate/`` function is a built-in Dylan function. It
takes two arguments, divides the first by the second, and returns two
values: the result of the truncating division, and the remainder.

.. topic:: Comparison with C:

   In C, ``/`` on integers produces a truncated result.  In Dylan,
   ``/`` on integers is implementation defined, and is not
   recommended for portable code. The Dylan functions named ``floor``,
   ``ceiling``, ``round``, and ``truncate`` convert a rational or
   floating-point result to an integer with the appropriate rounding.
   The Dylan functions named ``floor/``, ``ceiling/``, ``round/``, and
   ``truncate/`` take two arguments. Those generic functions divide the
   first argument by the second argument, and return two values: the
   rounded or truncated result, and the remainder.

Use of *let* to declare local variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a function returns multiple values, you can use ``let`` to store each
returned value in a local variable, as shown in lines 2 and 3 of the
``decode-total-seconds`` method in :ref:`method-for-decode-total-seconds`.
On line 2, we use ``let`` to declare two local variables, named
``total-minutes`` and ``seconds``, and to initialize their values to the
two values returned by the ``truncate/`` function. Similarly, on line 3,
we use ``let`` to declare the local variables ``hours`` and ``minutes``.

The local variables declared by ``let`` can be used within the method
until the method’s ``end``. Although there is no ``begin`` to define
explicitly the beginning of a body for local variables, ``define method``
begins a body, and its ``end`` finishes that body. Local variables are
scoped within the smallest body that surrounds them, so you can use
``begin`` and ``end`` within a method to define a smaller body for local
variables, although doing so is usually not necessary.

.. _usr-class-second-method-decode-total-seconds:

Second method for ``decode-total-seconds``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``decode-total-seconds`` method is called as follows:

.. code-block:: dylan-console

    ? decode-total-seconds(*your-time-of-day*.total-seconds);

If we envision calling ``decode-total-seconds`` frequently to see the
hours, minutes, and seconds stored in a ``<time-of-day>`` instance, we can
make it possible to decode ``<time-of-day>`` instances, as well as
integers. For example, we can make it possible to make this call:

.. code-block:: dylan-console

    ? decode-total-seconds(*your-time-of-day*);

We can implement this behavior easily, by defining another method for
``decode-total-seconds``, which takes a ``<time-of-day>`` instance as its
argument:

.. code-block:: dylan

    define method decode-total-seconds
        (time :: <time-of-day>)
     => (hours :: <integer>, minutes :: <integer>, seconds :: <integer>)
      decode-total-seconds(time.total-seconds);
    end method decode-total-seconds;)

`The decode-total-seconds generic function and its methods
<decode-total-seconds-methods>`_ shows the two methods for the
``decode-total-seconds`` generic function.

.. _decode-total-seconds-methods:

The ``decode-total-seconds`` generic function and its methods.

.. code-block:: dylan

    // Method on <integer>
    define method decode-total-seconds
        (total-seconds :: <integer>)
     => (hours :: <integer>, minutes :: <integer>, seconds :: <integer>)
      let (total-minutes, seconds) = truncate/(total-minutes, 60);
      values(hours, minutes, seconds);
    end method decode-total-seconds;

    // Method on <time-of-day>
    define method decode-total-seconds
        (time :: <time-of-day>)
     => (hours :: <integer>, minutes :: <integer>, seconds :: <integer>)
      decode-total-seconds(time.total-seconds);
    end method decode-total-seconds;

Looking at `The decode-total-seconds generic function and its methods
<decode-total-seconds-methods>`_, we analyze what happens in this call:

.. code-block:: dylan-console

    ? decode-total-seconds(*your-time-of-day*);

#. The argument is an instance of ``<time-of-day>``, so the method on
   ``<time-of-day>`` is called.
#. The body of the method on ``<time-of-day>`` calls
   ``decode-total-seconds`` on an instance of ``<integer>``, the value of
   the ``total-seconds`` slot of the ``<time-of-day>`` instance. In this
   call, the argument is an integer, so the method on ``<integer>`` is
   called.
#. The method on ``<integer>`` returns three values to its caller — the
   method on ``<time-of-day>``. The method on ``<time-of-day>`` returns
   those three values.

The purpose of the method on ``<time-of-day>`` is simply to allow a
different kind of argument to be used. The method extracts the integer
from the ``<time-of-day>`` instance, and calls ``decode-total-seconds`` with
that integer.

Method for ``say-time-of-day``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can provide a way to ask an instance of ``<time-of-day>`` to describe
the time in a conventional format, such as 8:30. For the application
that we are planning, there is no need to view the seconds. We want the
method to print the description in a window on the screen. We define a
method named ``say-time-of-day``:

.. code-block:: dylan
   :linenos:

    define method say-time-of-day (time :: <time-of-day>) => ()
      let (hours, minutes) = decode-total-seconds(time);
      format-out
        ("%d:%s%d", hours, if (minutes < 10) "0" else "" end, minutes);
    end method say-time-of-day;

On line 1, we provide an empty value declaration, which means that this
method returns no values.

On line 2, we use ``let`` to initialize two local variables to the first
and second values returned by ``decode-total-seconds``. Remember that
``decode-total-seconds`` returns three values (the third value is the
seconds). For the application that we are planning, the
``say-time-of-day`` method does not need to show the seconds, so we do not
use the third value. It is not necessary to receive the third value of
``decode-total-seconds``; here we do not provide a local variable to
receive the third value, so that value is simply ignored.

On line 4, we use ``if`` to print a leading 0 for the minutes when there
are fewer than 10 minutes, such as ``2:05``.

.. topic:: Comparison to C:

   In C, ``if`` does not return a value. In Dylan, ``if``
   returns the value of the body that is selected, if any is.

.. topic:: Note on ``format-out``:

   We have purposely used a limited subset of the ``format-out``
   function’s features to allow our examples to run on as many
   Dylan implementations as possible. The printing of times could be
   done much more elegantly if we used the full power of the ``format-out``
   function.

We can call ``say-time-of-day``:

.. code-block:: dylan-console

    ? say-time-of-day(*your-time-of-day*);
    => 8:30

    ? say-time-of-day(*my-time-of-day*);
    => 0:02

The listener displays the output (printed by ``format-out``), but
displays no values, because ``say-time-of-day`` does not return any
values.

Summary
-------

In this chapter, we covered the following:

- We defined a class (with ``define class``).
- We created an instance (with ``make``).
- We read the value of a slot by calling a getter.
- We set the value of a slot by using ``:=``, the assignment operator.
- We defined a method that returns multiple values (with ``values``),
  and showed how to initialize multiple local variables (with ``let``).
- We showed the syntax of some commonly used elements of Dylan; see
  :ref:`syntax-of-dylan-elements`.

.. _syntax-of-dylan-elements:

.. table:: Syntax of Dylan elements.

   +---------------------+---------------------------------------------------------+
   | Dylan element       | Syntax example                                          |
   +=====================+=========================================================+
   | calling a getter    | ``*my-time-of-day*.total-seconds;``                     |
   +---------------------+---------------------------------------------------------+
   | calling a setter    | ``*my-time-of-day*.total-seconds := 180;``              |
   +---------------------+---------------------------------------------------------+
   | keyword             | ``total-seconds:``                                      |
   +---------------------+---------------------------------------------------------+
   | single-line comment | ``// Text of comment``                                  |
   +---------------------+---------------------------------------------------------+
   | multiline comment   | ``/* Text of comment that spans more than one line */`` |
   +---------------------+---------------------------------------------------------+
   | value declaration   | ``=> (total-seconds :: <integer>)``                     |
   +---------------------+---------------------------------------------------------+
