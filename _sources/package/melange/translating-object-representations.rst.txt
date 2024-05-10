Translating Object Representations
==================================

Whenever a native C object is returned from a function or
a Dylan object is passed into a C function, it is necessary to
translate between the object representations used by the two
languages. From MelangeUs standpoint, native C objects consist
of an arbitrary bit pattern which can be translated to or from a
small number of "low level" Dylan types -- namely
``<integer>``, ``<float>``, or any subclass of
``<statically-typed-pointer>``. This translation is handled
automatically, although the user may explicitly specify which of
the possible Dylan types should be chosen for any given C object
type. In some cases, a further translation may take place,
converting the "low level" Dylan value to or from some arbitrary
"high level" Dylan type. (For example, an ``<integer>`` might
be translated into a ``<boolean>`` or a ``<character>``, and
a ``<c-string>`` might be translated into a
``<byte-string>``.) These "high level" translations are
automatically invoked at the appropriate times, but both the
"target" types and the methods for performing the translation
must be specified by the user.

.. _melange-low-level-transformations:

Specifying low level transformations
------------------------------------

The target Dylan type for "low level" translations is
typically chosen automatically by Melange. Integer and
enumeration types are translated into ``<integer>``;
floating point types are translated to ``<float>``; and all
other types are translated into newly created subclasses of
``<statically-typed-pointer>``. However, you may explicitly
declare the target Dylan type for any C type by means of an
``equate:`` option:

.. code-block:: dylan

    define interface
       #include "gc.h",
          equate: {"char *" => <c-string>};
    end interface;

This declaration makes the very strong statement that
any values declared in C as ``char *`` are identical in form to
the predefined type ``<c-string>`` (which is described in
Appendix I). The system will therefore not define a distinct
type for ``char *`` and will ignore any structural information
provided in the header file. You might also use an ``equate:``
option to equate a type mentioned in one interface definition
with an identically named type which was defined in an earlier
interface definition.

You should use caution when equating two types. Since
Melange has no way of knowing when two types are equivalent,
it must trust your declarations. No type checking can or will
be done, so if you incorrectly equate two types, the results
will be unpredictable. In some cases, you may wish to go with
the less efficient but slightly safer technique of letting
Melange create a new type and then "mapping" that new type
into the desired type. (This is described in detail
below.)

Note also that two types with identical purposes will
not necessarily have identical representations. For example,
C's boolean types are simple integers and are not equivalent
to Dylan's ``<boolean>``. Again, explicit "mapping" may be
used to transform between these two representations.

In the current implementation, an ``equate:`` option only
applies within a single interface definition. Other interface
definitions will not automatically inherit the effects of the
declaration. In future versions, we may add the ability to
"use" other interface definitions (just as you would "use"
another module within a module definition) and thus pick up
the effects of the ``equate:`` (and ``map:``) options within those
interfaces.

.. _melange-high-level-transformations:

Specifying high level transformations
-------------------------------------

Sometimes you may wish to use instances of some C type
as if they were instances of some existing Dylan class, even
though they have different representations. In this case, you
can specify a secondary translation phase which
semi-automatically translates between a "low level" and a
"high level" Dylan representation. In order to do this, you
must provide a ``map:`` option:

.. code-block:: dylan

    define interface
       #include "gc.h",
          equate: {"char *" => <c-string>},
          map: {"bool" => <boolean>};
    end interface;

This clause will cause any functions defined within the
interface to call transformation functions wherever the
original C functions accept or return values of type
``bool``. Two different functions may be called:

.. code-block:: dylan

    import-value (high-level-class :: <class>, low-level-value :: <object>)

This function is called to transform result values
returned by C functions into a "high level" Dylan class. It
should always return an instance of "high-level-class".

.. code-block:: dylan

    export-value (lowlevel-class :: <class>, high-level-value :: <object>)

This function is called to transform "high level"
argument values passed to C functions into the "low level"
representations which will be meaningful to native C code. It
should always return an instance of "low-level-class".

Default methods, which simply call ``as``, are provided
for each of these functions.  This will be sufficient to
transform C's integral ``char`` into ``<character>``,
``<c-string>`` into other ``<string>``, or one "pointer"
type into another. There is also a predefined method which
will transform ``<integer>`` into
``<boolean>``. However, if you wish to perform arbitrary
transformations upon the values, you may need to define
additional methods for either or both of these functions. For
example, the default methods for transforming to and from
``<boolean>`` are:

.. code-block:: dylan

    define method export-value (cls == <integer>, value :: <boolean>)
     => (result :: <integer>);
       if (value) 1 else 0 end if;
    end method export-value;

    define method import-value (cls == <boolean>, value :: <integer>)
     => (result :: <boolean>);
       value ~= 0;
    end method import-value;

It is important to note that, unlike ``equate:`` options,
``map:`` options don't prevent Melange from creating new
types. You may, in fact, both equate and map the same
type. This will cause low level values to be created as
instances of the "equated" type and then transformed into
instances of the "target" type of the mapping. For example,
you might take advantage of the defined transformations
between string types by declaring:

.. code-block:: dylan

    define interface
       #include "/usr/include/sys/dirent.h",
          equate: {"char *" => <c-string>},
           map: {"char *" => <byte-string>};
    end interface;

This causes the system to automatically translate ``char *``
pointers into ``<c-string>`` (i.e. a particular variety
of statically typed pointer) and then to call ``import-value``
to translate the ``<c-string>`` into a
``<byte-string>``. If we did not provide the ``equate:``
option, then we would have to explicitly provide a function to
transform "pointers to characters" into
``<byte-string>``. The ``equate:`` option lets us take
advantage of all of the predefined functions for
``<string>``, which includes transformation into other
string types.
