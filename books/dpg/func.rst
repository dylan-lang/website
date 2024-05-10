Functions
=========

Functions are ubiquitous in Dylan. Generic functions and methods — the
two kinds of function — are the primary means of specialization. Many
common operations, such as slot references and arithmetic operations,
are accomplished through function calls. In Dylan, unlike in many
languages, functions are first-class objects. They can be the values of
variables or slots, arguments to other functions, or values returned by
functions. Dylan has functions that build new functions out of existing
functions. Much of the power of Dylan arises through its sophisticated
treatment of functions.

This chapter discusses general aspects of the operation of functions in
Dylan. It does not describe all aspects of functions. In particular, we
discuss the process of method dispatch within generic functions
elsewhere (see Sections :ref:`offset-method-dispatch`, :ref:`multi-method-dispatch`,
:ref:`classes-method-dispatch-nonclass-types`, and :ref:`inherit-mi-and-md`).
This chapter covers three main topics:

#. The syntax of function calls, including abbreviations for function
   calls
#. The function-calling protocol, and particularly the interaction
   between a function and its caller
#. The uses of functions as objects, including ways of creating and
   operating on functions

Function-calling syntax
-----------------------

This section describes the syntax of Dylan function calls. An explicit
function call consists of the operand followed by the arguments enclosed
in parentheses and separated by commas. Several other syntactic
structures in Dylan are also abbreviations for function calls, including
the following:

- Slot references
- References to elements of collections
- Most unary and binary operator calls
- Certain assignment operations

The remainder of this section describes these syntactic forms and the
equivalent function calls. Unless otherwise noted, all expressions that
make up any of these function calls are evaluated from left to right. (A
notable exception is an expression containing the assignment operator,
discussed in `Assignment`_.) The common left-to-right rule makes it
easy to understand the order of execution of Dylan code. But it also
means that certain syntactic forms that we call *equivalent* — that is,
syntactic forms that generally result in calls to the same function
with the same arguments — differ in the order of evaluation of their
components. The components can appear in different orders in otherwise
equivalent syntactic forms. Usually, the order of evaluation makes no
difference, and you can use whichever of the equivalent syntactic
forms you find most convenient.

Explicit function calls
~~~~~~~~~~~~~~~~~~~~~~~

The Dylan syntax for an explicit function call has two parts:

#. The function to be called — This is an *operand* that is evaluated to
   yield the function itself. Usually, the operand is a reference to a
   variable or constant that names the function, although it can be any
   expression (except an operator call) whose value is a function. (For
   information on operator calls, see Sections `Unary operator
   calls`_ and `Binary-operator calls`_.)
#. The arguments to which the function is applied — The arguments are
   represented by a series of expressions, enclosed in parentheses and
   separated by commas. Each expression is evaluated, and its value is
   passed to the function as an argument.

In the following function call, the function is the value of the
variable ``truncate/``; the two arguments are the value of the variable
``n`` and the number ``3``:

.. code-block:: dylan

    truncate/(n, 3);

A function can be obtained in other ways: for example, it might be an
element of an array, the value of a slot of an instance, or the value
returned by a call to another function. The following example calls the
function that is the element of an ``operations`` array designated by the
constant ``$trunc``:

.. code-block:: dylan

    operations[$trunc](n, 3);

.. _func-slot-references:

Slot references
~~~~~~~~~~~~~~~

A slot reference is a reference to the value of a slot of an instance.
The syntax for a slot reference has two parts, separated by a period:

- An operand whose value is the instance
- The name of the slot’s getter generic function

In the following slot reference, the function ``get-employee-named``
returns an instance, which has a slot whose getter is named
``employee-number``:

.. code-block:: dylan

    get-employee-named("Jane").employee-number;

Note that the operand that yields the instance can itself be a slot
reference, so slot references can be chained:

.. code-block:: dylan

    plant.manager.employee-number;

Every slot value in Dylan is obtained by a call to the slot’s getter
generic function (although the compiler can often optimize this generic
function call to a direct slot access). A slot reference is just an
abbreviation for a function call. With one exception, the following
examples are equivalent:

.. code-block:: dylan

    plant.manager;
    manager(plant);

The one difference between these examples is that, in the first, ``plant``
is evaluated first, whereas in the second, ``manager`` is evaluated first.

In fact, you can use the slot-reference syntax for more than slot
references. The object that is the value of the left side can be any
object, and the function named by the right side can be any function
that can take the object as an argument. The function named by the
right side is always called with the object that is the value of the
left side as its only argument. Thus, using the ``plant.manager``
syntax is just another way of calling the function named by
``manager`` with the object that is the value of ``plant`` as the only
argument. The ``plant`` object does not have to have a ``manager`` slot.

In this book, we use slot-reference syntax for

- A call to a getter generic function for a slot
- A call to a function that takes one argument and returns one value
  that represents a property of an object

.. _func-element-references:

Element references
~~~~~~~~~~~~~~~~~~

Collections in Dylan include such data structures as arrays, strings,
lists, and tables. Each collection has a mapping from *keys* to
*elements*. Dylan’s syntax for referring to an element of a collection
has two parts:

#. An operand whose value is the collection
#. An expression, in square brackets, whose value is the key that maps
   to the desired element of the collection

If the collection is a multidimensional array, the key expression in
square brackets can be a series of expressions, separated by commas.
Each expression yields the index for one dimension of the array. (Dylan
array indices are zero based.)

The following example returns the first element of the array named by
``my-array``:

.. code-block:: dylan

    my-array[0];

An element reference, like a slot reference, is an abbreviation for a
function call. The generic function ``element`` takes a collection and a
key as arguments, and returns the element of the collection that is
associated with the given key. Except for the order of evaluation, the
following examples are equivalent:

.. code-block:: dylan

    my-array[0];
    element(my-array, 0);

For arrays of more than one dimension, the key expression in brackets is
instead a comma-separated series of expressions. In this case, the
element reference is an abbreviation for a call to the ``aref`` generic
function. This function takes an array and any number of indices as
arguments, and returns the element associated with the array indices.
Except for the order of evaluation, the following examples are
equivalent:

.. code-block:: dylan

    my-array[0, 2];
    aref(my-array, 0, 2);

Unary operator calls
~~~~~~~~~~~~~~~~~~~~

Dylan has two built-in unary operators, ``-`` and ``~``. The syntax for a
unary operator call has two parts:

#. The operator
#. An operand

The ``-`` operator performs the arithmetic negation of its operand, and
the ``~`` operator performs the logical negation. Both operator calls are
abbreviations for function calls. The following examples are equivalent:

.. code-block:: dylan

    - time-offset;
    negative(time-offset);

The following examples also are equivalent:

.. code-block:: dylan

    ~ test-condition(cond);
    \~(test-condition(cond));

In the preceding example, we must escape ``~`` with ``\`` so that Dylan
interprets ``~`` as a variable name, instead of as an operator. This
syntax indicates an explicit call to the function that is the value of
the variable named ``~``.

Binary-operator calls
~~~~~~~~~~~~~~~~~~~~~

Dylan has 16 built-in binary operators, of the following kinds:

- Arithmetic operations: ``+``, ``-``, ``*``, ``/``, and ``^``
- Comparisons: ``=``, ``==``, ``<``, ``>``, ``<=``, ``>=``, ``~=``, and ``~==``
- Logical operations: ``&`` and ``|``
- Assignment: ``:=``

The syntax for a binary-operator call has three parts:

#. An expression that serves as the first operand
#. The operator
#. An expression that serves as the second operand

All binary-operator calls, except those to the logical and assignment
operators, are abbreviations for calls to functions that have the same
names as do the operators. Except for the order of evaluation, the
following examples are equivalent:

.. code-block:: dylan

    a + b;
    \+(a, b);

The ``&`` and ``|`` operators are implemented as *macros*. (For
information on macros, see :doc:`macros`.) In an expression
that includes the ``&`` operator, if the first operand has a false value,
the second operand is not evaluated. In an expression that includes the
``|`` operator, if the first operand has a true value, the second operand
is not evaluated.

.. _func-assignment:

Assignment
~~~~~~~~~~

The assignment binary operator, ``:=``, also is implemented as a macro.
An expression that includes this operator works in a special way.

The operand to the *right* of the operator is evaluated first. The
result is the new value to be assigned.

The operand to the *left* of the operator determines the place to which
the new value is assigned. This operand can have one of the following
kinds of syntax:

Variable name
   The variable name is not evaluated. Dylan assigns the new value
   to the variable.
Explicit function call
   Dylan calls the function *name* *-setter*, where *name* is the
   name of the function in the function call. The first argument
   to *name* *-setter* is the new value, and the remaining arguments
   are the arguments to *name* in the original function call.
Slot reference
   Dylan first converts the slot reference to the corresponding
   function call. Dylan then calls the function *name* *-setter*
   just as it would have if the slot reference had been an
   explicit function call.
Element reference
   Dylan first converts the element reference to the corresponding
   function call, using ``element`` or ``aref`` as the name of
   the function, as appropriate. Dylan then calls the function
   ``element-setter`` or ``aref-setter`` just as it would have if the
   element reference had been an explicit function call.

Except for the order of evaluation and returned values, the following
examples are equivalent:

.. code-block:: dylan

    *my-position*.distance := 3.0;
    distance(*my-position*) := 3.0;
    distance-setter(3.0, *my-position*);

The first two examples return ``3.0``; the third returns whatever
``distance-setter`` returns. Usually, this value would be ``3.0``. Note that, if
``distance`` is the name of a slot’s getter, and if the slot is constant
or has a setter with a name other than ``distance-setter``, then the
assignment operation results in an error.

Except for the order of evaluation and returned values, the following
examples are equivalent:

.. code-block:: dylan

    vertices[2] := list(3.5, 4.5);
    element(vertices, 2) := list(3.5, 4.5);
    element-setter(list(3.5, 4.5), vertices, 2);

The function-calling protocol
-----------------------------

We have seen that Dylan has two kinds of function: methods and generic
functions. Both can be called; from the caller’s point of view, the two
are called in the same way. When a generic function is called, Dylan
selects one of its methods to execute, in a process called method
dispatch. This section discusses the interaction between a function and
that function’s caller, focusing on arguments, parameters, value
declarations, and returned values. We discuss interactions between
generic functions and their methods but do not describe the process of
method dispatch. For information on method dispatch, see
:ref:`offset-method-dispatch`; :ref:`multi-method-dispatch`;
:ref:`classes-method-dispatch-nonclass-types`; and :ref:`inherit-mi-and-md`.

Parameters, arguments, and return values
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In Dylan, a function is called with zero or more *arguments*. The
function can perform computations, which may have side effects. It then
*returns* zero or more *values* to its caller. Each argument and
each returned value is an object.

A function has zero or more *parameters* that determine the number and
types of arguments that the function takes. Following is a simplified
description of what happens when a function is called (for a generic
function, this description applies to the method that it invokes):

#. An implicit *body* is entered. A body establishes the scope for all
   local variables bound inside the body.
#. The parameters are matched with the arguments to the function.
#. A local variable is created with the name of each parameter.
#. Each parameter — that is, each local variable with the name of a
   parameter — is initialized, or bound, to one of the arguments. (In
   some cases, the parameter is bound to a list of arguments, or to a
   default value.)
#. The code that makes up the actual body of the function is executed.

A function can have a *value declaration* that determines the number and
types of values the function returns. If there is no explicit
declaration, a default declaration allows the function to return any
number of values of any type. Following is a simplified description of
what happens when a function returns (for a generic function, this
description applies to the method that it invokes):

#. The values returned by the last expression in the function’s implicit
   body are matched with the values declared in the value declaration.
#. The function’s implicit body is exited, ending the scope of all local
   variables (including parameters) established in that body.
#. The values specified by the value declaration are returned to the
   caller of the function. (Depending on the value declaration, the
   number of values returned to the function’s caller might be more or
   less than the number of values returned by the last expression in the
   function’s body.)

Note these two important implications of the way that arguments are
passed:

- All bindings of arguments to parameters are local to the body of the
  function called. Assignment to a parameter inside the called
  function’s body does not affect any variables outside the body that
  have the same name.

  For example, consider these definitions:

  .. code-block:: dylan

      define method calling-function ()
        let x = 1;
        let y = 2;
        format-out("In calling function, before call: x = %d, y = %d\n",
                   x, y);
        called-function(x, y);
        format-out("In calling function, after call: x = %d, y = %d\n", x, y);
      end method calling-function;

      define method called-function (x, y)
        x := 3;
        y := 4;
        format-out("In called function, before return: x = %d, y = %d\n",
                   x, y);
      end method called-function;

  A call to ``calling-function`` produces the following output::

      In calling function, before call: x = 1, y = 2
      In called function, before return: x = 3, y = 4
      In calling function, after call: x = 1, y = 2

- Although *parameters* are local to a function, all *arguments* and
  *return values* are shared between a function and its caller. If an
  argument or return value is a *mutable* object — one that can be
  changed — then any changes that a function makes to that object are
  visible to its caller.

  Consider the following definitions:

  .. code-block:: dylan

      define class <test> (<object>)
        slot test-slot, required-init-keyword: test-slot:;
      end class <test>;

      define method calling-function ()
        let x = make(<test>, test-slot: "before");
        format-out("In calling function, before call: x.test-slot = %s\n",
                   x.test-slot);
        called-function(x);
        format-out("In calling function, after call: x.test-slot = %s\n",
                   x.test-slot);
      end method calling-function;

      define method called-function (x :: <test>)
        x.test-slot := "after";
        format-out("In called function, before return: x.test-slot = %s\n",
                   x.test-slot);
      end method called-function;

  Note here that we have redefined the ``calling-function`` method, and have
  defined a new ``called-function`` method, which we first defined in the
  previous example. Our new ``called-function`` method has one parameter,
  whereas the previous method had two. The parameter list of this new
  method is not compatible with that of the previous method, and, if we
  actually tried to define the second ``called-function`` method, Dylan
  would signal an error. For more information on compatibility of
  parameter lists for generic functions and methods, see
  `Parameter-list congruence`_.

  A call to ``calling-function`` now produces the following output::

      In calling function, before call: x.test-slot = "before"
      In called function, before return: x.test-slot = "after"
      In calling function, after call: x.test-slot = "after"

  In this case, ``x`` in the calling function and ``x`` in the called function
  are different variables. But the *values* of both variables are the same
  object: the instance of ``<test>`` that we make in the calling function.
  The change to the slot value of this object that we make in the called
  function is visible to the calling function.

  It is equally proper to think of arguments that are *immutable*, like
  integers, as being shared between a function and its caller. By
  definition, however, a function cannot make any changes to such objects
  that are visible to the function’s caller.

.. topic:: Comparison with C and C++:

   As in Dylan, the parameters of a C function are local to the body of
   the function, and assignment to a parameter does not affect the value
   of a variable that has the same name in the function’s caller. But
   the relationship between *objects* and *values* is not the same in
   C and in Dylan. In C, a value can be an object (roughly meaning the
   contents of the object) or a *pointer* to an object (roughly meaning
   the location of the object in memory).  The value of a parameter in
   C is always a copy of the corresponding argument. When a C structure
   is an argument to a function, the value of the corresponding parameter
   is a copy of the structure; it is not the structure itself. If the
   function changes the value of a member of this structure, the change
   is not visible to the caller, because the function is changing only
   its own copy of the structure. But if the argument is a pointer to
   a structure, the function can gain access to the caller’s structure
   (by *dereferencing* the pointer). If the function changes the value
   of a member of such a structure by dereferencing the pointer, the
   change is visible to the caller.

   In Dylan, a value is always an object, which has a unique identity. The
   value of a parameter is always the same object as the corresponding
   argument. When a function changes such an object (as by changing the
   value of a slot), the change is always visible to the caller. Dylan has
   no equivalent to C pointers.

   In C++, a parameter declared using ordinary C syntax also receives a
   copy of a structure or an instance that is the corresponding argument.
   C++ has additional syntax for declaring that a parameter is a
   *reference* — essentially an implicit pointer — to the corresponding
   argument. In this case the argument is not copied, and if the function
   changes the object that the parameter refers to, the changes are visible
   to the caller. In some ways Dylan’s argument-passing protocol is similar
   to C++ references.

   In both C and C++, array arguments are always passed as pointers. In
   Dylan, arrays are instances of the ``<array>`` class, and array arguments
   are treated like all other arguments.

   For more comparisons between Dylan and C objects, see :doc:`c-comparisons`.

Return and reception of multiple values
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A Dylan function call — and, in general, a Dylan expression — can return
any number of values, including none. The ``values`` function is the means
of returning multiple values. This function takes zero or more
arguments, and returns them as separate values.

Multiple values can be received as the initial values of local variables
in a ``let`` declaration. If a ``let`` declaration contains multiple
variables, they are matched with the values returned by the
initialization expression, and each variable is bound to the
corresponding value. The following example initializes ``a`` to ``1`` and
``b`` to ``2``:

.. code-block:: dylan

    let (a, b) = values(1, 2);

The following example initializes ``ans`` to ``2`` and ``rem`` to ``1`` — the
two values returned by this call to ``truncate/``:

.. code-block:: dylan

    let (ans, rem) = truncate/(5, 2);

The variable list can also end with ``#rest`` followed by the name of a
variable. In this case, the variable is initialized to a sequence. This
sequence contains all the remaining values returned by the
initialization expression. If there is no ``#rest``, any excess values
are discarded. If the number of variables in the ``let`` declaration is
greater than the number of values returned, the remaining variables are
initialized to ``#f``. (But if the ``let`` declaration specifies a type for
any of these variables, and if ``#f`` is not an instance of that type,
then Dylan signals an error.)

Module variables and constants can also be initialized to multiple
values. The variable list of a ``define variable`` or ``define constant``
definition can contain multiple variables, and can receive multiple
values from its initialization expression in the same way as a ``let``
declaration.

.. _func-parameter-lists:

Parameter lists
~~~~~~~~~~~~~~~

A function’s parameter list is specified in the function definition. (If
Dylan implicitly defines a function, such as the getter and setter
functions for a slot, Dylan also defines the parameter list for that
function.) In a function definition, the parameter list follows the
function name and consists of zero or more parameter specifications,
separated by commas and enclosed in parentheses. A parameter list can
have three kinds of parameters:

#. *Required parameters* specify required arguments, or arguments that
   must be supplied when the function is called. All required parameters
   appear before other kinds of parameters in the parameter list.
#. A function can have at most one *rest parameter*, which allows the
   function to accept a variable number of arguments. The rest parameter
   is identified in the parameter list by ``#rest`` followed by the name
   of the parameter. When the function is called, all arguments that
   follow the required arguments are put into a sequence. This sequence
   is the initial value of the rest parameter in the function body.
#. *Keyword parameters* specify optional keyword arguments. In the
   parameter list, keyword parameters are identified by ``#key`` followed
   by the names of the parameters (and possibly by other information).
   Keyword parameters must follow all required parameters and the rest
   parameter (if any). When the function is called, the caller can
   supply any or none of the specified keyword arguments, in any order,
   after supplying all required arguments. The caller supplies each
   keyword argument as a symbol (usually in the form of the parameter
   name followed by a colon), followed by the argument value. This
   argument is the initial value of the corresponding keyword parameter
   in the function body.

The specification for each parameter in the parameter list includes the
name of the parameter. In addition, a required parameter (or, for a
method, a keyword parameter) can be *specialized* to correspond to an
argument of a given type. The type specializer follows the parameter
name and is identified by ``::`` followed by a type. When the function is
called, the argument that corresponds to the parameter must be of the
specified type, or Dylan signals an error. The default argument type is
``<object>``.

The specification for a keyword parameter can have two additional pieces
of information:

#. It may include a keyword for the caller to use in its argument list,
   if this keyword must be different from the parameter name. The
   keyword precedes the parameter name in the parameter list.
#. It may include a default value for the keyword argument, which is
   used if the caller does not supply that argument. The default
   expression appears at the end of the parameter specification,
   followed by ``=``. If no default expression is supplied and the
   caller does not supply the keyword argument, the argument’s
   value is ``#f``.

The following example shows how we could use a rest parameter to
implement a function to sum an arbitrary number of values:

.. code-block:: dylan

    // Sum one or more values
    define method sum (value, #rest more-values)
      for (next in more-values)
        value := value + next;
      end for;
      value;
    end method sum;

.. code-block:: dylan-console

    ? sum(3);
    => 3

    ? sum(1, 2, 3, 4, 5);
    => 15

In the preceding example, the ``for`` iteration statement performs the
addition once for every element of ``more-values``.

The following example shows how we could use keyword parameters in
defining a method similar to ``encode-total-seconds``:

.. code-block:: dylan

    // Convert days, hours, minutes, and seconds to seconds.
    // Named (keyword) arguments are optional
    define method convert-to-seconds
        (#key hours :: <integer> = 0, minutes :: <integer> = 0,
              seconds :: <integer> = 0) => (seconds :: <integer>)
      ((hours * 60) + minutes) * 60 + seconds;
    end method convert-to-seconds;

.. code-block:: dylan-console

    ? convert-to-seconds(minutes: 3, seconds: 9);
    => 189

    ? convert-to-seconds(minutes: 1, hours: 2);
    => 7260

Note from the preceding example that we can supply keyword arguments in
any order. Note also that all keyword arguments are optional; however,
if we try to call a function with a keyword argument that the function
does not accept — such as ``days:``, in this example — Dylan signals an
error. For more information on function calls and keyword arguments, see
`Keyword-argument checking`_.

Following are additional features and restrictions of keyword arguments:

- If a parameter list ends with ``#all-keys`` following ``#key``, the
  function accepts (but ignores) any keyword argument. A parameter list
  can have specific keyword parameters and also end with ``#all-keys``.
  In this case, the function accepts any keyword argument, and also has
  local variables whose values are the keyword-argument values (or
  their defaults) that correspond to the keyword parameters.
- If the parameter list of a method contains both ``#rest`` and ``#key``,
  the sequence that is the value of the rest parameter contains
  alternating symbols and argument values representing the keyword
  arguments passed to the function. In this case, *all* optional
  arguments must be keyword arguments. A generic function’s parameter
  list can have either ``#rest`` or ``#key``, but cannot have both.
- Keyword parameters for a generic function cannot be specialized.

The restrictions on a generic function’s parameter list have to do with
parameter-list congruency and keyword-argument checking in generic
function calls. For more information, see Sections `Parameter-list
congruence`_ and `Keyword-argument checking`_.

.. _func-value-declarations:

Value declarations
~~~~~~~~~~~~~~~~~~

A function definition’s value declaration follows the parameter list and
is preceded by ``=>``. The syntax of a value declaration is similar to
that of a parameter list. If the function returns no values, the value
declaration is an empty set of parentheses. Otherwise, the declaration
can contain separate declarations for all returned values, separated by
commas. Each of these individual declarations consists of a name and,
optionally, ``::`` followed by a type. The name does not specify a
variable and has no use other than documentation. But the returned value
that corresponds to the declaration must be of the declared type, or
Dylan signals an error. The default return value type is ``<object>``.

A value declaration can also end with ``#rest`` followed by a name and,
optionally, ``::`` and a type. This declaration indicates that the
function can return any number of additional arguments, each of which
must be of the specified type.

If a function has no explicit value declaration, the default declaration
is ``(#rest x :: <object>)``. This declaration indicates that the
function can return any number of arguments of any type.

The value declaration determines the number and types of values that the
function returns, even if the last expression in the function’s body
returns a different number of values. If the function’s body returns
fewer values than are declared, the function defaults the remaining
values to ``#f`` and returns them. (But if the value declaration
specifies a type for any of these values, and if ``#f`` is not an
instance of that type, Dylan signals an error.) If the function’s body
returns more values than are declared, the function returns the additional
values if the declaration contains ``#rest``; otherwise, the function
discards the additional values.

.. _func-parameter-list-congruence:

Parameter-list congruence
~~~~~~~~~~~~~~~~~~~~~~~~~

A generic function and its methods must all have parameter lists that
are compatible, or *congruent*. Following are the basic rules:

- A generic function and its methods must all have the same number of
  required arguments.
- The type of any given parameter in each method must be a subtype of
  the corresponding parameter in the generic function.
- If a generic function or any of its methods has only required
  arguments — that is, it has neither ``#rest`` nor ``#key`` in its
  parameter list — then the generic function and all its methods must
  have only required arguments.
- If a generic function or any of its methods accepts a variable number
  of arguments, but does not accept keyword arguments — that is, it has
  ``#rest``, but does not have ``#key``, in its parameter list — then the
  generic function and all its methods must accept a variable number of
  arguments, but must not accept keyword arguments.
- If a generic function or any of its methods accepts keyword arguments
  — that is, it has ``#key`` in its parameter list — then the generic
  function and all its methods must accept keyword arguments. For this
  rule, a generic function or method “accepts keyword arguments” even
  if its parameter list ends with just ``#key``.
- If a generic function has any specific keyword parameters, then all
  its methods must have (at least) those specific keyword parameters.
  The appearance of ``#all-keys`` in a method’s parameter list does not
  satisfy this requirement.

The following parameter lists are congruent, because both functions have
only required arguments, they have the same number of required
arguments, and the type of each method parameter is a subtype of the
same parameter in the generic function:

.. code-block:: dylan

    define generic g (arg1 :: <complex>, arg2 :: <integer>);

    define method g (arg1 :: <real>, arg2 :: <integer>)
      ...
    end method g;

The following parameter lists are congruent, because both functions meet
the tests for required arguments, both accept keyword arguments, and the
generic function has no specific keyword parameters:

.. code-block:: dylan

    define generic g (arg1 :: <real>, #key);

    define method g (arg1 :: <integer>, #key base :: <integer> = 10)
      ...
    end method g;

The following parameter lists are not congruent, because the method’s
parameter list does not include the specific keyword *base* of the
generic function, even though it does include ``#all-keys``:

.. code-block:: dylan

    define generic g (arg1 :: <integer>, #key base);

    define method g (arg1 :: <integer>, #key #all-keys)
      ...
    end method g;

Return-value congruence
~~~~~~~~~~~~~~~~~~~~~~~

Like parameter lists, the value declarations of a generic function and
that function’s methods must be congruent. The rules depend on whether
the generic function returns a fixed or a variable number of values:

- If the generic function returns a fixed number of values — that is,
  it does not have ``#rest`` in its value declaration — then its methods
  cannot have ``#rest``, and must return the same number of required
  values as the generic function. For each method, the type of each
  returned value must be a subtype of the same returned value in the
  generic function.
- If the generic function returns a variable number of values — that
  is, it has ``#rest`` in its value declaration — then its methods can
  (but are not required to) have ``#rest``, and must return at least as
  many required values as the generic function. For each method, the
  type of each returned value must be a subtype of the same returned
  value in the generic function. If the method has more required
  returned values than the generic function, their types must all be
  subtypes of the generic function’s ``#rest`` value.

The following value declarations are congruent, because the generic
function implicitly returns any number of values of any type:

.. code-block:: dylan

    define generic g (arg1 :: <complex>, arg2 :: <integer>);

    define method g
        (arg1 :: <real>, arg2 :: <integer>) => (result :: <real>)
      ...
    end method g;

The following value declarations are not congruent, because the type of
the method’s returned value is not a subtype of the generic function’s
returned value:

.. code-block:: dylan

    define generic g
        (arg1 :: <complex>, arg2 :: <integer>) => (result :: <integer>);

    define method g
        (arg1 :: <real>, arg2 :: <integer>) => (result :: <real>)
      ...
    end method g;

Keyword-argument checking
~~~~~~~~~~~~~~~~~~~~~~~~~

When a function is called, Dylan determines which keyword arguments, if
any, are permitted for that function call. The set of permitted keyword
arguments depends on whether or not a generic function is being called:

- If a method is called directly, rather than through a generic
  function, the specific keywords in the method’s parameter list are
  permitted. If the parameter list includes ``#all-keys``, any keyword
  argument is permitted.
- If a generic function is called, all the specific keywords in the
  parameter lists of all *applicable* methods are permitted. If the
  parameter list of the generic function or of *any* applicable method
  includes ``#all-keys``, any keyword argument is permitted.

When a generic function is called, one of its methods is *applicable* if
every required argument is an instance of the type of the corresponding
parameter of the method. For more information on applicable methods, see
:ref:`offset-method-dispatch`.

Consider the following definitions:

.. code-block:: dylan

    define generic g (arg1 :: <real>, #key);

    // Method 1
    define method g (arg1 :: <real>, #key real-key)
      ...
    end method g;

    // Method 2
    define method g (arg1 :: <float>, #key float-key)
      ...
    end method g;

    // Method 3
    define method g (arg1 :: <integer>, #key integer-key)
      ...
    end method g;

Now, if we call the generic function ``g`` with an instance of ``<float>``,
we can supply the keyword arguments ``real-key:`` and ``float-key:``,
because the methods that have those keyword parameters are both
applicable. If we call ``g`` with an instance of ``<integer>``, we can
supply the keyword arguments ``real-key:`` and ``integer-key:``.

Suppose that, in this same example, we call the generic function ``g``
with an instance of ``<float>``, and supply the keyword arguments
``real-key:`` and ``float-key:``. Method 2 is most specific, and is called
as a result of Dylan’s method dispatch. But method 2 does not have a
``real-key:`` parameter. If we were calling this method directly, Dylan
would signal an error. In this case, method 2 simply ignores the
``real-key:`` argument, because Dylan checks keyword arguments for a
generic function call as a whole, rather than for a particular method
chosen as a result of method dispatch.

There is an important subtlety of keyword-parameter specifications to
note in this example. Because of the rules for parameter-list
congruence, the generic function and all its methods must accept keyword
arguments — that is, they must all have ``#key`` in their parameter lists.
Notice that we terminated the generic function’s parameter list with
``#key``. This use indicates that the generic function permits — but does
not require — individual methods to specify keyword parameters.

Suppose that we had instead terminated the generic function’s parameter
list with ``#key, #all-keys``. This use also would have permitted, but
would not have required, individual methods to specify keyword
parameters. But it also would have allowed a caller of the generic
function to supply ``any`` keyword argument. In the earlier example, only
a small set of keyword arguments was permitted, and the members of the
set varied with the applicable methods.

In general, when you define a generic function or a method that accepts
keyword arguments, it is advisable not to specify ``#all-keys``
unnecessarily, because doing so defeats Dylan’s keyword-argument
checking. If a method needs to accept keyword arguments because of the
rules of parameter-list congruence, but does not need to recognize any
keywords itself, you should terminate its parameter list with ``#key``.

.. _func-functions-as-objects:

Functions as objects
--------------------

In Dylan, all functions are objects. A function can be the value of a
variable, an argument to another function, or a value returned by a
function. In fact, Dylan provides a number of operations on functions,
including operations to compose new functions from existing functions.

Types of functions
~~~~~~~~~~~~~~~~~~

All functions are instances of the class ``<function>``. Dylan has two
built-in instantiable subclasses of ``<function>``: ``<generic-function>``
and ``<method>``. Both methods and generic functions can be called in the
same way. As we have seen, a generic function can contain zero or more
methods. If a generic function is called, it must have at least one
applicable method or Dylan signals an error.

Creation of generic functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create a generic function in the following ways:

- You can create one explicitly by ``define generic``.
- You can create one explicitly by calling ``make`` on the
  ``<generic-function>`` class. You rarely need to create a generic
  function this way.
- You can create one implicitly by ``define method``. If the generic
  function named by this definition does not yet exist, Dylan creates
  it.
- You can create one implicitly by defining a slot in ``define class``.
  If a getter generic function for the slot does not yet exist, Dylan
  creates it.
- You can create one implicitly by defining a slot (other than a
  constant slot) in ``define class``. If a setter generic function for
  the slot does not yet exist, Dylan creates it.

Each of these procedures, except a call to ``make``, defines a module
constant whose value is the generic function created.

When Dylan creates a generic function implicitly, it creates a parameter
list and a value declaration for the generic function that are designed
to restrict the addition of subsequent methods to the generic function
as little as possible. All required arguments to the generic function
have type specializers of ``<object>``, and the generic function can
return any number of values of any type. The generic function’s
parameter list is congruent with that of the method being defined. If
the generic function accepts keyword arguments, the parameter list ends
with ``#key``.

Creation of methods
~~~~~~~~~~~~~~~~~~~

You can create a method in the following ways:

- You can create one explicitly by ``define method``. This definition
  also adds the method to a generic function, creating the generic
  function if the latter does not already exist.
- You can create one explicitly by a ``method`` statement. This statement
  does not add the method to a generic function.
- You can create one explicitly by a ``local method`` declaration. This
  declaration creates one or more methods, and assigns each to a local
  variable such that the binding is visible to all other methods
  defined in the same ``local`` declaration. This declaration does not
  add the method to a generic function.
- You can create one implicitly by defining a slot (other than a
  virtual slot) in ``define class``. Dylan defines a getter method for
  the slot, and adds it to a generic function, creating the generic
  function if that function does not already exist.
- You can create one implicitly by defining a slot (other than a
  virtual or a constant slot) in ``define class``. Dylan defines a
  setter method for the slot, and adds it to a generic function,
  creating the generic function if that function does not already
  exist.

Creating a method by using ``method`` is useful when the method does not
need to be part of a generic function. For instance, various Dylan
functions take as arguments other functions that act as predicates, or
test functions. One of these is ``choose``, which selects members of a
sequence that satisfy a test function, and returns those members as a
new sequence. We might pick all the strings out of a mixed sequence as
follows:

.. code-block:: dylan

    define method choose-strings
        (sequence :: <sequence>) => (new-seq :: <sequence>)
      // choose takes two arguments: a function and a sequence
      choose(method (object) instance?(object, <string>) end method,
             sequence);
    end method choose-strings;

Creating a method by using ``local method`` is useful for a method that
does not need to be part of a generic function, but does need to be
given a name so that it can call itself recursively, or so that other
code in the enclosing body can refer to it. For an example, see
:ref:`collect-recursive-list-copier`.

Application of a function to arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Dylan function ``apply`` takes as arguments a function and one or more
additional arguments, the final one of which must be a sequence. The
``apply`` function calls its first argument — the function — and passes
that function the remaining arguments to ``apply``. But instead of
passing its final argument as a sequence, it passes each element of the
sequence as an individual argument.

The ``apply`` function is perhaps most useful in the body of a function
that receives a variable number of arguments and must pass those
arguments to another function that also takes a variable number of
arguments. For example, we can use ``apply`` to write a recursive version
of the ``sum`` function that we defined iteratively in `Parameter lists`_:

.. code-block:: dylan

      // Sum one or more values
      define method sum (value, #rest more-values)
        // If only one value, that is the answer
        if (empty?(more-values))
          value;
          // Otherwise, add the first value to the sum of the rest
        else
          value + apply(sum, more-values);
        end if;
      end method sum;

Operations on functions
~~~~~~~~~~~~~~~~~~~~~~~

Dylan has several functions that take functions as arguments, and return
new functions that are transformations of those arguments. These
operations permit many kinds of composition of functions and other
objects to generate new functions.

Three of these functions take predicates as arguments, and return the
complement, disjunction, or conjunction of the predicates. For example,
``complement`` takes a predicate and returns the latter’s complement — a
function that returns ``#t`` when the original predicate would have
returned ``#f``, and otherwise returns ``#f``.

.. index:: curry

The ``curry`` function takes a function and any number of additional
arguments. It returns a new function that applies the original function,
first to the additional arguments to ``curry``, then to the arguments to
the new function. In :ref:`collect-using-map-curry`, we call ``curry`` with
``*`` and a number to return a function that multiplies that function’s
argument by the given number. We then map this new function over the
elements of a vector to perform a scalar multiplication of the vector.

In fact, Dylan has a set of functions that map other functions over the
elements of collections in different ways. We used one of these,
``choose``, in `Creation of methods`_. Some of these functions return
new collections; others return single values. For more examples, see
:ref:`collect-iteration-over-sequence`.

.. _func-closures:

Closures
~~~~~~~~

This section describes closures — an advanced concept. If you do not
understand or wish to study this section, you can safely skip it.

Consider the following example:

.. code-block:: dylan

    define method call-and-show (function :: <function>, #rest arguments)
      format-out("The result is %=.\n", apply(function, arguments));
    end method call-and-show;

    define method show-next (x :: <integer>)
      call-and-show(method () x + 1 end method);
    end method show-next;

When we execute this code, we get the expected result:

.. code-block:: dylan-console

    ? show-next(41);
    => The result is 42.

But why did we get that result? We created an anonymous method in
``show-next``, and passed that anonymous method into a completely
separate method (``call-and-show``), where ``x`` is not bound to anything.
And yet, when the ``call-and-show`` method executed the anonymous method
that we made, somehow the anonymous method could still access the ``x``
binding. We got this reasonable result because the ``method`` statement
can create a special kind of method called a closure.

Recall that Dylan has two kinds of variable: module variables and local
variables. A local variable is defined explicitly by a ``let`` or ``local``
declaration, and implicitly by a function call, when a method’s
parameters are initialized to that method’s arguments. Local variables
are defined within a limited *lexical scope* — that is, they *bind* a
name to a value only within a particular textual portion of the program.
This portion of the program is that part of the innermost body that
follows the definition of the local variable.

A ``method`` statement or a ``local`` declaration can define a method in a
portion of a program where local variables are in effect. In the
preceding example, we use a ``method`` statement to define a method inside
the body of the ``show-next`` method, where the local variable ``x`` (the
parameter for the ``show-next`` method) is bound to the argument to
``show-next``. The method that we define inside ``show-next`` refers to
that local variable ``x``.

In general, when a program exits a body, the local variables defined
inside that body cease to be defined, and it is an error for the program
to refer to those variables. But there is an exception. If we use
``method`` or ``local`` to define a method, and if we then execute that
method outside the body in which we define it, the method can still
refer to the local variables that were in effect when the method was
defined. Such a method is called a closure.

A *closure* is a method that *closes over* or captures local variables
that are in effect when the method is defined and that are referred to
in the body of the method. The closure created by the ``method`` statement
in our example captures the local variable ``x``. So, even though the
local variable ``x`` is not defined in the lexical scope of the
``call-and-show`` method, the closure called by ``call-and-show`` can access
the captured binding of ``x``.

For examples of closures as iteration or mapping functions for
collections, see :ref:`collect-mapping-functions`, and :ref:`collect-using-map-curry`.

Summary
-------

In this chapter, we covered the following:

- We described the syntax of Dylan function calls, including syntactic
  structures that are abbreviations for function calls. These syntactic
  structures include slot references, element references, and most
  operator calls.
- We described how a function and its caller interact. In particular,
  we discussed the relations among arguments, parameters, value
  declarations, and returned values.
- We discussed the kinds of parameters that a function can have
  (required, rest, and keyword). We then outlined the rules for
  congruent parameter lists and value declarations of a generic
  function and its methods.
- We discussed ways of creating generic functions and methods, and of
  applying a function to arguments.
- We outlined Dylan’s operations on functions.
- We introduced the concept of closures.

