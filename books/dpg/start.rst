Quick Start
===========

We start by jumping right into Dylan. We show how to interact with a
development environment, to use basic arithmetic functions, to define
variables and constants, and to create a simple but complete Dylan
program.

The Dylan language does not specify a development environment, but many
Dylan implementations provide one. A *development environment* can
contain many tools, such as an editor custom-tailored for Dylan code, a
browser that helps you to examine objects, a debugger, and a *listener*
that enables you to type in expressions and to see their return values
and output. You can use a listener to test pieces of your program
without compiling the whole program. When you start using Dylan, a good
way to learn and explore is to use a listener. We use a hypothetical
listener in this chapter to show the results of evaluating Dylan
expressions. Of course, Dylan also supports the traditional approach of
editing source files, compiling the program, and running the program.

Dialog with a Dylan listener
----------------------------

Here is a sample dialog between a user and a listener. The *bold
typewriter font* shows what the user types. The *bold-oblique typewriter
font* shows what the listener displays.

.. code-block:: dylan-console

    ? 7 + 12;
    => 19

In our hypothetical listener, the Dylan prompt is the question mark, ``?``.
The user types in ``7 + 12;`` and presses Enter. The listener executes
the expression and displays the value returned by that expression, which
is ``19``. The listener displays any return values and output produced by
the expression.

.. topic:: Environment note:

   Our hypothetical development environment does not represent any
   particular Dylan development environment. The Dylan language does
   not require a development environment, so any given implementation
   may not provide one.

Simple arithmetic operations
----------------------------

We can do other simple arithmetic:

.. code-block:: dylan-console

    ? 7 * 52;
    => 364

    ? 7 - 12;
    => -5

.. topic:: Caution: Spaces are needed!

   In Dylan, it is legal to use characters such as ``+``, ``-``, ``*``,
   ``<``, ``>``, and ``/`` in names of variables.  Therefore, in most
   cases, you must leave spaces around those characters in code, to
   make it clear that you are using them as functions, and that they
   are not part of the name of a variable. For example:

   * ``a + b`` means add ``a`` and ``b``.
   * ``a+b`` means the name ``a+b``.

We can multiply several numbers together:

.. code-block:: dylan-console

    ? 24 * 7 * 52;
    => 8736

True and false
~~~~~~~~~~~~~~

We can compare the magnitude of two numbers:

.. code-block:: dylan-console

    ? 1 = 1;
    => #t

    ? 3 < 30;
    => #t

    ? 15 > 16;
    => #f

The functions ``=``, ``<``, and ``>`` are *predicates*. A predicate returns
true if the condition it is testing is true; otherwise, it returns
false. As you might guess, ``#t`` means true and ``#f`` means false. False
is represented by the unique value ``#f`` only, but any object that is not
``#f`` is true (thus, ``0`` is a true value).

.. topic:: Comparison with C and C++: Caution!

   C and C++ use integers to represent Boolean values — ``0`` represents
   false, and any nonzero value is considered true. Dylan has an explicit
   ``<boolean>`` type with two instances: ``#f`` represents false, and
   ``#t`` represents the *canonical* true value. However, any value
   other than ``#f`` is also considered true in a Boolean test. Thus,
   in Dylan, 0 is considered true.

.. topic:: Comparison with Java:

   Java has a separate type for Boolean values.  Unlike Dylan, C, or C++,
   the Java ``Boolean`` class has only two values, ``true`` and ``false``.
   This design allows the compiler to issue warnings for the common C error
   ``if (a=b) ...``, because an assignment does not typically yield a
   Boolean result. An explicit conversion is required to test nonzero
   in Java: ``if (a!=0) ...``.

Infix syntax and function-call syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The functions ``+``, ``-``, ``*``, ``<``, ``>``, and ``=`` use *infix syntax*;
that is, the function name appears between the arguments to the
function. Most other Dylan functions use the function-call syntax shown
in the following call to the ``min`` function, which returns the smallest
of its arguments:

.. code-block:: dylan-console

    ? min(2, 4, 6);
    => 2

The function name appears first, followed by its arguments, which are
surrounded by parentheses and separated by commas. Other examples of the
function-call syntax follow:

.. index::
   single: even?
   single: zero?

.. code-block:: dylan-console

    ? even?(3);
    => #f

    ? zero?(0);
    => #t

.. index::
   pair: naming conventions; predicate

.. topic:: Convention:

   The names of most predicates end with a question mark — for example,
   ``even?``, ``odd?``, ``zero?``, ``positive?`` and ``negative?``.
   The question mark is part of the name, and does not have any
   special behavior. There are exceptions to this convention, such as
   the predicates named ``=``, ``<``, and ``>``.

Case insensitivity
~~~~~~~~~~~~~~~~~~

Dylan is case insensitive. Therefore, we can call the ``max`` function as
follows:

.. code-block:: dylan-console

    ? MAX(-1, 1);
    => 1

    ? mAx(0, 55.3, 92);
    => 92

.. _start-variables-constants:

Variables and constants
-----------------------

We can define variables for storing values:

.. code-block:: dylan-console

    ? define variable *my-number* = 7;

    ? define variable *your-number* = 12;

.. index::
   single: module variable; introduction

In Dylan, these variables are called *module variables*. A module
variable has a name and a value. For now, you can consider module
variables to be like global variables in other languages. (See
:ref:`libraries-modules`, for information about modules.) Module variables
can have different values assigned to them during the execution of a
program. When you define a module variable, you must *initialize* it;
that is, you must provide an initial value for it. For example, the
initial value of ``*my-number*`` is ``7``.

.. index::
   pair: variable; naming conventions

.. topic:: Convention:

   Module variables have names that start and end with an asterisk — for
   example, ``*my-number*``. The asterisks are part of the name, and do
   not have any special behavior.

We can ask the listener for the values of module variables:

.. code-block:: dylan-console

    ? *my-number*;
    => 7

    ? *your-number*;
    => 12

We can add the values stored in these variables:

.. code-block:: dylan-console

    ? *my-number* + *your-number*;
    => 19

We can multiply the values stored in these variables:

.. code-block:: dylan-console

    ? *my-number* * *your-number*;
    => 84

We can use the *assignment operator*, ``:=``, to change the values
stored in a variable:

.. code-block:: dylan-console

    ? *my-number* := 100;
    => 100

Assignment, initialization, and equality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

People new to Dylan may find ``=`` and ``:=`` confusing, because the names
are similar, and the meanings are related but distinct.

The meaning of ``=`` depends on whether it appears an expression, or in a
definition of a variable or constant. In an expression, ``=`` is a
function that tests for equality; for example,

.. code-block:: dylan-console

    ? 3 = 3;
    => #t

In a definition of a variable or constant, ``=`` precedes the initial
value of the variable or constant; for example,

.. code-block:: dylan-console

    ? define variable *her-number* = 3;

After you initialize a variable with ``=``, the ``=`` function returns
true:

.. code-block:: dylan-console

    ? *her-number* = 3;
    => #t

The assignment operator, ``:=``, performs assignment, which is setting
the value of an existing variable; for example,

.. code-block:: dylan-console

    ? *her-number* := 4;
    => 4

After you have assigned a value to a variable, the ``=`` function returns
true:

.. code-block:: dylan-console

    ? *her-number* = 4;
    => #t

Dylan offers an identity predicate, which we discuss in
:ref:`oo-1-equality-predicates`.

.. index::
   single: variable; type constraints
   single: type constraint

Variables that have type constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We defined the variables ``*my-number*`` and ``*your-number*`` without
giving a *type constraint* on the variables. Thus, we can store any type
of value in these variables. For example, here we use the assignment
operator, ``:=``, to store strings in these variables:

.. code-block:: dylan-console

    ? *my-number* := "seven";
    => "seven"

    ? *your-number* := "twelve";
    => "twelve"

What happens if we try to add the string values stored in these
variables?

.. code-block:: dylan-console

    ? *my-number* + *your-number*;
    => ERROR: No applicable method for + with arguments ("seven", "twelve")

Dylan signals an error because the ``+`` function does not know how to
operate on string arguments.

.. topic:: Environment note:

   The Dylan implementation defines the exact wording of error messages,
   and what happens when an error is signaled. If your implementation
   opens a Dylan debugger when an error is signaled, you now have an
   opportunity to experiment with the debugger!

We can redefine the variables to include a type constraint, which
ensures that the variables can hold only numbers. We specify that
``*my-number*`` can hold any integer, and that ``*your-number*`` can
hold a single-precision floating-point number:

    ? define variable *my-number* :: <integer> = 7;

    ? define variable *your-number* :: <single-float> = 12.01;

What happens if we try to store a string in one of the variables?

.. code-block:: dylan-console

    ? *my-number* := "seven";
    => ERROR: The value assigned to *my-number* must be of type <integer>

Both ``<integer>`` and ``<single-float>`` are *classes*. For now, you can
think of a class as being like a datatype in another language. Dylan
provides a set of built-in classes, and you can also define new classes.

.. index::
   pair: class; naming conventions

.. topic:: Convention:

   Class names start with an open angle bracket and end with a close
   angle bracket — for example, ``<integer>``. The angle brackets are
   part of the name, and do not have any special behavior.

The ``+`` function can operate on numbers of different types:

.. code-block:: dylan-console

    ? *my-number* + *your-number*;
    => 19.01

.. index::
   single: constant; module constant
   single: module constant

Module constants
~~~~~~~~~~~~~~~~

A *module constant* is much like a module variable, except that it is an
error to assign a different value to a constant. Although you cannot
assign a different value to a constant, you may be able to change the
elements of the value, such as assigning a different value to an element
of an array.

You use ``define constant`` to define a module constant, in the same way
that you use ``define variable`` to define a variable. You must initialize
the value of the constant, and you cannot change that value throughout
the execution of a Dylan program. Here is an example:

.. code-block:: dylan-console

    ? define constant $pi = 3.14159;

.. index::
   pair: constant; naming conventions

.. topic:: Convention:

   Module constant names start with the dollar sign, ``$`` — for example,
   ``$pi``. The dollar sign is part of the name, and does not have any
   special behavior.

Both module variables and module constants are accessible within a
*module*.

(See :ref:`libraries-modules`, for information about modules.) Dylan also
offers variables that are accessible within a smaller area, called
*local variables*. There is no concept of a local constant; all
constants are module constants. Therefore, throughout the rest of this
book, we use the word *constant* as shorthand for module constant.

Local variables
~~~~~~~~~~~~~~~

You can define a local variable by using a ``let`` declaration. Unlike
module variables, local variables are established dynamically, and they
have *lexical scope*. During its lifetime, a local variable shadows any
module variable, module constant, or existing local variable with the
same name.

Local variables are scoped within the smallest body that surrounds them.
You can use ``let`` anywhere within a body, rather than just at the
beginning; the local variable is declared starting at its definition,
and continuing to the end of the smallest body that surrounds the
definition.

A *body* is a region of program code that delimits the scope of all
local variables declared inside the body. When you are defining
functions, usually there is an implicit body available. For example,
``define method`` creates an implicit body. (For information about method
definitions, see :ref:`oo-1-method-definitions`.) Other control structures, such
as ``if``, create implicit bodies. Bodies can be nested. If there is no
body handy, or if you want to create a body smaller than the implicit
one, you can create a body by using ``begin`` to start it and ``end``
to finish it:

.. code-block:: dylan-console

    ? begin
       let radius = 5;
       let circumference = 2 \* $pi \* radius;
       circumference;
     end;
    => 31.4159

The local variables ``radius`` and ``circumference`` are declared,
initialized, and used within the body. The value returned by the body is
the value of the expression executed last in the body, which is
``circumference``. Outside the lexical scope of the body, the local
variables are no longer declared, and trying to access them is an error:

.. code-block:: dylan-console

    ? radius
    => ERROR: The variable radius is undefined.

Formatted output
----------------

Throughout this book, we use the ``format-out`` function to print output.
The syntax of ``format-out`` is

.. code-block:: dylan

    format-out(string, arg1, ... argn)

The ``format-out`` function sends output to the standard output
destination, which could be the window where the program was invoked, or
a new window associated with the program. The standard output
destination depends on the platform.

The *string* argument can contain ordinary text, formatting instructions
beginning with ``%``, and characters beginning with a backslash, ``\``.
Ordinary text in the format string is sent to the destination verbatim.
You can use the backslash character in the *string* argument to insert
unusual characters, such as ``\n``, which prints the newline character.

.. code-block:: dylan-console

    ? format-out("Your future is filled with wondrous surprises.\n")
    => Your future is filled with wondrous surprises.

Formatting instructions begin with a percent sign, ``%``. For each ``%``,
there is normally a corresponding argument giving an object to output.
The character after the ``%`` controls how the object is formatted. A wide
range of formatting characters is available, but we use only the
following formatting characters in this book:


- ``%d`` Prints an integer represented as a decimal number
- ``%s`` Prints the contents of its string argument unquoted
- ``%=`` Prints an implementation-specific representation of the object;
  you can use ``%=`` for any class of object

Here are examples:

.. code-block:: dylan-console

    ? format-out
        ("Your number is %= and mine is %d\n", *your-number*,
         *my-number*);
    => Your number is 12.01 and mine is 7.

    ? format-out("The %s meeting will be held at %d:%d%d.\n", "Staff", 2,
                 3, 0);
    => The Staff meeting will be held at 2:30.

In Dylan, functions do not need to return any values. The ``format-out``
function returns no values. Thus, it is called only for its side effect
(printing output).

.. topic:: Comparison with C:

   ``format-out`` is similar to ``printf``.

The ``format-out`` function is available from the ``format-out`` library,
and is not part of the core Dylan language. We now describe how to make
the ``format-out`` function accessible to our program, and how to set up
the files that constitute the program. Many of the details depend on the
implementation of Dylan, so you will need to consult the documentation
of your Dylan implementation.

.. topic:: Usage note:

   The Apple Technology Release does not currently provide the
   ``format-out`` function. For information about how to run these
   examples in the Apple Technology Release, see Harlequin’s or
   Addison-Wesley’s Web page for our book. See :doc:`environ`.

.. _start-complete-program:

A complete Dylan program
------------------------

In this section, we show how to create a complete Dylan program. The
Dylan program will print the following::

    Hello, world

The Dylan expression that prints that output is

.. code-block:: dylan

    format-out("Hello, world\n");

A Dylan *library* defines a software component — a separately compilable
unit that can be either a stand-alone program or a component of a larger
program. Thus, when we talk about creating a Dylan program, we are
really talking about creating a library.

A library contains *modules*. Each module contains definitions and
expressions. The module is a *namespace* for the definitions and
expressions. For example, if you define a module variable in one
particular module, it is available to all the code in that module. If
you choose to export that module variable, you can make it accessible to
other modules that import it. In this chapter, we give the bare minimum
of information about libraries and modules — just enough for you to get
started quickly. For a complete description of libraries and modules,
see :doc:`libraries`.

To create a complete Dylan program, we need

-  To define the library that is our program; we shall create a library
   named ``hello``
-  To define a module (or more than one) in the library, to hold the
   definitions and expressions in our program; we shall create a module
   named ``hello`` in the ``hello`` library
-  To write the program code, in the module; we shall put the
   ``format-out`` expression in the ``hello`` module of the
   ``hello`` library

.. _start-files-of-dylan-program:

Files of a Dylan program
~~~~~~~~~~~~~~~~~~~~~~~~

Different Dylan environments store programs in different ways, but there
is a file-based *interchange format* that all Dylan environments accept.
In this interchange format, any program consists of a minimum of two
files: a file containing the program itself, and a file describing the
libraries and modules. The most trivial program consists of a single
module in a single library, but it is still expressed in two files. Most
Dylan implementations also accept a third file, which enumerates all the
files that make up a program; this file is called a *library-interchange
definition (LID)* file.

The details of how the files are named and stored depends on your Dylan
implementation. Typically, however, you have a directory containing all
the files of the program. As shown below, we name our program directory
``hello``, and name the files ``hello.lid``, ``library.dylan``, and
``hello.dylan`` (the latter is the program file).

    hello

    - hello.lid
    - library.dylan
    - hello.dylan

.. topic:: Comparison with C:

   The following analogies may help you to understand how the elements of
   Dylan programs correspond to elements of C programs:

   - The *program files* are similar to *.c* files in C.
   - The *library file* is similar to a C header file.
   - The *LID file* is similar to a *makefile*, which is used in certain
     C development environments.

Components of a Dylan program
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We start with this simple Dylan expression:

.. code-block:: dylan

    format-out("Hello, world\n");

All Dylan expressions must be in a module. Therefore, we use a text
editor to create a file that contains the expression within a module:

The program file: ``hello.dylan``.

.. code-block:: dylan

    module: hello

    format-out("Hello, world\n");

The ``hello.dylan`` file is the top-level file; you can think of it as the
program itself. When you run this program, Dylan executes all the
expressions in the file in the order that they appear in the file. There
is only one expression in this program — the call to ``format-out``.

The first line of this file declares that the expressions and
definitions in this file are in the ``hello`` module. Before we can run
(or even compile) this program, we need to define the ``hello`` module.
All modules must be in a library, so we must also define a library for
our ``hello`` module. We create a second file, called the library file,
and define the ``hello`` module and ``hello`` library in the library file:

The library file: ``library.dylan``.

.. code-block:: dylan

    module: dylan-user

    define library hello
      use dylan;
      use format-out;
    end library hello;

    define module hello
      use dylan;
      use format-out;
    end module hello;

The first line of ``library.dylan`` states that the expressions in this
file are in the ``dylan-user`` module. Every Dylan expression and
definition must be in a module, including the definitions of libraries
and modules. The ``dylan-user`` module is the starting point — the
predefined module that enables you to define the libraries and modules
that your program uses.

In the file ``library.dylan``, we define a library named ``hello``, and
a module named ``hello``. We define the ``hello`` library to use the
``dylan`` library and the ``format-out`` library, and we define the
``hello`` module to use the ``dylan`` module and the ``format-out`` module.

One library *uses* another library to allow its modules to use the other
library’s exported modules. Most libraries need to use the ``dylan``
library, because it contains the ``dylan`` module. One module *uses*
another module to allow its definitions to use the other module’s
exported definitions. Most modules need to use the ``dylan`` module in the
``dylan`` library, because that module contains the definitions of the
core Dylan language. We also need to use the ``format-out`` module in the
``format-out`` library, because that module defines the ``format-out``
function, which we use in our program.

Finally, we create a LID file that enumerates the files that make up the
library. This file does not contain Dylan expressions, but rather is
simply a textual description of the library’s files:

The LID file: ``hello.lid``.

.. code-block:: dylan

    library: hello
    files: library
           hello

The LID file simply states that the library ``hello`` comprises two files,
named ``library`` and ``hello``. In other words, to build the ``hello``
library, the compiler must process the two files listed, in the order
that they appear in the file. The order is significant, because a module
must be defined before the code that is in the module can be analyzed
and compiled.

You can consult the documentation of your Dylan implementation to find
out how to build an executable program from these files, and how to run
that program once it is built. Most Dylan environments produce
executable programs that can be invoked in the same manner as any other
program on the particular platform that you are using.

We incur a fair amount of overhead in setting up the files that make up
a simple program. Most environments automate this process — some of the
complexity shown here occurs because we are working with the lowest
common denominator: interchange files. The advantages of libraries and
modules are significant for larger programs. See :doc:`libraries`.

Summary
-------

In this chapter, we covered the following:

-  We entered Dylan expressions to a listener and saw their values or
   output.
-  We used simple arithmetic functions: ``+``, ``*``, ``-``. We used
   predicates: ``=``, ``<``, ``>``, ``even?``, and ``zero?``.
-  We described certain naming conventions in Dylan; see
   :ref:`dylan-naming-conventions-start`.
-  We described the syntax of some commonly used elements of Dylan; see
   :ref:`syntax-of-dylan-elements-start`.
-  We defined module variables (with ``define variable``), constants
   (with ``define constant``), and local variables (with ``let``).
-  We set the value of variables by using ``:=``, the assignment
   operator.
-  We defined a simple but complete Dylan program, consisting of a LID
   file, a library file, and a program file.

Here, we summarize the most basic information about libraries and
modules:

-  A Dylan library defines a software component — a separately
   compilable unit that can be either a stand-alone program or a
   component of a larger program. Thus, when we talk about creating a
   Dylan program, we are really talking about creating a library.
-  Each Dylan expression and definition must be in a module. Each module
   is in a library.
-  One module uses another module to allow its definitions to use the
   other module’s exported definitions. Most modules need to use the
   ``dylan`` module in the ``dylan`` library, because it contains the
   definitions of the core Dylan language.
-  One library uses another library to allow its modules to use the
   other library’s exported modules. Most libraries need to use the
   ``dylan`` library, because it contains the ``dylan`` module.

.. _dylan-naming-conventions-start:

.. table:: Dylan naming conventions shown in this chapter.

   +-----------------+-----------------+
   | Dylan element   | Example of name |
   +=================+=================+
   | module variable | ``*my-number*`` |
   +-----------------+-----------------+
   | constant        | ``$pi``         |
   +-----------------+-----------------+
   | class           | ``<integer>``   |
   +-----------------+-----------------+
   | predicate       | ``positive?``   |
   +-----------------+-----------------+

.. _syntax-of-dylan-elements-start:

.. table:: Syntax of Dylan elements.

   +----------------------------+------------------------------+
   | Dylan element              | Syntax example               |
   +============================+==============================+
   | string                     | ``"Runway"``                 |
   +----------------------------+------------------------------+
   | true                       | any value that is not ``#f`` |
   +----------------------------+------------------------------+
   | canonical true value       | ``#t``                       |
   +----------------------------+------------------------------+
   | false                      | ``#f``                       |
   +----------------------------+------------------------------+
   | infix syntax function call | ``2 + 3;``                   |
   +----------------------------+------------------------------+
   | function call              | ``max(2, 3);``               |
   +----------------------------+------------------------------+

