Historical Notes
================

Melange was originally created as part of the Gwydion
project at Carnegie Mellon University. As such, it was
built to target the mindy interpreter and the d2c compiler.

It has since been converted to target the OpenDylan
environment, using OpenDylan's C-FFI support.

Differences from Creole
-----------------------

It would be difficult to produce an exhaustive list of the
differences between Creole and Melange. We can, however, include
a brief examination of the most important incompatibilities
between the two systems.

- Creole's ``type:`` options have been replaced by
  Melange's ``equate:`` and ``map:`` options.
- Creole's "access path" options have been replaced by
  ``object-file:`` and ``mindy-include-file:``.
- The interface to ``import-value`` and ``export-value`` differ
  between the two systems.
- Melange does not inherit type mappings from other
  ``define interface`` forms.
- Creole does not import definitions from "recursively
  included" header files, even if they are referenced by
  definitions which are imported.
- Creole does not support C vectors or "sub-structures" as
  first class objects.
- Melange does not presently support callbacks,
  ``export-temporary-value``, ``<pascal-string>``,
  ``with-stack-structure``, ``with-stack-block``, or
  ``alien-method``.
- Creole will never consider instances of two distinct
  statically typed pointer classes to be ``=``, even if they
  refer to the same address.
