Pointer Clauses
===============

"Pointer clauses" modify the definitions of pointer
declarations such as ``int *`` or ``struct foo ***``, or vector
declarations such as ``char [256]``. Like all such clauses, they
may be used to specify renamings for the classes. This is
particularly useful for pointer types since they are not
automatically assigned user-meaningful names. It also allows
specification of the ``superclasses:`` option described in
:ref:`melange-class-inheritance`. A typical use might be:

.. code-block:: dylan

    define interface
       #include "vec.h";
       pointer "int *" => <int-vector>,
          superclasses: {<c-vector>};
       pointer "struct person **" => <people>,
          superclasses: {<c-vector>};
       pointer "char [256]" => <fixed-string>;
    end interface;

This clause is particularly useful for declaring pointer
types to be subclasses of ``<c-vector>`` so that they can be
indexed via ``element``. (Note that this is not necessary for
vector declarations, since they are automatically declared to be
<c-vectors>.)
