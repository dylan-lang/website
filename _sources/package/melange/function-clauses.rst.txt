Function Clauses
================

Imported functions can be easily invoked, in almost every
case, without any additional declarations. However, by exerting
explicit control over argument handling, the interfaces to some
functions may be made cleaner. This control is exerted via
function clauses. The primary purpose of these clauses is to
specify additional type information for specific parameters or
to specify alternative argument passing conventions. For
example, if we had two alternate ``read-integers`` functions with
the following declarations:

.. code-block:: c

    int ReadInts1(int **VectorPtr);  /* result is a count of integers */
    int *ReadInts2(int *Count);      /* result is a vector of  integers */

we might use the following interface definition:

.. code-block:: dylan

    define interface
       #include "readints.h",
          rename: {"int *" => int-vector};
       function "ReadInts1",
          output-argument: 1;
       function "ReadInts2" => Read-Integers-Vector,
          output-argument: Count;
    end interface;

This would produce two functions, both of which take 0
arguments but return two values. The first would return an
``<integer>`` following by an ``<int-vector>``, while the
second would return the ``<int-vector>`` first and the
``<integer>`` second.

.. code-block:: dylan

    let (count :: <integer>, values :: <int-vector>)
       = Read-Ints1();
    let (values :: <int-vector>, count :: <integer>)
       = Read-Integers-Vector();

The function clause consists of a function name (which is
a string), an optional renaming (as illustrated above), and an
optional sequence of "options". The options include the
following:

``inline:``
   specifies whether or not the resulting method should be
   inlined. Possible values are ``inline``, ``inline-only``,
   ``may-inline`` and ``not-inline``. The default is to not
   specify an inlining adjective.

``seal:``
   specifies whether the resulting method should be
   sealed. Possible values are sealed or open, and the
   default is taken from the value specified in the initial
   file clause. (The "default default" is sealed.)

``equate-result:``
   overrides the default interpretation of the result
   type. The named type is assumed to be fully
   defined.

``map-result:``
   specifies that ``import-value`` should be called to map the
   result value to the named type.

``ignore-result:``
   specifies that the functions result value should be
   ignored, just as if the function had been declared
   ``void``. Although you may specify any boolean literal, the
   only meaningful value is ``#t``.

``map-error-result:``
   specifies that ``import-value`` should be called to map the result value to
   the named type, but the actual result value should be ignored. This is
   useful for wrapping C APIs which return error codes that should signal
   conditions when an error occurs.

``equate-argument:``
   overrides the default interpretation of some
   argument's type. The argument may be specified by name or
   by position.

``map-argument:``
   specifies that ``export-value`` should be called to
   map the given argument into the named type. Again, the
   argument may be specified by position or by name.

``input-argument:``
   indicates that the specified argument should be
   passed by value. This is the default.

``output-argument:``
   indicates that the specified argument should be be
   treated as a return value rather than a "parameter". The
   effect is to declare that the C parameter will be passed
   by reference and that the reference variable need not be
   initialized to any object.  This option assumes that the C
   parameter will have been declared as a "pointer" type, and
   will strip one ``*`` off of the argument type. Thus, if the
   parameter declaration specifies ``int **``, the actual value
   returned will have the Dylan type corresponding to ``int *``.

``input-output-argument:``
   indicates that the specified argument should be
   considered both an input argument and that its
   (potentially modified) value should be returned as an
   additional result value. The effect is similar to that of
   ``output-argument`` except that the reference variable will
   be initialized with the argument value.

The following (nonsensical) example demonstrates all of
the options, as they might be applied to the functions:

.. code-block:: c

    extern struct object *bar(int first, int *second, struct object **third);
    extern baz(char first, struct object *second);

.. code-block:: dylan

    define interface
       #include "demo.h";
       function "bar",
          seal: open,
          equate-result: <object>,
          map-result: <bar-object>,
          input-argument: first,   // passed normally
          output-argument: 2,      // nothing passed in, second result value
                // will be <integer>
          input-output-argument: third;   // passed in as second argument,
                // returned as third result
       function "baz" => arbitrary-function-name,
          seal: sealed,      // default
          ignore-result: #t,
          equate-argument: {second => <object>},
          map-argument: {2 => <baz-object>};
    end interface;
