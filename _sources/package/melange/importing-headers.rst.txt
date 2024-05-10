Importing Header Files
======================

You import C definitions into Dylan by specifying one or more
header files in an ``#include`` clause. This may take one of two
different forms:

.. code-block:: dylan

    define interface
       #include "file1.h";
    end interface;

or

.. code-block:: dylan

    define interface
       #include {"file1.h", "file2.h"};
    end interface;

Melange will parse all of the named files in the specified
order, and produce Dylan equivalents for (i.e. "import") some
fraction of declarations in these files. By default, Melange
will import all of the declarations from the named files, and
any declarations in recursively included files (i.e. those
specified via ``#include`` directives in the ``.h`` file) which are
referenced by imported definitions. It will not, however, import
every declaration in recursively included files. This insures
that you will see a complete, usable, set of declarations
without having to closely control the importation process. If
you wish to exert more control over the set of objects to be
imported, you may do so via the ``import``, ``exclude``, and
``exclude-file`` options.

If you only need a small set of definitions from a set of
imported files, you can explicitly specify the complete list of
declarations to be imported by using the ``import:`` option. You
could, for example, say:

.. code-block:: dylan

    define interface
       #include "gc.h",
          import: {"scavenge", "transport" => move};
    end interface;

This would result in Dylan definitions for ``scavenge``,
``move``, ``<obj_t>``, and ``<obj>``.  The latter types
would be dragged in because they are referenced by the two
imported functions.  Again, if you equated ``obj_t`` to
``<object>`` then neither of the types would be imported. The
second import in the above example performs a renaming at the
same time as it specifies the object to be imported. Other forms
specify global behaviors. ``Import: all`` will cause Melange to
import every "top level" definition which is not explicitly
excluded. ``Import: all-recursive`` causes it to import
definitions from recursively included files as well. ``Import:
none`` restricts importation to those declarations which are
explicitly imported or are used by an imported declaration.  You
may also use the ``import:`` option to specify importation
behavior on a per-file basis. The options

.. code-block:: dylan

    import: "file.h" => {"import1", ...}
    import: "file.h" => all
    import: "file.h" => none

work like the options described above, except that they
only apply to the symbols in a single imported file.  The
``exclude:`` and ``exclude-file:`` options may be used to keep one
or more unwanted definitions from being imported. For example,
you could use:

.. code-block:: dylan

    define interface
       #include "gc.h",
          exclude: {"scavenge", "transport"},
          exclude-file: "gc1.h";
    end interface;

This would prevent the two named functions and everything
in the named file from being imported, while still including all
of the other definitions from ``gc.h``. Note that these options
should be used with care, as they can easily result in
"incomplete" interfaces in which some declarations refer to
types which are not defined. This could result in errors in the
generated Dylan code. (The ``import: file => none`` option
described above is a safer way of achieving an effect similar to
``exclude-file:``.)

You may also prevent some type declarations from being
imported by using the ``equate:`` option (described in a later
section). If, for example, you equated ``obj_t`` to
``<object>``, then Melange would ignore the definition for
``obj_t`` and simply assume that the existing definition for
``<object>`` was sufficient.

You may have any number of ``import:``, ``exclude:``, and
``exclude-file:`` options, and may name the same declarations in
multiple clauses. ``Exclude:`` options take priority over
``import:``. If no ``import:`` options are specified, the system
will import all non-excluded symbols, just as if you had said
``import: all``.
