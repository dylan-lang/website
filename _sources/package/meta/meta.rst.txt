The Meta Library
****************

.. current-library:: meta
.. current-module:: meta

Introduction
============

This is an implementation of Meta, a technique used to simplify the task
of writing parsers. `[Baker91] <#baker91>`__ describes Meta and shows
the main ideas for an implementation in Common Lisp.

    If all META did was recognize regular expressions, it would not be
    very useful. It is a programming language, however, and the
    operations [], {} and $ correspond to the Common Lisp control
    structures AND, OR, and DO.[8] Therefore, we can utilize META to not
    only parse, but also to transform. In this way, META is analogous to
    "attributed grammars" [Aho86], but it is an order of magnitude
    simpler and more efficient. Thus, with the addition of the "escape"
    operation "!", which allows us to incorporate arbitrary Lisp
    expressions into META, we can not only parse integers, but produce
    their integral value as a result. -- `[Baker91] <#baker91>`__

The macro defined here is an attempt to implement Meta (with slightly
adapted syntax) for Dylan. It is functional, but not yet optimized.

Exported facilities
===================

The ``meta``-library exports the ``meta`` module with the following
macros:

.. macro:: meta-definer

   Meta integrates the ability to parse from streams and strings in one
   facility. (The parsing of lists is not implemented yet, because it's
   rather useless in Dylan. This addition would be simple to do, though.)

   :macro-call:

     .. code-block:: dylan

         define meta name (variables) => (results)
           meta body
         end

   :parameter name: The meta-function name, which is immediately transformed into
      ``scan-``\ *name*
   :parameter variables: -- token-holders used in *meta body*.
   :parameter results: An expression returned on a successful scan. This can be omitted
      in which case, the return value will be ``#f``.
   :parameter meta body: A sequence of `Meta expressions`_ to scan

   :discussion:

     The ``meta-definer`` form works only with the ``parse-string``
     *source-type* of the :macro:`with-meta-syntax` form.

     The user of this form has control over the return value. Usually ``#t``
     is sufficient (in which case the results clause may be omitted, see
     below); however, e.g., the values of the *variables* may need to be
     manipulated during the parse phase.

   :example:

     .. code-block:: dylan

        define meta public-id(s, pub) => (pub)
          "PUBLIC", scan-s(s), scan-pubid-literal(pub)
        end meta public-id;

     This definition returns ``pub`` when it successfully scans the tokens
     "PUBLIC", (some) spaces, and a literal which ``pub`` receives. Note
     that, hereafter, the meta definition is referred to as
     ``scan-public-id`` outside the meta syntax block.

     This example, without an explicit results block, is from the ``meta``
     library:

     .. code-block:: dylan

        define meta s(c)
          element-of($space, c), loop(element-of($space, c))
        end meta s;

     Scans in at least one space (``element-of`` and ``loop`` are discussed
     in the section on `Meta expressions`_).

.. macro:: with-meta-syntax

   The guts of the :macro:`meta-definer` form; use when requiring precise
   control of variables or constructs

   :macrocall:

     .. code-block:: dylan

        with-meta-syntax
        source-type (source #key keys)
          [ variables ]
          meta;
          body
        end

   :parameter source-type: Either ``parse-stream`` or ``parse-string``.
   :parameter source: Either a stream or a string, depending on *source-type*
   :parameter #key start: If *source-type* is ``parse-string``,
     the index to start at.
   :parameter #key end: If *source-type* is ``parse-string``,
     the index to finish before.
   :parameter #key pos: If *source-type* is ``parse-string``,
     a name that will be bound to the current index during execution
     of the ``with-meta-syntax`` forms.
   :parameter meta: A `Meta expression <#meta-expressions>`__.
   :parameter body: A body. Evaluated only if parsing is successful.
   :value results: If parsing fails ``#f``, otherwise the values of *body*.

   :description:

     **Special programming aids:**

     ``variables (variable [ :: type ] [ = init ], ...);``

     Bind variables to *init*, which defaults to ``#f``;

     Future versions will have further special forms.

   :example:

     .. code-block:: dylan

        with-meta-syntax parse-stream (*standard-input*)
          body
        end with-meta-syntax;

        let query :: <string> = ask-user();
        with-meta-syntax parse-string (query, start: 23, end: 42)
          body
        end with-meta-syntax;

        with-meta-syntax parse-string (query)
          ... ['\n', finish()] ...
          values(these, values, will, be, returned);
        end with-meta-syntax;

.. macro:: collector-definer

   General facility to collect data into sequences (by default, into
   strings). Initially :macro:`with-meta-syntax` had this functionality
   integrated. This is a more modular approach.

.. macro:: with-collector

   The guts of the :macro:`collector-definer` form. This macro allows
   collecting data into a sequence.  This is similar in spirit to
   Common Lisp's ``LOOP`` clauses ``COLLECT`` and ``APPEND``, but more
   flexible. If you want to extract subsequences from a string while
   parsing it, this is the tool to use.

   :macrocall:

     .. code-block:: dylan

        with-collector operation ... #key collect, append;
          body
        end

   :parameter operation: Specifies the mode of operation. See below.
   :parameter collect: A name for a function that, called with a parameter, inserts this parameter into the sequence.
   :parameter append: A name for a function that, called with a sequence, appends this parameter to the sequence.
   :parameter body: A Dylan body (bnf).
   :value result: Normally the values of *body*. There is `minimal form <#minimal>`__ of ``with-collector``, which always returns the collected sequence.

   :discussion:

     Like ``COLLECT``, ``with-collector`` can put objects into a list.
     Unlike ``COLLECT``, it can also create vectors or write into
     already created vectors.

     **Collecting into a list or vector**

     Two simple forms of ``with-collector`` are ``into-list`` and
     ``into-vector``. They create a list or a vector and write into it. The
     sequence is available as a variable with a user-defined name:

     .. code-block:: dylan

        with-collector into-list name #key collect, append;
          body
        end

        with-collector into-vector name #key collect, append;
          body
        end

     **Writing into an existing vector**

     ``into-vector``, by default, creates a <stretchy-sequence>. If you don't
     like this behaviour, you can specify a different vector that will be
     used. For instance, if you already know how long the result will be, you
     might want to create a string in the first place.

     .. code-block:: dylan

        with-collector into-vector name = init, #key collect, append;
          body
        end


     **Using buffers**

     Normally it is not known in advance how long the result will be. What is
     really needed is a sequence that automatically reduces its size after
     processing is finished. ``into-buffer`` implements this by returning a
     subsequence of the original vector.

     Instead of a variable holding the sequence there is now a function which
     creates the subsequence.

     .. code-block:: dylan

        with-collector into-buffer function-name, #key collect, append;
          body
        end


     But how do you find out what the maximum buffer size has to be? A safe
     guess is the length of the original vector you are extracting elements
     from. The following construct automatically creates a vector of the same
     class (well, :drm:`type-for-copy`) and size as *big-one*:

     .. code-block:: dylan

        with-collector into-buffer function-name like big-one, #key collect, append;
          body
        end

     **A minimal collection form**

     If you don't need to write into vectors or use buffers, but just want to
     collect some stuff and return it, use this idiom:

    .. code-block:: dylan

       with-collector {into-list|into-vector}, #key collect, append;
         body
         // Note: Values of body will be thrown away.
       end

   :example:

     .. code-block:: dylan

        define function parse-finger-query (query :: <string>)
          with-collector into-buffer user like query, collect: collect;
            with-meta-syntax parse-string (query)
              let (whois, at, c);
              [loop(' '), {[{"/W", "/w"}, yes!(whois)], []},        // Whois switch?
               loop(' '), loop({[{'\n', '\r'}, finish()],           // Newline? Quit.
                    {['@', yes!(at), do(collect('@'))], // @? Indirect.
                     [type(<character>, c),             // Else:
                      do(collect(c))]}})];              //   Collect char
              values(whois, user(), at);
            end with-meta-syntax;
          end with-collector;
        end function parse-finger-query;

Aside from the syntactic constructors exported above, the Meta library
also provides some commonly-used forms for scanning and parsing:

.. function:: scan-s

   Scans in at least one space.

.. function:: scan-word

   Scans in a token and returns that token as a ``<string>`` instance. A
   "word" is surrounded by spaces or any of the following characters: '<',
   '>', '{', '}', '[', ']', punctuation (',', '?', or '!'), or the single-
   or double- quotation-mark.

.. function:: scan-int

   Reads in digit characters and returns an ``<integer>`` instance.

.. function:: scan-number

   Although this is not an all-encompassing conversion utility (although,
   IMHO, it's good enough to be part of the standard, once there is one,
   YMMV), it reads in just about any fixed-point number format and returns
   a :drm:`<real>` instance.

.. function:: string-to-number

   :parameter str: An instance of :drm:`<string>`, the string to convert
     to a number.
   :parameter base: An instance of :drm:`<integer>`, defaults to ``10``,
     the base of the number in the string.
   :value ans: An instance of :drm:`<real>`, the resulting number.

   :discussion:

     This really should belong to the common-dylan spec, so that instead of
     rolling their own, everyone should use this function. It is
     therefore exported to this end.

Scanning tokens usually entails using some common character types. Meta
exports the following:

.. constant:: $space

   :equivalent: Any whitespace.

.. constant:: $digit

   :equivalent: ``[0-9]``

.. constant:: $letter

   :equivalent: ``[a-zA-Z]``

.. constant:: $num-char

   :equivalent: ``$digit ++ [.eE+]``

.. constant:: $graphic-char

   :equivalent: ``[_@#$%^&*()+=~/]``

.. constant:: $any-char

   :equivalent: ``$letter ++ $num-char ++ $graphic-char``

Meta expressions
================

Meta is a small, but featureful language, so naturally it has its own
syntax. This syntax is adapted to Dylan's way of writing things, of
course.

There are several basic Meta expressions implementing the core
functionality. Additionally there are some *pseudo-functions*,
syntactically function-like constructs which simplify certain tasks that
would otherwise have to be written manually.

Basic Meta expressions as described by Baker
--------------------------------------------

**Baker**

**``with-meta-syntax``**

**Description**

``fragment``

``fragment``

try to match this

``[a b c ... n]``

``[a, b, c, ..., n]``

and/try all

``{a b c ... n}``

``{a, b, c, ..., n}``

or/first hit

``@(type variable)``

``type(type, variable)``

match any *type*, store result in *variable*

.. warning:: *deprecated* ``type`` is most often used in seeing if a
   character is one of several possibilities. Use ``element-of`` instead.

``$foo``

``loop(foo)``

zero or more

``!Lisp``

``(Dylan)``

call the code (and check result)

The same grammar which works for streams will works for strings. When
parsing strings, more than just one-character look-ahead is possible,
though. You can therefore not only match against characters, but also
whole substrings. This does not work when reading from a stream.

Additional pseudo-function expressions
--------------------------------------

+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| **``with-meta-syntax``**           | **Description**                                                                                            | **Could be written as**                               |
+====================================+============================================================================================================+=======================================================+
| ``do(Dylan)``                      | call the code and continue (whatever the result is)                                                        | ``(Dylan; #t)``                                       |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``finish()``                       | finish parsing successfully                                                                                | not possible                                          |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``test(predicate)``                | Match against a predicate.                                                                                 | not possible                                          |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``test(predicate, variable)``      | Match against a predicate, saving the result.                                                              | not possible                                          |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``peeking(variable, test)``        | Save result first, so that expression test can use it.                                                     | not possible;                                         |
|                                    |                                                                                                            |  **Warning:** *deprecated, use*\ ``peek`` *instead*   |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``peek(variable, test)``           | Look one character ahead and store in *variable* if it passes *test*. Leave the character on the stream.   | not possible                                          |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``element-of(sequence, variable)`` | Sees if the *variable* (a character) is a member of the *sequence*, storing the result                     | { 'a', 'b', 'c' } (but not storing result)            |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``yes!(variable)``                 | Set *variable* to ``#t`` and continue.                                                                     | ``(variable := #t)``                                  |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``no!(variable)``                  | Set *variable* to ``#f`` and continue.                                                                     | ``(variable := #f; #t)``                              |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``set!(variable, value)``          | Set *variable* to *value* and continue.                                                                    | ``(variable := value; #t)``                           |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+
| ``accept(variable)``               | Match anything and save result.                                                                            | ``type(<object>, variable)``                          |
+------------------------------------+------------------------------------------------------------------------------------------------------------+-------------------------------------------------------+

Example code
============

Parsing an integer (base 10)
----------------------------

Common Lisp version:

.. code-block:: common-lisp

    (defun parse-integer (&aux (s +1) d (n 0))
      (and
       (matchit
        [{#\+ [#\- !(setq s -1)] []}
        @(digit d) !(setq n (digit-to-integer d))
        $[@(digit d) !(setq n (+ (* n 10) (digit-to-integer d)))]])
       (* s n)))


Direct translation to Dylan:

.. code-block:: dylan

    define constant <digit> = one-of('0','1','2','3','4','5','6','7','8','9');

    define function parse-integer (source :: <stream>);
      let s = +1; // sign
      let n = 0;  // number
      with-meta-syntax parse-stream (source)
        variables(d);
        [{'+', ['-', (s := -1)], []},
         type(<digit>, d), (n := digit-to-integer(d)),
         loop([type(<digit>, d), (n := digit-to-integer(d) + 10 * n)])];
        (s * n)
      end with-meta-syntax;
    end function parse-integer;


Alternative version:

.. code-block:: dylan

    // this will actually return a fn named 'scan-int', not 'parse-int'
    define collector int(i) => (as(<string>, str).string-to-integer)
      loop([element-of("+-0123456789", i), do(collect(i))])
    end collector int;


Parsing finger queries
----------------------

.. code-block:: dylan

    define function parse-finger-query (query :: <string>)
      with-collector into-buffer user like query (collect: collect)
        with-meta-syntax parse-string (query)
          variables (whois, at, c);
          [loop(' '), {[{"/W", "/w"}, yes!(whois)], []},        // Whois switch?
           loop(' '), loop({[{'\n', '\r'}, finish()],           // Newline? Quit.
                {['@', yes!(at), do(collect('@'))], // @? Indirect.
                 [accept(c), do(collect(c))]}})];   // then collect char
          values(whois, user(), at);
        end with-meta-syntax;
      end with-collector;
    end function parse-finger-query;


References
==========

`[Baker91] <lisp-meta.htm>`__ Baker, Henry. "Pragmatic Parsing in Common
Lisp". *ACM Lisp Pointers 4, 2* (Apr-Jun 1991), 3-15.
