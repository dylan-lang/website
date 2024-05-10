Specifying Object Names
=======================

Because naming conventions differ between C and Dylan,
Melange attempts to translate the names specified in C
declarations into a form more appropriate to Dylan. This
involves

- Adding angle brackets around type names.
- Adding dollar signs at the beginning of constant names.
- Translating (non-initial) underlines into hyphens.
- Adding ``struct-name$`` prefixes to slot accessors.

In many cases, this default behavior will be precisely
what you want. However, Melange provides mechanisms for
specifying different translations for some or all of the
declarations.

.. _melange-mapping-functions:

Mapping functions
-----------------

The translations described above are provided by calls
to a built-in "name mapping function" named
"minimal-name-mapping-with-structure-prefix". You may specify
other mapping functions via a ``name-mapper:`` option. Our
example interface might then look like this:

.. code-block:: dylan

    define interface
       #include "gc.h",
          name-mapper: c-to-dylan;
    end interface;


.. _melange-name-mapping:

.. figure:
   :caption: Standard Name Mapping Functions

+--------------------------------------------+-----------------------------------------------+
| Function                                   | Result                                        |
+============================================+===============================================+
| minimal-name-mapping-with-structure-prefix | Provides the translations described above.    |
+--------------------------------------------+-----------------------------------------------+
| minimal-name-mapping                       | Same as above, but excludes ``struct-name$``. |
+--------------------------------------------+-----------------------------------------------+
| c-to-dylan                                 | Like ``minimal-name-mapping``, but:           |
|                                            | * Adds hyphens to reinforce "CaseBased" word  |
|                                            | * Adds ``get-`` prefixes to slot accessors.   |
+--------------------------------------------+-----------------------------------------------+
| identity-name-mapping                      | Does no translation.                          |
+--------------------------------------------+-----------------------------------------------+


New name-mapping functions can be implemented within melange by
defining methods on the ``map-name`` generic function which
accepts the following parameters:

``mapper``
    a ``<symbol>`` which is typically specialized by a
    singleton to select a specific name mapper method.
``category``
    a ``<symbol>`` which will always be one of:
    ``#"type"``, ``#"constant"``, ``#"variable"``, or
    ``#"function"``.
``prefix``
    a ``<string>`` which is typically prepended to the
    result string.
``name``
    a ``<string>`` which supplies the original C
    name.
``sequence-of-classes``
    a sequence of simple names for the classes
    which logically "contain" the given object. For
    example, if we were processing the declaration
    ``struct str {int size; char *chars;}``, one of the
    calls to the mapping function would have with
    name bound to "size" and classes bound to
    ``#["str"]``.

It must return a ``<string>`` which will be used as the
Dylan name for the declaration.

Mapping functions may call ``hyphenate-case-breaks`` which
performs the same "CaseBased separation" as is done by
``c-to-dylan``. The trivial ``identity-name-mapping`` described
above might be implemented by:

.. code-block:: dylan

    define method map-name
       (mapper == #"identity-name-mapping", category, prefix, name, classes)
    => (result :: <string>)
       name;
    end method map-name;

You may specify different name mappers to be applied to
the slots of "container types". This capability is described
in a later section.

.. _melange-prefixes:

Prefixes
--------

As noted above, the name mapping function is passed a
``prefix`` argument. By default, it is an empty string, but
users may specify a different value by adding a ``prefix:``
option to the interface definition. For example, we might
expand the previous example to:

.. code-block:: dylan

    define interface
       #include "gc.h",
          name-mapper: c-to-dylan,
          prefix: "gc-";
    end interface;

This would cause Melange to tack ``gc-`` onto the
beginning of every translated symbol.  Because the system
knows about the "standard" Dylan naming conventions, it can do
this intelligently. You would, therefore, get names like
``<gc-bool>``, ``gc-time-to-gc``, and ``gc-scavenge``.

Note that the interpretation of the ``prefix`` is entirely
up to the name mapping routine.  Identity-name-mapping, for
example, completely ignores the prefix. All of the other
standard mapping functions prepend it to the name before
adding brackets or dollar signs, but after performing all
other transformations.

Facilities for adding "localized" prefixes to slot
accessors, enumeration literals, etc.  will be described in
later sections.

.. _melange-explicit-renaming:

Explicit Renaming
-----------------

Although the automatic name mapping described above is
sufficient for most objects named within a header file, there
are cases in which you might wish to explicitly control the
name of one or more specific objects. You can do this through
a ``rename:`` option. This options specifies a list of
translations between raw C names and Dylan identifiers. For
example, we might have:

.. code-block:: dylan

    define interface
       #include "gc.h",
          name-mapper: c-to-dylan,
          prefix: "gc-"
          rename: {"struct obj" => <C-Object>, "collect_garbage" => GC};
    end interface;

Note that the "target" of the renaming is an ordinary
Dylan variable and is therefore case-insensitive. However, the
source is an "alien name", which is (like all C code) case
sensitive. Alien names should refer to an object, function, or
type in exactly the same way you would refer to them in C. We
therefore say ``struct obj`` instead of simply ``obj``, and might
also say ``enum foo`` or ``union bar``. Alien names are actually
parsed according to the standard lexical conventions of C, so
you may use arbitrary spacing and even include comments if you
really wish.

Note that ``rename:`` options supply names for new objects
(and types) that are being imported into Dylan. You cannot,
therefore, simply rename ``bool`` to ``<Boolean>`` to make
it equivalent to the existing type -- this would simply result
in a name conflict. For these purposes, you would instead use
the ``equate`` and ``map`` operations, which will be described
later. (In fact, if the C declaration had defined a type name
``boolean``, you might have to explicitly rename it to something
else in order to avoid name conflicts with the existing type.
Of course, in the above example, the ``gc-`` prefix would be
sufficient to make the name unique.)

.. _melange-anonymous-types:

Anonymous Types
---------------

The alien names described above can also be used to
refer to C's so-called "anonymous types". You can therefore
refer to ``char *``, ``int [23]``, or even ``int (*) (char *foo)``
(i.e. a pointer to function which takes a string and returns
an integer) [At present, function types are not fully
supported. You should not depend upon them to work as
expected.]. The ability to refer to anonymous types is
useful because it allows you to use the ``rename`` option to
provide explicit names for such types. Normally Melange would
simply generate a an arbitrary "anonymous" identifier for the
type. Without knowing the name of this type, you could not
define new operations upon it. However, by saying, for
example, ``rename: {"char *" => <char-ptr>}``, you can
provide a convenient handle to use in defining new
operations.
