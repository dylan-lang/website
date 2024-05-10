Constant Clauses
================

Constant clauses are used to override the values of
constants specified in header files (i.e. ``#define MAXINT
27``). The ``value:`` option, which is the only one supported,
specifies a Dylan literal which will be taken as the value of
the named constant. A typical use might be:

.. code-block:: dylan

    define interface
       #include "const.h";
       constant "MAXINT" => $maximum-fixed-integer,
          value: 9999999;
    end interface;
