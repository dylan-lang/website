Proposed modifications
======================

Although Melange seems to be fairly useful in its present
form, we are currently considering a number of ways in which it
may be made more useful. This section contains a brief
discussion of several potential changes which may be implemented
in the future.

.. _melange-enumeration-clauses:

Enumeration clauses
-------------------

At present, there is no way to modify the default
handling of a C enumeration declaration. It is clear that you
might wish a mechanism to specify several different explicit
options: prefixes for the enumeration constants;
re-specification of constant values; and, of course, explicit
``import:`` and ``exclude:`` options.

.. _melange-map-equate-inheritance:

Inheritance of "map" and "equate" options
-----------------------------------------

There are some cases in which a set of types imported
within one interface definition might be used extensively
within another. In the present implementation, the two
interface definitions would be handled independently and
equivalences between types would not be recognized in the
absence of explicit ``equate:`` options.

One proposed solution would involve the ability to
explicitly "use" one interface definition within another. This
would result in all identically named types being implicitly
equated and all top-level ``map:`` options being inherited. The
"use" clause could support roughly the same syntax as the
"use" clauses in library and module definitions.In order to
make this work, it would be necessary to assign arbitrary
names to interface definitions. This would have the added
benefit of making them more consistent with other standard
Dylan definition forms.

If this change were implemented, a typical interface
definition might look something like the following:

.. code-block:: dylan

    define interface date
       #include "date.h";
       use time, import: {"struct time"};
    end interface date;

A less ambitious version might remain compatible with
the current syntax by replacing the interface name with an
"interface-name" option, which would default to the root of
the file name. Thus,

.. code-block:: dylan

    define interface
       #include "date.h",
          interface-name: "date";
    end interface;

would yield the same effect as the previous example.

.. _melange-merging-map-equate:

Re-merging of the "equate:" and "map:" options
----------------------------------------------

It has been pointed out that the current method of
specifying low-level and high-level mappings, while
sufficiently expressive, is somewhat verbose and confusing. It
would therefore be good to find an alternative
notation.

It has been suggested that definitions like:

.. code-block:: dylan

    define interface
       #include "dirent.h",
          equate: {"char *" => <c-string>},
          map: {"char *" => <byte-string>};
    end interface;

might be replaced by something like:

.. code-block:: dylan

    define interface
       #include "dirent.h",
          equate-and-map: {"char *" => <c-string> => <byte-string>};
    end interface;

or

.. code-block:: dylan

    define interface
       #include "dirent.h";
       transform "char *",
          low-level: <c-string>,
          high-level: <byte-string>;
    end interface;
