Slots
=====

In this chapter, we show how to call getters and setters with the
function-call syntax, and how to define methods for getters and setters.
We show techniques for initializing slots, including slot options and
``initialize`` methods. We describe the different allocations that slots
can have. We find a need for symbols, so we describe and use symbols as
well.

Dot-syntax abbreviation for simple function calls
-------------------------------------------------

The dot syntax that we have shown for getters in :ref:`usr-class-getters-setters`,
is an abbreviation for a function call. The first expression is an
abbreviation for the second expression:

.. code-block:: dylan

    object.function-name
    function-name(object)

The dot syntax used with the assignment operator also is an abbreviation
for a function call. The first two expressions are abbreviations for the
third expression:

.. code-block:: dylan

    object.name := new-value;
    name(object) := new-value;
    name-setter(new-value, object);

You can use the dot syntax as an abbreviation for any function call that
takes a single argument and returns a single value. For example, in
:ref:`offset-methods-on-time-offset`, we defined the following method:

.. code-block:: dylan

    define method past? (time :: <time-offset>) => (past? :: <boolean>)
      time.total-seconds < 0;
    end method past?;

The following two calls are equivalent:

.. code-block:: dylan

    past?(*my-time-offset*);
    *my-time-offset*.past?;

In the remainder of this book, we use the dot syntax for function calls
that return a property of an object (such as the ``past?`` property of a
``<time-offset>`` instance), and that take a single argument and return a
single value.

Getters and setters for slots
-----------------------------

As shown in :ref:`usr-class-getters-setters`, when you define a class, Dylan
automatically defines a getter method to return the value of a slot, and
defines a setter method to change the value of a slot.

.. topic:: Performance note:

   For slot accesses, given accurate type declarations, the compiler can
   typically optimize away not only the method dispatch, but also the
   function call, making the executed code just as efficient as it would be
   in a language such as C, where structure or record slots are accessed
   directly. See :doc:`perform`.

The name of the getter is always the name of the slot. Thus, the getter
for the ``total-seconds`` slot is ``total-seconds``. Let’s look at an
example of calling a getter. The first expression is an abbreviation for
the second expression:

.. code-block:: dylan

    *my-time-of-day*.total-seconds;
    total-seconds(*my-time-of-day*);

The preceding expressions are calls to the getter function named
``total-seconds``. The choice of which syntax to use is purely a matter
of personal style. The first syntax is provided for those people who
prefer the slightly more concise dot syntax. The second syntax is
provided for those people who prefer slot accesses to look like function
calls. In this book, we use the dot syntax.

By default, the name of the setter is the slot’s name followed by
``-setter``. Thus, the setter for the ``total-seconds`` slot is
``total-seconds-setter``. You can use the ``setter:`` slot option to
specify a different name for the setter.

The dot-syntax abbreviation for assignment enables you to invoke the
setter by using assignment with the name of the getter. For example, the
first two expressions are abbreviations for the third expression:

.. code-block:: dylan

    *my-time-of-day*.total-seconds := 180;
    total-seconds(*my-time-of-day*) := 180;
    total-seconds-setter(180, *my-time-of-day*);

Each of these expressions stores the value ``180`` in the slot named
``total-seconds`` of the object that is the value of the
``*my-time-of-day*`` variable.

Most Dylan programmers do not use the syntax of the third expression to
call a setter, because it is more verbose than the first and second
expressions. However, it is important to know the name of the setter, so
that you can define setter methods. For example, to define a method on
the setter for the ``total-seconds`` slot, you define it on
``total-seconds-setter``. For an example of a setter method, see
`Setter methods`_.

If you do not want Dylan to define a setter method for a slot, you can
define the slot to be constant, using the ``constant`` slot adjective, or
you can give the ``setter: #f`` slot option.

For more information about accessing slots, see :ref:`func-slot-references`,
and :ref:`func-assignment`.

Advantages of accessing slots via generic functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A slot is conceptually like a variable, in that it has a value. But the
only way to access a slot’s value is to call a generic function. Using
generic functions and methods to gain access to slot values has three
important advantages:

- Generic functions provide a public interface to the private
  implementation of a slot. By making the representation of the slot
  visible to only the methods of the generic functions, you can change
  the representation without changing any of the users of the
  information — the callers of the generic functions. In most cases, a
  compiler can optimize slot references to reduce or eliminate the cost
  of hiding the implementation.
- A subclass can specialize, or filter, references to superclass slots.
  For example, the classes ``<latitude>`` and ``<longitude>`` inherit the
  ``direction`` slot from their superclass ``<directed-angle>``. In
  `Virtual slots`_, we show how to provide a setter
  method for the direction slot of ``<latitude>`` that ensures that the
  value is north or south, and a setter method for the direction slot
  of ``<longitude>`` that ensures that the value is east or west.
- A slot access can involve arbitrary computation. For example, a slot
  can be *virtual*. See `Virtual slots`_.

.. _slots-setter-methods:

Setter methods
~~~~~~~~~~~~~~

In most cases, the getter and setter methods that Dylan defines for each
slot are perfectly adequate. In certain cases, however, you might want
to change the way a getter or setter works.

For example, we can define a setter method to solve a problem in our
time library. The class ``<time-of-day>`` inherits the ``total-seconds``
slot from the class ``<sixty-unit>``. The type of the slot is ``<integer>``
. However, the semantics of ``<time-of-day>`` state that the
``total-seconds`` should not be less than 0. We can define a setter method
for ``<time-of-day>`` to ensure that the new value for the total-seconds
slot is 0 or greater.

In our setter method, we will use the type defined in
:ref:`classes-examples-types-not-classes`, and repeated here:

.. code-block:: dylan

    // Define nonnegative integers as integers that are >= zero
    define constant <nonnegative-integer> = limited(<integer>, min: 0);

The setter method is as follows:

.. code-block:: dylan

    define method total-seconds-setter
        (total-seconds :: <integer>, time :: <time-of-day>)
     => (total-seconds :: <nonnegative-integer>)
      if (total-seconds >= 0)
        next-method();
      else
        error("%d is invalid. total-seconds cannot be negative.",
              total-seconds);
      end if;
    end method total-seconds-setter;

When the setter for the ``total-seconds`` slot is called with an instance
of ``<time-of-day>``, the preceding method will be invoked, because it is
more specific than the method that Dylan generated on the ``<sixty-unit>``
class. If the new value for the ``total-seconds`` slot is valid (that is,
is greater than or equal to 0), then this method calls ``next-method``,
which invokes the setter method on ``<sixty-unit>``. If the new value is
less than 0, an error is signaled.

The following example show what happens when you call
``total-seconds-setter`` with a negative value for ``total-seconds``:

.. code-block:: dylan-console

    ? begin
        let test-time-of-day = make(<time-of-day>);
        test-time-of-day.total-seconds := -15;
      end;
    => ERROR: -15 is invalid. total-seconds cannot be negative.

This setter method ensures that no one can assign an invalid value to
the slot. For completeness, we must also ensure that no one can
initialize the slot to an invalid value. The way to do that is to define
an ``initialize`` method, as shown in `Initialize methods`_.

Considerations for naming slots and other objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A *binding* is an association between a name and an object. For example,
there is a binding that associates the name of a constant and the value
of the constant. The names of functions, module variables, local
variables, and classes are also bindings. There is a potential problem
that can occur if you use short names. If a client module uses other
modules that also define and export bindings with short names, there is
a significant chance that name clashes will occur, with different
bindings with the same name being imported from different modules.

If you use the Dylan naming conventions, then a variable will not have
the same name as a class, a function, or a constant. The naming
conventions avoid name clashes between different kinds of objects.

A slot is identified by the name of its getter. The getter is visible to
all client modules. There is no problem if two getters with the same
name are defined by unrelated classes, because the appropriate getter is
selected through method dispatch. There is a problem if a getter has the
same name as a generic function with an incompatible parameter list or
values declaration. (See :ref:`func-parameter-list-congruence`.) When
such a problem occurs, the only way to resolve it is to use options to
``define module`` to exclude or rename some of the problem bindings. This
solution is undesirable, because it requires work on the part of the author
of the client module, who must spot and resolve such clashes, and then use
an interface that no longer matches its documentation.

Therefore, for getters that you intend to export, it makes sense prevent
clashes by considering the name of the slot carefully. One technique is
to prefix the name of the property with the name of the class. For
example, you might define a ``<person>`` class with a slot ``person-name``,
instead of the shorter possibility, ``name``. One drawback of this
technique is that it might expose too much information about the
implementation — that is, the name betrays the class that happens to
implement the slot at a particular time, and you have to remember which
superclass introduces a property if you are to access that property.

There is a compromise between using short names and using the class name
as a prefix — you can choose a prefix for a whole group of classes
beneath a given class. For example, you might use the prefix ``person-``
for slots of many classes that inherit from the ``<person>`` class,
including ``<employee>``, ``<consultant>``, and so on.

.. code-block:: dylan

    define class <person> (<object>)
      slot person-name;
      slot person-age;
    end class <person>;

    define class <employee> (<person>)
      slot person-number;
      slot person-salary;
    end class <employee>;

    define class <consultant> (<employee>)
      slot person-perks;
      slot person-parking-lot;
    end class <consultant>;

Now, in a method on ``<consultant>``, all accesses are consistent, and we
do not have to remember where the slots actually originate:

.. code-block:: dylan

    // Method 1
    define method person-status (p :: <consultant>) => (status :: <integer>)
      (p.person-perks.evaluation + p.person-salary.evaluation)
        / p.person-age;
    end method person-status;

If we had defined the classes differently, such that we prefixed each
getter with the name of the class that defined it, the method would look
like this:

.. code-block:: dylan

    // Method 2
    define method person-status (p :: <consultant>) => (status :: <integer>)
      (p.consultant-perks.evaluation + p.employee-salary.evaluation)
        / p.person-age;
    end method person-status;

Method 2 is more difficult to write and read than is Method 1, and is
more fragile. If, at some point, all employees are allocated perks, then the
use of the ``consultant-perks`` getter becomes a problem.

.. topic:: Comparison with C++:

   In C++, the class is the namespace of its member functions. In Dylan,
   the module is the namespace of getters and setters.  In general, the
   module is the namespace of all module bindings, including generic
   functions; getters and setters are generic functions.

.. _slots-initialize-methods:

Initialize methods
------------------

Every time you call ``make`` to create an instance of a class,
``make`` calls the ``initialize`` generic function. The purpose
of the ``initialize`` generic function is to initialize the
instance before it is returned by ``make``. You can customize the
initialization by defining a method on ``initialize``. Methods for
``initialize`` receive the instance as the first argument, and receive
all keyword arguments given in the call to ``make``.

We define an ``initialize`` method:

.. code-block:: dylan

    define method initialize (time :: <time-of-day>, #key)
      next-method();
      if (time.total-seconds < 0)
        error("%d is invalid. total-seconds cannot be negative",
              time.total-seconds);
      end if;
    end method initialize;

On line 2, we call ``next-method``. All methods for ``initialize`` should
call ``next-method`` as their first action, to allow any less specific
initializations (that is, ``initialize`` methods defined on superclasses)
to execute first. If you call ``next-method`` as the first action, then,
in the rest of the method, you can operate on an instance that has been
properly initialized by any ``initialize`` methods of superclasses. If you
forget to include the call to ``next-method``, your ``initialize`` method
will be operating on an improperly initialized instance.

Lines 3 through 6 contain the real action of this method. We check that
the value is valid. If it is invalid, we signal an error.

The following example shows what happens when ``total-seconds`` is not
valid when we are creating an instance:

.. code-block:: dylan-console

    ? make(<time-of-day>, total-seconds: -15);
    => ERROR: -15 is invalid. total-seconds cannot be negative.

Slot options for initialization of slots
----------------------------------------

Unlike variables and constants, slots can be *uninitialized*; that is,
you can create an instance without initializing all the slots. If you
call a getter for a slot that has not been initialized, Dylan signals an
error. In the following sections, we describe a variety of techniques
for avoiding the problem of accessing an uninitialized slot. The most
general technique is to define an ``initialize`` method for a slot, as
shown in `Initialize methods`_.

A slot can be uninitialized. Once a slot receives a value, however, it
will always have a value: There is no way to return a slot to the
uninitialized state. Sometimes it is useful to store in a slot a value
that means none. To make that possible, you need to define a new type
for that slot, as shown in :ref:`classes-examples-types-not-classes`. In Sections
`The init-value: slot option`_ through `The init-function: slot option`_,
we show techniques for initializing slots.

The ``init-value:`` slot option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can use the ``init-value:`` slot option to give a default initial value
to a slot:

.. code-block:: dylan

    define abstract class <sixty-unit> (<object>)
      slot total-seconds :: <integer>,
        init-keyword: total-seconds:, init-value: 0;
    end class <sixty-unit>;

When we use ``make`` to create any subclass of ``<sixty-unit>`` (such as
``<time-of-day>``), and we do not supply the ``total-seconds:`` keyword to
``make``, the ``total-seconds`` slot is initialized to 0.

The ``init-value:`` slot option specifies an expression that is evaluated
once, before the first instance of the class is made, to yield a value.
Every time that an instance is made and the slot needs a default value,
this same value is used as the default.

In general, a slot receives its default initial value when no init
keyword is defined or when the caller does not supply the init-keyword
argument to ``make``.

The ``required-init-keyword:`` slot option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of giving the slot a default initial value, we can require the
caller of ``make`` to supply an init keyword for the slot. The
``required-init-keyword:`` slot option defines a required init keyword. If
the caller of ``make`` does not supply the required init keyword, then an
error is signaled.

.. code-block:: dylan

    define abstract class <sixty-unit> (<object>)
      slot total-seconds :: <integer>, required-init-keyword: total-seconds:;
    end class <sixty-unit>;

The ``total-seconds`` slot is defined in the ``<sixty-unit>`` class. By
making ``total-seconds:`` a required init keyword in this class, we make
it required for every class that inherits from it, including ``<time>``,
``<angle>``, and all their subclasses.

Slot options for an inherited slot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can define a slot in only one particular class in a set of classes
related by inheritance. You can use the ``inherited slot`` specification
to override the default initial value of an inherited slot, or the *init
function* of an inherited slot. See `The init-function: slot option`_.

In this example, assume that the ``<sixty-unit>`` class defines the
``total-seconds`` slot and the init keyword ``total-seconds:``, and
provides the default initial value of 0 for that slot, as shown:

.. code-block:: dylan

    define abstract class <sixty-unit> (<object>)
      slot total-seconds :: <integer>,
        init-keyword: total-seconds:, init-value: 0;
    end class <sixty-unit>;

    define abstract class <time> (<sixty-unit>)
    end class <time>;

The ``<time-offset>`` class provides a different default initial value for
the inherited slot ``total-seconds``:

.. code-block:: dylan

    define class <time-offset> (<time>)
      inherited slot total-seconds, init-value: encode-total-seconds(1, 0, 0);
    end class <time-offset>;

By using the ``inherited slot`` specification, we are not defining the
slot, but rather are stating that this slot is defined by a superclass.
We can then provide either a default initial value or an init function
for the inherited slot.

The ``init-function:`` slot option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can use the ``init-function:`` slot option to provide a function of no
arguments to be called to return a default initial value for the slot.
These functions are called *init functions*. They allow the initial
value of a slot to be an arbitrary computation.

.. code-block:: dylan

    define class <time-of-day> (<time>)
      inherited slot total-seconds, init-function: get-current-time;
    end class <time-of-day>;

Every time that we make an instance of the ``<time-of-day>`` class and we
need a default value for the ``total-seconds`` slot, the ``get-current-time``
function is called to provide an initial value. Here, we assume that
``get-current-time`` is available as a library function; it is not part
of the core Dylan language.

The ``init-function:`` slot option specifies an expression that is
evaluated once, before the first instance of the class is made, to yield
a function. The function must have no required arguments and must return
at least one value. Every time that an instance is made and the slot
needs a default value, this function is called with no arguments, and
the value that it returns is used as the default. An init function is
called during instance creation when no keyword argument is defined or
when an optional keyword argument is not passed to ``make``.

Init expressions
~~~~~~~~~~~~~~~~

An *init expression* is another way of providing a default slot value.
Here is an example:

.. code-block:: dylan

    define class <time-of-day> (<time>)
      inherited slot total-seconds = get-current-time();
    end class <time-of-day>;

Every time that we make an instance of the ``<time-of-day>`` class and we
need a default value for the ``total-seconds`` slot, the expression
``get-current-time();`` is evaluated to provide an initial value.

An init expression specifies an expression. Every time that an instance
is made and the slot needs a default value, this expression is evaluated
and its value is used as the default.

Notice the similarity between the ``init-function:`` slot option and an
init expression. In fact, the following slot specifications are
equivalent:

.. code-block:: dylan

    inherited slot total-seconds, init-function: get-current-time;
    inherited slot total-seconds = get-current-time();

That substitution works for functions that have no required arguments.
More generally, the following slot specifications are equivalent:

.. code-block:: dylan

    slot slot = expression;
    slot slot, init-function: method () expression end method;

The expression can be a call to a function that requires arguments.
Here, we use *method* to define a method with no name.

The ``init-value:`` slot option, ``init-function:`` slot option, and init
expression are mutually exclusive. A given slot specification can have
only one of these.

Allocation of slots
-------------------

Each slot has a particular kind of *allocation*. The allocation of a
slot determines where the storage for the slot’s value is allocated, and
it determines which instances share the value of the slot. There are
four kinds of allocation:

.. index::
   single: slot allocation; instance

Instance:
  Each instance allocates storage for the slot, and each
  instance of the class that defines the slot has its own value for the
  slot. Changing a slot in one instance does not affect the value of
  the same slot in a different instance. Instance allocation is the
  default, and is the most commonly used kind of allocation.

.. index::
   single: slot allocation; virtual
   single: virtual slot allocation

Virtual:
  No storage is allocated for the slot. You must provide a
  getter method that computes the value of the virtual slot. See
  `Virtual slots`_.

.. index::
   single: slot allocation; class

Class:
  The class that defines the slot allocates storage for the slot.
  Instances of the class that defines the slot and instances of all
  that class’s subclasses see the same value for the slot. That is, all
  general instances of the class share the value for the slot.

.. index::
   single: slot allocation; each-subclass

Each-subclass:
  The class that defines the slot and each of its subclasses allocate
  storage for the slot. Thus, if the class that defines the slot has
  four subclasses, the slot is allocated in five places. All the direct
  instances of each class share a value for the slot.

We can give an example of an each-subclass slot by defining a
``<vehicle>`` class:

.. code-block:: dylan

    define class <vehicle> (<physical-object>)
      // Every vehicle has a unique identification code
      slot vehicle-id :: <string>, required-init-keyword: id:;
      // The normal operating speed of this class of vehicle
      each-subclass slot cruising-speed :: <integer>;
    end class <vehicle>;

The slot ``cruising-speed`` is defined with the ``each-subclass`` slot
allocation. We use ``each-subclass`` allocation to express that, for
example, all instances of Boeing 747 aircraft share a particular
cruising speed, and all instances of McDonnell Douglas MD-80 aircraft
share a particular cruising speed, but the cruising speed of 747s does
not need to be the same as the cruising speeds of MD-80s.

.. index::
   single: virtual slot
   single: virtual slot allocation
   single: slot; virtual

.. _slots-virtual-slots:

Virtual slots
-------------

Virtual slots are useful when there is information conceptually
associated with an object that is better computed than stored in an
ordinary slot. By using a virtual slot instead of writing a method, you
make the information appear like a slot to the callers of the getter.
The information appears like a slot because the caller cannot
distinguish the getter of a virtual slot from a getter of an ordinary
slot. In both cases, the getter takes a single required argument — the
instance — and returns a single value.

A virtual slot does not occupy storage; instead, its value is computed.
When you define a virtual slot, Dylan defines a generic function for the
getter and setter. You must define a getter method to return the value
of the virtual slot. Unlike those of other slots, the value of a virtual
slot can change without a setter being called, because that value is
computed, rather than stored. You can optionally define a setter method.
If you want to initialize a virtual slot when you create an instance,
you can define an ``initialize`` method.

We can use virtual slots to control the access to a slot. For example,
we want to ensure that the value of the ``direction`` slot is north or
south for ``<latitude>``, and is east or west for ``<longitude>``. (An
alternative technique is to use enumeration types, as shown in
:ref:`perform-enumerations`.) To enforce this restriction, we must

- Check the value when the setter method is invoked. In this section,
  we show how to do this check using a virtual slot. We also show how
  to use symbols, instead of strings, to represent north, south, east,
  and west.
- Check the value of the ``direction`` slot when an instance is created
  and initialized. We do that checking in `Initialize method for a
  virtual slot`_.

We redefine the ``<directed-angle>`` class to include a virtual slot and
an ordinary slot:

.. code-block:: dylan

    define abstract class <directed-angle> (<angle>)
      virtual slot direction :: <symbol>;
      slot internal-direction :: <symbol>;
    end class <directed-angle>;

We define the slot ``direction`` with the *virtual slot allocation*.
Notice that the slot’s allocation appears before the name of the slot
(as contrasted with slot options, which appear after the name of the
slot).

In the ``<directed-angle>`` class, we use the slot ``internal-direction``
to store the direction. We shall provide a setter method for the virtual
slot ``direction`` that checks the validity of the value of the direction
before storing the value in the ``internal-direction`` slot.

Symbols
~~~~~~~

Symbols are much like strings. A *symbol* is an instance of the built-in
class ``<symbol>``. The key difference between strings and symbols lies in the
way similarity (as tested by ``=`` ) and identity (as tested by ``==`` ) are
defined for each of them. Two string operands can be similar but not
identical. However, two symbol operands that are similar are always
identical — that is, they always refer to the same object.

There are two reasons to use symbols in certain cases where you might
consider using strings. First, symbol comparison is not case sensitive.
Second, comparison of two symbols is much faster than is comparison of
two strings, because symbols are compared by identity, and strings are
usually compared element by element.

In the ``<directed-angle>`` class, we define the type of the two slots as
``<symbol>``, instead of ``<string>``, which we used in previous versions
of this class. If we use strings, then when we checked whether the
direction slot of a latitude was ``"north"`` or ``"south"``, we would have
to worry about uppercase versus lowercase. For example, we would have to
decide whether each of these were valid values: ``"north"``, ``"NORTH"``,
``"North"``, ``"NOrth"``, and so on. We simplify that decision by using
the ``<symbol>`` type instead of ``<string>``.

There are two equivalent syntaxes for specifying symbols:

- Examples of use of the keyword syntax are: ``north:`` and ``south:``.
- Examples of use of the hash syntax are:``#"north"`` and ``#"south"``.

Here, we show that symbol comparison is not case sensitive:

.. code-block:: dylan-console

    ? #"NORTH" == #"North";
    => #t

Here, we show that the two syntaxes are equivalent:

.. code-block:: dylan-console

    ? north: == #"norTH";
    => #t

It is our convention in this book to reserve the keyword syntax for
keyword parameters, and otherwise to use the hash syntax. For example,
we would give the call:

.. code-block:: dylan

    make(<latitude>, direction: #"north")

instead of the call:

.. code-block:: dylan

    make(<latitude>, direction: north:)

Getter and setter methods for a virtual slot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is the getter method for the virtual slot ``direction``:

.. code-block:: dylan

    // Method 1
    define method direction (angle :: <directed-angle>) => (dir :: <symbol>)
      angle.internal-direction;
    end method direction;

Here are the setter methods for the virtual slot ``direction``:

.. code-block:: dylan

    // Method 2
    define method direction-setter
        (dir :: <symbol>, angle :: <directed-angle>) => (new-dir :: <symbol>)
      angle.internal-direction := dir;
    end method direction-setter;

    // Method 3
    define method direction-setter
        (dir :: <symbol>, latitude :: <latitude>) => (new-dir :: <symbol>)
      if (dir == #"north" | dir == #"south")
        next-method();
      else
        error("%= is not north or south", dir);
      end if;
    end method direction-setter;

    // Method 4
    define method direction-setter
        (dir :: <symbol>, longitude :: <longitude>) => (new-dir :: <symbol>)
      if (dir == #"east" | dir == #"west")
        next-method();
      else
        error("%= is not east or west", dir);
      end if;
    end method direction-setter;

The preceding methods work as follows:

- When you call ``direction`` on an instance of ``<directed-angle>`` or any
  of its subclasses, method 1 is invoked. Method 1 calls the getter
  ``internal-direction``, and returns the value of the
  ``internal-direction`` slot.
- When you call ``direction-setter`` on a direct instance of ``<latitude>``,
  method 3 is invoked. Method 3 checks that the direction is valid
  for latitude; if it finds that the direction is valid, it calls
  ``next-method``, which invokes method 2. Method 2 stores the direction
  in the ``internal-direction`` slot.
- When you call ``direction-setter`` on a direct instance of
  ``<longitude>``, method 4 is called. Method 4 checks that the
  direction is valid for longitude; if it finds that the direction is
  valid, it calls ``next-method``, which invokes method 2. Method 2
  stores the direction in the ``internal-direction`` slot.
- When you call ``direction-setter`` on a direct instance of
  ``<directed-angle>``, method 2 is invoked. Method 2 stores the
  direction in the ``internal-direction`` slot.

In these methods, we use ``dir``, rather than ``direction``, as the name
of the parameter that represents direction. Recall that ``direction`` is
the name of a getter. Although we technically could use ``direction`` as
the parameter name in these methods (because we do not call the
``direction`` getter in the bodies), ``direction`` as a parameter name might
be confusing to other people reading the code.

The ``error`` function signals an error. For more information about
signaling and handling errors, see :doc:`exceptions`.

The ``direction-setter`` methods check the direction when the setter is
called. In `Initialize method for a virtual slot`_, we check the direction
when an instance is made.

Initialize method for a virtual slot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We define the ``initialize`` method:

.. code-block:: dylan
   :linenos:

    define method initialize (angle :: <directed-angle>, #key direction: dir)
      next-method();
      angle.direction := dir;
    end method initialize;

For keyword parameters, the name of the keyword that you supply to
``make`` is normally the same name as the parameter that is initialized
within the body. In this case, we want to avoid confusion between the
getter ``direction`` and the keyword parameter ``direction:``, so we use
``dir`` as the name of the keyword parameter for the ``initialize`` method.
When you call ``make``, you use the ``direction:`` keyword. However,
within this method, the parameter is named ``dir``.

Line 3 calls the setter for the ``direction`` slot. We defined the methods
for ``direction-setter`` in `Getter and setter methods for a virtual
slot`_. If the argument is a latitude, then method 3 is invoked to check
the value. If the argument is a longitude, then method 4 is invoked to
check the value.

We can create a new instance of ``<absolute-position>``.

.. code-block:: dylan-console

    ? define variable *my-absolute-position* =
        make(<absolute-position>,
             latitude:
               make(<latitude>,
                    total-seconds: encode-total-seconds(42, 19, 34),
                    direction: #"north"),
             longitude:
               make(<longitude>,
                    total-seconds: encode-total-seconds(70, 56, 26),
                    direction: #"west"));

The preceding example works, because the values for direction are
appropriate for latitude and longitude. The following example shows what
happens when the direction is not valid when an instance is created:

.. code-block:: dylan-console

    ? make(<latitude>, direction: #"nooth");
    => ERROR: nooth is not north or south

The following example shows what happens when the direction is not valid
when the ``direction`` setter is used:

.. code-block:: dylan-console

    ? begin
        let my-longitude = make(<longitude>, direction: #"east");
        my-longitude.direction := #"north";
      end;
    => ERROR: north is not east or west

Summary
-------

In this chapter, we covered the following:

- We described techniques for initializing slots; see
  :ref:`summary-slot-initialization`.
- We discussed the syntax of calling getters and setters; see
  :ref:`syntax-calling-getters-setters`.
- We showed how to define methods for getters and setters.
- We showed how and why you can use symbols instead of strings.
- We described the different kinds of slot allocation; see
  :ref:`summary-slot-allocations`.

.. _summary-slot-initialization:

.. table:: Summary of slot-initialization techniques

   +-----------------------+-----------------------------------------------------------+
   | Technique             | Summary                                                   |
   +=======================+===========================================================+
   | ``initialize`` method | You can define a method for ``initialize`` for a class to |
   |                       | perform any actions to initialize the instance. The       |
   |                       | ``make`` function calls the ``initialize`` generic        |
   |                       | function after ``make`` creates an instance and supplies  |
   |                       | those initial slot values that it can. If you need to do  |
   |                       | any complex computation to determine and set the value of |
   |                       | a slot, you can do it in an ``initialize`` method.        |
   +-----------------------+-----------------------------------------------------------+
   | Init keyword          | You can use the ``init-keyword:`` slot option to declare  |
   |                       | an optional keyword argument, or the                      |
   |                       | ``required-init-keyword:`` slot option to declare a       |
   |                       | required keyword argument for ``make`` when you create an |
   |                       | instance of the class. The value of the keyword argument  |
   |                       | becomes the value of the slot.                            |
   +-----------------------+-----------------------------------------------------------+
   | Init value            | You can use the ``init-value:`` slot option to give a     |
   |                       | default initial value for the slot. This option specifies |
   |                       | an expression that is evaluated once, before the first    |
   |                       | instance of the class is made, to yield a value. Every    |
   |                       | time an instance is made and the slot needs a default     |
   |                       | value, this same value is used as the default. The slot   |
   |                       | receives its default initial value when no init keyword   |
   |                       | is defined, or when the caller does not supply the        |
   |                       | init-keyword argument to ``make``.                        |
   +-----------------------+-----------------------------------------------------------+
   | Init function         | You can use the ``init-function:`` slot option to provide |
   |                       | a function that returns a default value. This option      |
   |                       | specifies an expression that is evaluated once, before    |
   |                       | the first instance of the class is made, to yield a       |
   |                       | function. The function must have no required arguments    |
   |                       | and must return at least one value. Every time that an    |
   |                       | instance is made and the slot needs a default value, this |
   |                       | function is called with no arguments, and the value that  |
   |                       | it returns is used as the default. The slot receives its  |
   |                       | default initial value when no init keyword is defined or  |
   |                       | when the caller does not supply the init-keyword argument |
   |                       | to ``make``.                                              |
   +-----------------------+-----------------------------------------------------------+
   | Init expression       | You can use an init expression to provide an expression   |
   |                       | that yields a default value. Every time that an instance  |
   |                       | is made and the slot needs a default value, this          |
   |                       | expression is evaluated, and its value is used as the     |
   |                       | default. The slot receives its default initial value when |
   |                       | no init keyword is defined, or when the caller does not   |
   |                       | supply the init-keyword argument to ``make``.             |
   +-----------------------+-----------------------------------------------------------+

.. _syntax-calling-getters-setters:

.. table:: Syntax of calling getters and setters

   +-------------------------------------------+-------------------------------------------------+
   | Call                                      | Translation                                     |
   +===========================================+=================================================+
   | ``object.function-name``                  | ``function-name(object)``                       |
   +-------------------------------------------+-------------------------------------------------+
   | ``*my-time-of-day*.total-seconds;``       | ``total-seconds(*my-time-of-day*);``            |
   +-------------------------------------------+-------------------------------------------------+
   | ``object.name := new-value;``             | ``name-setter(new-value, object);``             |
   +-------------------------------------------+-------------------------------------------------+
   | ``name(object) := new-value;``            | ``name-setter(new-value, object);``             |
   +-------------------------------------------+-------------------------------------------------+
   | ``*my-time-of-day*.total-seconds := 0;``  | ``total-seconds-setter (0, *my-time-of-day*);`` |
   +-------------------------------------------+-------------------------------------------------+
   | ``total-seconds(*my-time-of-day*) := 0;`` | ``total-seconds-setter(0, *my-time-of-day*);``  |
   +-------------------------------------------+-------------------------------------------------+

.. _summary-slot-allocations:

.. table:: Summary of slot allocations

   +---------------+--------------------------------------------------------+
   | Allocation    | Summary                                                |
   +===============+========================================================+
   | Instance      | Each instance allocates storage for the slot, and each |
   |               | instance of the class that defines the slot has its    |
   |               | own value of the slot. Instance allocation is the      |
   |               | default.                                               |
   +---------------+--------------------------------------------------------+
   | Virtual       | No storage is allocated for the slot. You must provide |
   |               | a getter method that computes the value of the virtual |
   |               | slot.                                                  |
   +---------------+--------------------------------------------------------+
   | Class         | The class that defines the slot allocates storage for  |
   |               | the slot. All general instances of the class share the |
   |               | value of the slot.                                     |
   +---------------+--------------------------------------------------------+
   | Each-subclass | The class that defines the slot and each of its        |
   |               | subclasses allocate storage for the slot. All the      |
   |               | direct instances of each class share the value of the  |
   |               | slot.                                                  |
   +---------------+--------------------------------------------------------+
