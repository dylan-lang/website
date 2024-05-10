Other File Options
==================

There are a few other options that may be specified within
an ``#include`` clause, but which do not fit into any of the above
categories. These options are ``define:``, ``undefine:``,
``seal-functions:`` and ``read-only:``.

The ``define:`` and ``undefine:`` options control the C
preprocessor definitions which will be implicitly defined
during parsing of the header files. If you specify neither of
these options, Melange will use a default set of definitions
which correspond to those used by a typical C compiler for the
machine you are running on. The define options takes a string
containing a single C token and an optional string or integer
literal, which will be used as the expansion. (If no literal is
specified, the token will be expanded to ``1``.)  The ``undefine:``
removes one or more of the default definitions. You might, for
example, use:

.. code-block:: dylan

    define interface
       #include "gc.h",
          define: {"PMAX", "BSD_VERSION" => "4.3"},
          undefine: {"HPUX"};
    end interface;

The ``seal-functions:`` option controls whether the various
imported functions and slot accessors will be sealed or open. By
default, functions are sealed, but you may explicitly specify
this by using ``seal-functions: sealed`` or reverse it by using
``seal-functions: open``. Melange does not support the Creole's
``inline`` sealing option as this is handled with the
``inline-functions:`` option instead.

The ``inline-functions:`` option specifies how functions
should be inlined. It may have values of ``inline``, ``inline-only``,
``may-inline`` or ``not-inline``.

The ``read-only:`` option specifies whether setter functions
should be defined for slot and object accessors. They will be
defined by default, but if you specify ``read-only: #t``, no
setters will be defined.

The effects of the ``seal-functions:``, ``inline-functions:`` and
``read-only:`` options can be modified for particular container
types. We will explain how to do this in a later section.
