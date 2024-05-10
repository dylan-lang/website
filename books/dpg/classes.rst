Nonclass Types
==============

.. index::
   single: non-class types
   single: type; non-class types

Every class is a type, but not every type is a class. In this chapter,
we describe how to create nonclass types, and how to make use of them.

.. _classes-functions-create-nonclass-types:

Functions that create nonclass types
------------------------------------

There are three functions that create types that are not classes:
``singleton``, ``type-union``, and ``limited``.

``singleton``
  Takes any instance, and creates a type whose only member is
  that instance. You can define a singleton type to be used as the
  parameter specializer of a method that should be chosen for a particular
  instance.

``type-union``
  Takes one or more classes or types, and creates a new type
  whose members include all the members of the types that are its
  arguments.

``limited``
  Takes a type and creates a new type, which is a more
  restricted version of the type that is its argument (the *base type*).
  For example, you can define a new type that is based on ``<integer>``,
  but has a given minimum or maximum value. Another example is to define a
  new collection type that specifies the type of elements, such as a type
  that is a vector of integers. The main reasons for defining types with
  ``limited`` are to perform type checking and to increase efficiency. For
  information about the performance of limited types, see
  :ref:`perform-limited-types`.

.. index::
   pair: type; naming conventions

.. topic:: Convention:

   Type names, like class names, are surrounded with angle
   brackets — for example, ``<nonnegative-integer>``.

.. _classes-examples-types-not-classes:

Examples of types that are not classes
--------------------------------------

Later in our development of the time library, we shall find it useful to
define a new type that represents nonnegative integers:

.. code-block:: dylan

    // Define nonnegative integers as integers that are >= zero
    define constant <nonnegative-integer> = limited(<integer>, min: 0);

We can use a nonclass type as a parameter specializer of a method, or as
the type of a return value:

.. code-block:: dylan

    define method encode-total-seconds
        (max-unit :: <nonnegative-integer>,
         minutes :: <nonnegative-integer>,
         seconds :: <nonnegative-integer>)
     => (total-seconds :: <nonnegative-integer>)
      ((max-unit * 60) + minutes) * 60 + seconds;
    end method encode-total-seconds;

To see how we use ``<nonnegative-integer>`` in the time library, see
:ref:`slots-setter-methods`.

We can define a type whose only member is the false value, ``#f``:

.. code-block:: dylan

    singleton(#f);

We can define a type that is the union of the false value and
``<integer>``:

.. code-block:: dylan

    type-union(singleton(#f), <integer>);

We can make it convenient for people to create new types like the one
defined in the preceding code. The new type is the union of the false
value and the argument to the method:

.. code-block:: dylan

    define method false-or (other-type :: <type>)
     => (combined-type :: <type>)
      type-union(singleton(#f), other-type);
    end method false-or;

``false-or`` types are useful as the type of slots. Note that a slot can
be uninitialized. Once a slot receives a value, however, it will always
have a value: There is no way to return a slot to the uninitialized
state. Sometimes it is useful to store in a slot a value that means
none. Later on in our development of the airport example, we use a
false-or type as the type of a slot that stores “the next vehicle, if
there is one.” If there is no next vehicle, the slot contains ``#f``. We
create the type by calling ``false-or(<vehicle>)``, and use the result as
the type of the slot. Note that, if the type of the slot were just
``<vehicle>``, we could not store ``#f`` in the slot, and there would be no
way to represent none.

You can use ``type-union`` and ``singleton`` together to define a type that
is an enumeration of multiple-choice objects. For example,

.. code-block:: dylan

    define constant <latitude-direction>
      = type-union(singleton(#"north"), singleton(#"south"));

The ``<latitude-direction>`` type has two valid values: the keywords
``#"north"`` and ``#"south"``. For an explanation of how we could use that
type to enforce the correct values of a latitude slot, and for
information about the performance of enumerations, see
:ref:`perform-enumerations`.

.. _classes-method-dispatch-nonclass-types:

Method dispatch and nonclass types
----------------------------------

In this section, we describe the implications for method dispatch of
using nonclass types as parameter specializers. This advanced topic is
included as reference material; you can skip it safely if you prefer.
The description that we give here is meant to provide a general
understanding, and does not cover all cases. For exact details, you
should consult *The Dylan Reference Manual*.

Recall that, when a generic function is called, Dylan determines which
method to invoke by comparing the required *arguments* passed to the
generic function with the types of the corresponding *parameters* of the
generic function’s methods. Dylan uses the following procedure, assuming
that there is only one required argument:

#. Find all the applicable methods. A method is applicable if the
   required argument is an instance of the type of the specialized
   parameter.

#. Sort the applicable methods in order of specificity. One method is
   more specific than another if the type of its specialized parameter
   is a *proper subtype* of the type of the other method’s specialized
   parameter. For definitions of “proper subtype” in various situations,
   see Sections `Method dispatch and classes`_ through
   `Method dispatch and limited collections`_.

   (In the presence of multiple inheritance, the specificity rule is more
   complex. For more information, see :ref:`inherit-mi-and-md`.)

#. Call the most specific method.

   (If there is more than one required argument, Dylan constructs the
   sorted list of methods by combining separate sorted lists for all
   required arguments.)

For any given argument and any given set of parameter types, Dylan has
to answer two questions:

#. Is the argument an instance of a given type? The answer determines
   method applicability.

#. Is one type a proper subtype of another type? The answer determines
   method specificity.

Method dispatch and classes
~~~~~~~~~~~~~~~~~~~~~~~~~~~

We have already seen that, when all types are classes, Dylan uses the
following rules:

#. An object is an instance of a class if it is a general instance of
   that class (a direct instance of the class or of one of that class’s
   subclasses).
#. One class is a proper subtype of another if the first class is a
   subclass of the second.

For example, suppose that we have these definitions:

.. code-block:: dylan

    // Method 1
    define method say (x :: <number>) ... end method say;

    // Method 2
    define method say (x :: <integer>) ... end method say;

Now, if ``say`` is called with an argument of *100*, both methods are
applicable, and method 2 is more specific than method 1.

Method dispatch and singletons
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a type is a singleton, Dylan uses the following rules:

#. An object is an instance of a singleton only if the object is
   identical to the object used as the argument in the call to
   ``singleton`` that created the singleton.
#. A singleton is a proper subtype of any other type that the object
   belongs to. Thus, a singleton is more specific than any other type of
   which an object is an instance. In particular, a singleton is more
   specific than the object’s class.

For example, suppose that we have these definitions:

.. code-block:: dylan

    // Method 1
    define method say (x :: <integer>) ... end method say;

    // Method 2
    define method say (x == 0) ... end method say;

Note that method 2 illustrates a convenient syntax for defining a method
on a singleton without calling ``singleton`` explicitly.

Now, if ``say`` is called with an argument of ``0``, both methods are
applicable, and method 2 is more specific than method 1. If ``say`` is
called with an argument that is any other integer, only method 1 is
applicable.

Method dispatch and unions
~~~~~~~~~~~~~~~~~~~~~~~~~~

When a type is a union, Dylan uses the following rules:

#. An object is an instance of a union if it is an instance of any of
   the types that make up that union.
#. If none of the types that make up a union is a subtype of any other,
   then
#. A nonunion type is a proper subtype of a union if the nonunion type
   is a subtype of any of the types that make up the union.
#. A union is a proper subtype of a nonunion type if all types that make
   up the union are subtypes of the nonunion type, and if all the types
   that make up the union, taken together, are not equivalent to the
   nonunion type.
#. A union is a proper subtype of another union if *each* of the types
   that make up the first union is a subtype of *one* of the types that
   make up the other union, and if the two unions are not equivalent.

For example, suppose that we have these definitions:

.. code-block:: dylan

    define constant <false-or-integer> = type-union(<integer>,
                                                    singleton(#f));

    // Method 1
    define method say (x :: <false-or-integer>) ... end method say;

    // Method 2
    define method say (x :: <integer>) ... end method say;

Now, if ``say`` is called with an argument that is an integer, both
methods are applicable, and method 2 is more specific than method 1. If
``say`` is called with an argument of ``#f``, only method 1 is applicable.

Method dispatch and limited integers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a type is a limited-integer type, Dylan uses the following rules:

#. An object is an instance of a limited-integer type if it is an
   instance of ``<integer>`` and if it is (inclusively) within the
   specified range.
#. A limited-integer type is a proper subtype of ``<integer>``, as long
   as it is not equivalent to ``<integer>``.

One limited-integer type is a proper subtype of another limited-integer
type if the range of the first type is entirely within the range of the
second type, and if the two types are not equivalent.

For example, suppose that we have these definitions:

.. code-block:: dylan

    define constant <nonnegative-integer> = limited(<integer>, min: 0);

    // Method 1
    define method say (x :: <integer>) ... end method say;

    // Method 2
    define method say (x :: <nonnegative-integer>) ... end method say;

Now, if ``say`` is called with an argument of ``1``, both methods are
applicable, and method 2 is more specific than method 1. If ``say`` is
called with an argument of ``-1``, only method 1 is applicable.

Now suppose that, instead, we have the following definitions:

.. code-block:: dylan

    define constant <limited-integer-1> = limited(<integer>, min: -2,
                                                  max: 2);

    define constant <limited-integer-2> = limited(<integer>, min: 0,
                                                  max: 4);

    // Method 1
    define method say (x :: <limited-integer-1>) ... end method say;

    // Method 2
    define method say (x :: <limited-integer-2>) ... end method say;

Now, if ``say`` is called with an argument of ``1``, both methods are
applicable, and neither method is more specific than the other; the two
methods are *ambiguous*. If no more specific method exists, Dylan
signals an error when we call ``say`` with an argument of ``1``.

Method dispatch and limited collections
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a type is a limited-collection type, Dylan uses the following
rules:

#. An object is an instance of a limited-collection type if all the
   following are true: the class of the object is a subclass of the base
   type; the two element types are equivalent; and, if the
   limited-collection type restricts the size or dimensions, the size or
   dimensions of the object are the same as those specified for the
   type. If the object is an instance of ``<strectchy-collection>``, the
   limited-collection type cannot restrict the size or dimensions.
#. A limited-collection type is a proper subtype of its base type, as
   long as it is not equivalent to the base type.

Generally, one limited-collection type is a proper subtype of another
limited-collection type if all the following are true: the base type of
the first is a subclass of the base type of the second; the two element
types are equivalent; the size or dimensions of the first limited type
are no less restricted than those of the second type; and the first
limited type is not equivalent to the second.

For example, suppose that we have these definitions:

.. code-block:: dylan

    define constant <limited-vector-of-3-integers>
      = limited(<vector>, of: <integer>, size: 3);

    define constant <limited-vector-of-3-numbers>
      = limited(<vector>, of: <number>, size: 3);

    define constant $v1 = make(<limited-vector-of-3-integers>,
                               size: 3, fill: 1);

    define constant $v2 = vector(1, 1, 1);

    // Method 1
    define method say (x :: <vector>) ... end method say;

    // Method 2
    define method say (x :: <limited-vector-of-3-integers>)
      ...
    end method say;

    // Method 3
    define method say (x :: <limited-vector-of-3-numbers>)
      ...
    end method say;

Now, if ``say`` is called with an argument of ``$v1``, both method 1 and
method 2 are applicable, and method 2 is more specific than method 1.
Note that ``$v1`` is an instance of ``<limited-vector-of-3-integers>`` but
is not an instance of ``<limited-vector-of-3-numbers>``, because the
element type of ``$v1`` is not equivalent to the element type of
``<limited-vector-of-3-numbers>``.

If ``say`` is called with an argument of ``$v2``, only method 1 is
applicable. Note that ``$v2`` is not an instance of either of the
limited-collection types we defined, even though ``$v2`` is a vector that
contains three integers. (For example, we could store objects other than
integers in ``$v2``.)

Summary
-------

In this chapter, we discussed types that are not classes:

- A *singleton type* is a type whose only member is one particular
  instance. An example of creating a singleton type is:

  .. code-block:: dylan

      singleton(#f);

- A *union type* is a type whose members include all the members of one
  or more base types. An example of creating a union type is:

  .. code-block:: dylan

      type-union(singleton(#f), <integer>);

- A *limited type* is a type that is a more restricted version of its
  base type. For example, a limited-integer type is based on
  ``<integer>``, but has a given minimum or maximum value:

  .. code-block:: dylan

      limited(<integer>, min: 0);

  Another example of a limited type is a limited-collection type, which is
  a collection type that specifies the type of elements, and/or the size
  of the collection:

  .. code-block:: dylan

      limited(<vector>, of: <integer>, size: 3);

