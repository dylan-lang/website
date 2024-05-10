Variable Clauses
================

Global variables declared within C header files are
translated into "getter" functions which retrieve the value of
the C variables and optional "setter" functions to modify those
values. In effect, they are treated as slots of a "null
object" -- the getter function takes no arguments and returns
the value of the variable, while the setter function takes a
single value which will be the new value of the variable. Type
mapping takes place for the arguments and results of these
functions, just as it would for slot accessors.

Variable clauses support the following options:

``getter:``
  specifies a Dylan variable name which will be used
  to hold the getter function.

``setter:``
  specifies either a Dylan variable name which will be
  used to hold the setter function, or #f to indicate that
  there should be no setter function.

``read-only:``
  specifies whether the variable should be settable. ``read-only: #t``
  is equivalent to ``setter: #f``.

``seal:``
  specifies whether the getter and setter functions
  should be sealed. Possible values are ``sealed`` or ``open``,
  and the default is taken from the ``seal-functions:`` option
  in the ``#include`` clause (or ``sealed`` if not specified
  there).

``map:``
  specifies the high-level type to which the variable
  should be mapped.

``equate:``
  specifies the low-level type to which the raw C
  value should be implicitly converted.
