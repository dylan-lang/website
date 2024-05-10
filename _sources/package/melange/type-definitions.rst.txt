Type Definitions
================

When Melange encounters a "type definition" [The
definition may be implicit, as in ``char ** int`` or ``struct foo
*bar``. Simply by being present these code fragments supply
implicit definitions for ``char *``, ``char **`` and ``struct foo``.]
within a header file, it will typically create a new Dylan class
which corresponds to that C type. Usually, this will be a
subclass of <statically-typed-pointer>, which encapsulates
the raw C pointer value (i.e an object address).  Each
statically typed pointer class will have exactly the same
structure (i.e. a single address), but the class itself can be
used to determine what operations are supported on the
data. This could include slot accessors for "struct"s and
"union"s, dereference operations for "pointer" types, or general
information about the objectUs size, etc.

There are times when you will find that some of the types
defined in a header file are not really "new". It might be that
they are completely identical to some type defined in another
interface definition, or they might be "isomorphic" to some
existing type which has more complete support. Melange provides
support for both of these cases. The first case is handled by
"equating" the two types, while the second is handled by
"mapping" (i.e.  transforming) one type into the other.

For example, many header files contain definitions use the
types ``char *`` and ``boolean``.  The declarations of these types
don't provide any semantic interpretations -- ``char *`` is simply
the address of a character, and boolean is nothing but a
one-byte integer. However, by equating ``char *`` to the
predefined ``<c-string>`` type, we can tell Melange that it is
actually a ``<string>`` and should inherit all of the
operations defined upon ``<string>``. Likewise, we can map
the integral ``boolean`` values into ``#t`` and ``#f`` to get a
``<boolean>``. These integral values will be automatically
translated into ``<boolean>`` when they are returned by a C
function, and ``<boolean>`` will be translated back into
integers when passed as arguments to C functions.

.. _melange-implicit-class-definitions:

Implicit class definitions
--------------------------

Unless otherwise specified, new classes will be created
for each type defined in a C header file. When the header file
provides meaningful names for these types, then Melange will
pass those names to the mapping functions to generate names
for the Dylan classes. Otherwise, an anonymous name will be
generated, limiting your ability to refer to the new type. For
example, ``struct foo`` would typically generate the class
``<foo>``, while ``struct foo ***`` might generate the class
``<anonymous-107>``. In either case, you can explicitly
specify the name for the new class by using the ``rename:``
option described above.

Different sorts of C declarations will yield different
sorts of Dylan classes as well as different sets of operations
defined upon them. Therefore, we will consider each variety
separately:

Primitive types
   The types ``int``, ``char``, ``long``, ``short`` and their
   unsigned counterparts are simply translated into
   ``<integer>``, while ``float`` and ``double`` are
   translated into ``<float>``.  However, Melange knows
   the sizes of each of these types so that pointers and
   native C "vectors" of them (described below) will work
   properly. No new types are created for these types.

Pointer types
   Declarations like ``int *`` or ``struct foo ***``
   generate new subclasses of
   ``<statically-typed-pointer>``. Note that ``struct
   foo *`` is actually treated as a synonym for ``struct
   foo``, and does not get a distinct class, although any
   extra levels of indirection (i.e.  ``struct foo **``) will
   generate new pointer classes. Three operations are
   supported upon pointer classes:

   .. code-block:: dylan

       pointer-value (pointer, #key index) => (value)

   This function "dereferences" the pointer and
   returns the value. If index is supplied, then "pointer"
   is treated as a vector of values and the appropriate
   element is returned.

   .. code-block:: dylan

       content-size (cls) => integer

   Returns the size of the value referenced by instances of "cls".
   If the size is not known, this is 0.

   Note that these types are not automatically treated as vectors.
   You may, however, make them so by using a ``superclasses:``
   option to make them <c-vector>s.

Vector types
   Declarations like ``char [256]`` are treated almost
   identically to pointer types, but they are automatically
   defined as subclasses of <c-vector>, so that all
   vector operations will be defined on them. However,
   because many systems depend upon the lack of bounds
   checking in C, vector types have a default size of
   ``#f``. You may explicitly define ``size`` functions to
   provide a more accurate size.

Structure types
   Declarations like ``struct bar {int a; char *b;}``
   also generate new subclasses of
   ``<statically-typed-pointer>``. Melange will define
   all of the operations defined for pointer values
   (described above), as well as accessors for each of the
   structure slots.  Structure objects are always accessed
   through "pointers" to them. Therefore, unless a non-zero
   index is specified, ``pointer-value`` will simply return
   the object passed to it. (The operation is still defined
   because non-zero indices can be used for vector access.)

Union types
   Declarations like ``union bar {int a, char *b;}``
   are treated the same as struct declarations, except that
   the slot accessors all refer to the same areas in
   memory.  Enumeration types -- Declarations like ``enum
   foo {one, two, three};`` are simply aliased to
   <integer>. However, constants are defined for each
   of the enumeration literals.

Typedefs
   Declarations like ``typedef struct foo bar`` simply define
   new names for existing types.

.. _melange-class-inheritance:

Specifying class inheritance
----------------------------

When Melange creates new ``<statically-typed-pointer>``
classes, it typically creates them as simple subclasses of
``<statically-typed-pointer>``, with no other
superclasses. However, you might sometimes need more control
over the class hierarchy. For example, you might wish to
specify that a C type should be considered a subtype of the
abstract class ``<sequence>``. You could accomplish this
via the following declarations:

.. code-block:: dylan

    define interface
       #include "sequence.h";
       struct "struct cons_cell" => <c-list>,
          superclasses: {<sequence>};
       function "c_list_size" => size;
    end interface;

    define method forward-iteration-protocol (seq :: <c-list>)
      ...

Note that the type ``<c-list>`` will still be a
subclass of ``<statically-typed-pointer>`` -- we have
simply added ``<sequence>`` to the list of
superclasses. If ``<statically-typed-pointer>`` is not
explicitly included in the ``superclasses:`` option, then it
will be added at the end of the superclass list.

As demonstrated in the above example, you are still
responsible for specifying whatever functions are required to
satisfy the contract for the declared
superclasses. ``<C-list>`` will be declared as a sequence,
but you must specify a forward iteration protocol before any
of the standard sequence operations will work properly.

The ``superclasses:`` option may currently be used within
``struct``, ``union``, and ``pointer`` clauses.
