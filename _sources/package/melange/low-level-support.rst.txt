Low level support facilities
============================

The high level functions for calling C routines or for
accessing global variables are all built upon a relatively small
number of built-in primitives which perform specific low-level
tasks. You should seldom have any need to deal with these
primitives directly, but they are nonetheless available should
you need to make use of them.

Melange generates code using the `Open Dylan C-FFI`_. The lower
level ``direct-c-ffi`` isn't documented yet.

.. _Open Dylan C-FFI: http://opendylan.org/documentation/library-reference/c-ffi/index.html
