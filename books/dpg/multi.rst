Multimethods
============

In this chapter, we show two important techniques. First, we define
methods for built-in generic functions — in this case, for the functions ``+``,
``<``, and ``=``. Second, we define multimethods. We describe how method
dispatch works for multimethods.

.. _multi-methods-for-plus-gf:

Methods for the ``+`` generic function
--------------------------------------

We need to make it possible to add one time to another. We could define
a method with a name such as ``add`` or ``plus``. However, the concept of
adding times is the same as the concept of adding numbers. Dylan already
provides the + generic function for adding numbers. Instead of inventing
a new name for the addition operation, we define new methods on the
built-in generic function ``+``. We can extend ``+`` by defining new
methods for it. In certain languages, this technique is called *operator
overloading*.

.. topic:: Comparison with C++ and Java:

   In C++, operator overloading means customizing the action of any built-in
   operator for classes that you define. In Dylan, operators are just generic
   functions, and you can add methods to those generic functions for your
   classes. In C++, the meaning of an overloaded operator is resolved at compile
   time — the types of the operands must be known at compile time. Because Dylan
   operators are generic functions, the method is chosen dynamically according
   to the argument types —at run time, if the types may vary at run time.

   Java does not allow operator overloading. The Java designers believe
   that overloading of operators results in inscrutable code (because the
   meaning of the operator can vary). Dylan and C++ designers believe that,
   judiciously used, operator overloading permits clearer, more concise
   code.

Method for adding two time offsets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We now define a method for ``+``. The method adds two time offsets and
returns the sum, which is also a time offset:

.. code-block:: dylan
   :linenos:

    // Method on <time-offset>, <time-offset>
    define method \+
        (offset1 :: <time-offset>, offset2 :: <time-offset>)
     => (sum :: <time-offset>)
      let sum = offset1.total-seconds + offset2.total-seconds;
      make(<time-offset>, total-seconds: sum);
    end method \+;

On line 2, notice that the method is defined on ``\+``, rather than
simply on ``+``. When we define a method on ``+`` or on another infix
function, we need to use a backslash before the function name. The
backslash clarifies that we mean the value of the variable + (which is a
generic function), and that we are not trying to call the function.

On line 4, we add the values stored in the ``total-seconds`` slots of the
two instances. On line 5, we make and return a new instance of
``<time-offset>``. We initialize the ``total-seconds`` slot to contain the
sum calculated in line 4.

To test the method, we need to create two instances of ``<time-offset>``:

.. code-block:: dylan

    define variable *minus-2-hours* =
      make(<time-offset>, total-seconds: - encode-total-seconds (2, 0, 0));

    define variable *plus-15-20-45* =
      make(<time-offset>, total-seconds: encode-total-seconds (15, 20, 45));

We can add the time offsets:

.. code-block:: dylan-console

    ? *minus-2-hours* + *plus-15-20-45*;
    => {instance <time-offset>}

The result is a new instance of ``<time-offset>``. We did not save the
value returned. (Many environments offer a way to access values returned
by the listener.) We can add the time offsets again, and view the
``total-seconds`` slot of the result:

.. code-block:: dylan-console

    ? decode-total-seconds(*minus-2-hours* + *plus-15-20-45*);
    => 13
    => 20
    => 45

Methods for adding a time of day to a time offset
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These methods implement addition between a time offset and a time of
day:

.. code-block:: dylan

    // Method on <time-offset>, <time-of-day>
    define method \+
        (offset :: <time-offset>, time-of-day :: <time-of-day>)
     => (sum :: <time-of-day>)
      make(<time-of-day>,
      total-seconds: offset.total-seconds + time-of-day.total-seconds);
    end method \+;

The method on ``<time-offset>``, ``<time-of-day>`` is invoked when the
first argument is a time offset and the second argument is a time of
day. It does the work of creating a new ``<time-of-day>`` instance with
the ``total-seconds`` slot initialized to the sum of the ``total-seconds``
slots of the two arguments.

.. code-block:: dylan

    // Method on <time-of-day>, <time-offset>
    define method \+
        (time-of-day :: <time-of-day>, offset :: <time-offset>)
     => (sum :: <time-of-day>)
      offset + time-of-day;
    end method \+;

The method on ``<time-of-day>``, ``<time-offset>`` is invoked when the
first argument is a time of day and the second argument is a time
offset. It simply calls ``+`` with the order of the arguments switched —
this call invokes the method on ``<time-offset>``, ``<time-of-day>``.

To test these methods, we can use one of the time offsets created in
`Method for adding two time offsets`_, and define
the ``*8-30-59*`` variable, which contains a ``<time-of-day>`` instance,
which we define as follows:

.. code-block:: dylan

    define variable *8-30-59* =
      make(<time-of-day>, total-seconds: encode-total-seconds(8, 30, 59));

We add the time offset and the time of day:

.. code-block:: dylan-console

    ? decode-total-seconds(*minus-2-hours* + *8-30-59*);
    => 6
    => 30
    => 59

We add the time of day and the time offset:

.. code-block:: dylan-console

    ? decode-total-seconds(*8-30-59* + *minus-2-hours*);
    => 6
    => 30
    => 59

.. _multi-adding-other-times:

Method for adding other kinds of times
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We have already defined methods for adding the kinds of time that it
makes sense to add together. It is not logical to add one time of day to
another time of day — what would three o’clock plus two o’clock mean?
Someone could create another concrete subclass of ``<time>``, without
providing any methods for adding that time to other times. If someone
tries to add times that we do not intend them to add, the result will be
a “No applicable method” error.

We could provide a method whose sole purpose is to give more information
to the user than “No applicable method” when + is called on two times
that cannot be added, because there is no applicable method for adding
them. We define such a method here:

.. code-block:: dylan

    // Method on <time>, <time>
    define method \+ (time1 :: <time>, time2 :: <time>)
      error("Sorry, we can't add a %s to a %s.",
            object-class(time1), object-class(time2));
    end method \+;

This method is called only when the arguments are both general instances
of ``<time>``, and none of the more specific methods are applicable to
the arguments. The ``error`` function signals an error. For more
information about signaling and handling errors, see :doc:`exceptions`.

Note: This method is useful for explaining how method dispatch works for
multimethods, but it does not really give the user any more useful
information than that supplied by the “No applicable method” error.
Therefore, we define the method in this chapter, but do not include it
as part of the final library.

.. _multi-method-dispatch:

Method dispatch for multimethods
--------------------------------

A method is *specialized* on the required parameters that have explicit
types. The type of the required parameter is called that parameter’s
*specializer*. A *multimethod* is a method that specializes more than
one of its parameters. The methods that we defined in
`Methods for the + generic function`_ specialize two required
parameters, and therefore are multimethods.

.. topic:: Comparison with C++ and Java:

   Neither C++ nor Java supports multimethods. In both languages, method
   dispatch is based on the first argument of virtual functions.

The method dispatch considers all the required parameters, and sorts the
applicable methods by specificity as follows: For each required
parameter, construct a separate list of the applicable methods, sorted
from most specific to least specific for that parameter. Then, combine
the separate sorted lists into an overall list of methods, sorted by
specificity. In the overall method ordering, a method is more specific
than another if it satisfies two constraints:

#. The method is *no less specific* than the other method for *all*
   required parameters. (The two methods might have the same types for some
   parameters.)

#. The method is *more specific* than the other method for *some*
   required parameter.

One method might be more specific than another for one parameter, but
less specific for another parameter. These two methods are *ambiguous*
in specificity and cannot be ordered. If the method-dispatch procedure
cannot find any method that is more specific than all other methods,
Dylan signals an error.

.. _applicable-methods-multi:

.. table:: Applicable methods for different arguments to ``+``, ordered by specificity.

   +------------------------+-------------------------+--------------------------------------------------+
   | Type of first argument | Type of second argument | Applicable methods, ordered by specificity       |
   +========================+=========================+==================================================+
   | ``<time-offset>``      | ``<time-offset>``       | # method on ``<time-offset>``, ``<time-offset>`` |
   |                        |                         | # method on ``<time>``, ``<time>``               |
   +------------------------+-------------------------+--------------------------------------------------+
   | ``<time-of-day>``      | ``<time-offset>``       | # method on ``<time-of-day>``, ``<time-offset>`` |
   |                        |                         | # method on ``<time>``, ``<time>``               |
   +------------------------+-------------------------+--------------------------------------------------+
   | ``<time-offset>``      | ``<time-of-day>``       | # method on ``<time-offset>``, ``<time-of-day>`` |
   |                        |                         | # method on ``<time>``, ``<time>``               |
   +------------------------+-------------------------+--------------------------------------------------+
   | ``<time-of-day>``      | ``<time-of-day>``       | # method on ``<time>``, ``<time>``               |
   +------------------------+-------------------------+--------------------------------------------------+
   | ``<integer>``          | ``<time-offset>``       | none                                             |
   +------------------------+-------------------------+--------------------------------------------------+

:ref:`applicable-methods-multi` shows the applicable methods for
various arguments to +. If two methods are applicable, we number the
more specific method 1, and the less specific method 2.

We call ``+`` on two instances of ``<time-offset>``:

.. code-block:: dylan-console

    ? *minus-2-hours* + *plus-15-20-45*;
    => {instance of <time-offset>}

When both arguments are instances of ``<time-offset>``, the first row of
the table applies. Two methods are applicable. The method on
``<time-offset>``, ``<time-offset>`` is more specific than the method on
``<time>``, ``<time>``. The parameter specializers of the method on
``<time-offset>``, ``<time-offset>`` are subtypes of the parameter
specializers of the method on ``<time>``, ``<time>``. That is, for the
first parameter, ``<time-offset>`` is a subtype of ``<time>``; for the
second parameter, ``<time-offset>`` is a subtype of ``<time>``.

Methods for comparison of times
-------------------------------

We need to compare times to see whether they are the same, and to see
whether one is greater (later) than another. These methods do the
comparisons we need:

.. code-block:: dylan

    define method \<
        (time1 :: <time-of-day>, time2 :: <time-of-day>)
     => (boolean :: <boolean>)
      time1.total-seconds < time2.total-seconds;
    end method \<;

    define method \<
        (time1 :: <time-offset>, time2 :: <time-offset>)
     => (boolean :: <boolean>)
      time1.total-seconds < time2.total-seconds;
    end method \<;

    define method \=
        (time1 :: <time-of-day>, time2 :: <time-of-day>)
     => (boolean :: <boolean>)
      time1.total-seconds = time2.total-seconds;
    end method \=;

    define method \=
        (time1 :: <time-offset>, time2 :: <time-offset>)
     => (boolean :: <boolean>)
      time1.total-seconds = time2.total-seconds;
    end method \=;

We can call these methods:

.. code-block:: dylan-console

    ? *plus-15-20-45* = *minus-2-hours*;
    => #f

To compare times, we need only to define methods for ``<`` and ``=``. All
other numerical comparisons in Dylan are based on these two methods. So, we
can call ``>``, ``>=``, ``<=``, and ``~=`` (the not-equal-to function). Here
are examples:

.. code-block:: dylan-console

    ? *plus-15-20-45* ~= *minus-2-hours*;
    => #t

    ? *plus-15-20-45* > *minus-2-hours*;
    => #t

Summary
-------

In this chapter, we covered the following:

- We defined new methods on the built-in generic functions +, ``<``, and
  ``=``.
- We discussed how method dispatch works for multimethods.

