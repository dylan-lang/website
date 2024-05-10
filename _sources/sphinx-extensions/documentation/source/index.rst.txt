**************************
  Dylan Domain Reference
**************************

.. contents::


`Dylan Reference Manual`:t: links
=================================


Roles
-----

``:dylan:drm:``
^^^^^^^^^^^^^^^

   Reference to a page or exported name in the `Dylan Reference Manual`:t:.

   :Syntax 1:  ``:dylan:drm:`LINKNAME```
   :Syntax 2:  ``:dylan:drm:`DISPLAYED TEXT <LINKNAME>```

   *LINKNAME* is an exported name or the last part of the URL of a page or
   section. Exported names are converted into partial URLs per the file
   configured by `dylan_drm_index`_. The partial URL is appended to the base URL
   configured by the `dylan_drm_url`_.


Configurables
-------------

``dylan_drm_url``
^^^^^^^^^^^^^^^^^

   The base URL of the `Dylan Reference Manual`:t:. Defaults to the base URL of
   the copy at `<http://opendylan.org>`_.

``dylan_drm_index``
^^^^^^^^^^^^^^^^^^^

   A file listing Dylan names and the corresponding `Dylan Reference Manual`:t:
   partial URLs. Each line is a correspondence. The first word is the Dylan
   name, followed by whitespace, then the remainder is the partial URL. Defaults
   to partial URLs corresponding to the copy of the `Dylan Reference Manual`:t:
   at `<http://opendylan.org>`_.


Library, module, and binding documentation
==========================================

The Dylan domain generates an API index in a file called
``dylan-apiindex.html``. Unfortunately, you have to link to it by filename, e.g.
::

  * `API Index <dylan-apiindex.html>`_


Directives with content
-----------------------

``dylan:library::``
^^^^^^^^^^^^^^^^^^^

   A library. You can document the modules exported by the library inside or
   after this directive, or elsewhere via `dylan:current-library::`_.

   :Syntax:       ``.. dylan:library:: NAME``
   :Options:      None
   :Doc Fields:   `:summary:`_, `:discussion:`_, `:seealso:`_
   :References:   `:dylan:lib:`_

``dylan:module::``
^^^^^^^^^^^^^^^^^^

   A module. You can document the names exported by the module inside or after
   this directive, or elsewhere via `dylan:current-module::`_.

   :Syntax:       ``.. dylan:module:: NAME``
   :Options:      `:library:`_
   :Doc Fields:   `:summary:`_, `:discussion:`_, `:seealso:`_
   :References:   `:dylan:mod:`_

``dylan:class::``
^^^^^^^^^^^^^^^^^

   A class.

   :Syntax:       ``.. dylan:class:: NAME``
   :Options:      `:open:`_, `:sealed:`_, `:primary:`_, `:free:`_, `:abstract:`_,
                  `:concrete:`_, `:instantiable:`_, `:uninstantiable:`_,
                  `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:supers:`_, `:keyword:`_, `:slot:`_, `:summary:`_,
                  `:discussion:`_, `:conditions:`_, `:operations:`_, `:example:`_,
                  `:seealso:`_
   :References:   `:dylan:class:`_

   Example::

      .. class:: <vector>
         :open:

         :supers: `<array>`:class
         :keyword size:  An instance of `<integer>`:class: specifying the size
                         of the vector. The default value is ``0``.
         :keyword required fill:
             An instance of `<object>`:class: specifying the initial value for
             each element of the vector. The default value is ``#f``.

``dylan:generic-function::``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   A generic function.

   :Syntax:       ``.. dylan:generic-function:: NAME``
   :Options:      `:open:`_, `:sealed:`_, `:adjectives:`_, `:library:`_,
                  `:module:`_
   :Doc Fields:   `:param:`_, `:value: (1)`_, `:signature:`_, `:summary:`_,
                  `:discussion:`_, `:conditions:`_, `:example:`_, `:seealso:`_
   :References:   `:dylan:gf:`_

   Example::

      .. generic-function:: member?
         :sealed:

         :param value:        An instance of `<object>`:class:.
         :param collection:   An instance of `<collection>`:class:.
         :param #key test:    An instance of `<function>`:class:. The default is
                              `==`:gf:.
         :value bool:         An instance of `<boolean>`:class:.

``dylan:method::``
^^^^^^^^^^^^^^^^^^

   A method of a generic function.

   :Syntax:       ``.. dylan:method:: NAME``
   :Options:      `:specializer:`_, `:sealed:`_, `:adjectives:`_, `:library:`_,
                  `:module:`_
   :Doc Fields:   `:param:`_, `:value: (1)`_, `:signature:`_, `:summary:`_,
                  `:discussion:`_, `:conditions:`_, `:example:`_, `:seealso:`_
   :References:   `:dylan:meth:`_

   References to a method must be disambiguated by enclosing *SPECIALIZER* in
   parentheses, as shown by the reference to ``type-for-copy`` in the following
   example. The specializer is author-defined and does not necessarily have to
   reflect all the parameters of the method.

   Example::

      .. method:: copy-sequence
         :specializer: <range>

         :param source:       An instance of `<range>`:class:.
         :param #key start:   An instance of `<integer>`:class. The default is
                              ``0``.
         :param #key end:     An instance of `<integer>`:class. The default is
                              the size of *source*.
         :value new-range:    A freshly allocated instance of `<range>`:class:.

         *new-range* will be a `<range>`:class: even though the return value of
         `type-for-copy(<range>)`:meth: is a `<list>`:class:.

``dylan:function::``
^^^^^^^^^^^^^^^^^^^^

   A function that does not belong to a generic function.

   :Syntax:       ``.. dylan:function:: NAME``
   :Options:      `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:param:`_, `:value: (1)`_, `:signature:`_, `:summary:`_,
                  `:discussion:`_, `:conditions:`_, `:example:`_, `:seealso:`_
   :References:   `:dylan:func:`_

``dylan:primitive::``
^^^^^^^^^^^^^^^^^^^^^

   A primitive operation.

   :Syntax:       `.. dylan:primitive:: NAME``
   :Options:      `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:param:`_, `:value: (1)`_, `:signature:`_, `:summary:`_,
                  `:discussion:`_, `:conditions:`_, `:example:`_, `:seealso:`_
   :References:   `:dylan:prim:`_

``dylan:constant::``
^^^^^^^^^^^^^^^^^^^^

   A constant.

   :Syntax:       ``.. dylan:constant:: NAME``
   :Options:      `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:type:`_, `:value: (2)`_, `:summary:`_, `:discussion:`_,
                  `:example:`_, `:seealso:`_
   :References:   `:dylan:const:`_

``dylan:type::``
^^^^^^^^^^^^^^^^

   A type.

   :Syntax:       ``.. dylan:type:: NAME``
   :Options:      `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:type:`_, `:value: (2)`_, `:summary:`_, `:discussion:`_,
                  `:example:`_, `:supertypes:`_, `:operations:`_, `:equivalent:`_,
                  `:seealso:`_
   :References:   `:dylan:type:`_

``dylan:variable::``
^^^^^^^^^^^^^^^^^^^^

   A variable.

   :Syntax:       ``.. dylan:variable:: NAME``
   :Options:      `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:type:`_, `:value: (2)`_, `:summary:`_, `:discussion:`_,
                  `:example:`_, `:seealso:`_
   :References:   `:dylan:var:`_

``dylan:macro::``
^^^^^^^^^^^^^^^^^

   A macro.

   :Syntax:       ``.. dylan:macro:: NAME``
   :Options:      `:statement:`_, `:function:`_, `:defining:`_, `:macro-type:`_,
                  `:adjectives:`_, `:library:`_, `:module:`_
   :Doc Fields:   `:param:`_, `:value: (1)`_, `:macrocall:`_, `:summary:`_,
                  `:discussion:`_, `:example:`_, `:seealso:`_
   :References:   `:dylan:macro:`_


Directives without content
--------------------------

``dylan:current-library::``
^^^^^^^^^^^^^^^^^^^^^^^^^^^

   Sets the library currently being documented when the actual library
   documentation is elsewhere. You can document the modules exported by the
   library after this directive.

   :Syntax:    ``.. dylan:current-library:: LIBRARY``
   :Options:   None

``dylan:current-module::``
^^^^^^^^^^^^^^^^^^^^^^^^^^

   Sets the module currently being documented when the actual module
   documentation is elsewhere. You can document the names exported by the module
   after this directive.

   :Syntax:    ``.. dylan:current-module:: MODULE``
   :Options:   None


Directive doc fields
--------------------

Doc fields appear in the directive's content. Doc fields must be separated from
the directive and any directive options by a blank line.

``:summary:``
^^^^^^^^^^^^^

   A brief summary of a Dylan language element.

   :Syntax:    ``:summary: DISCUSSION``
   :Synonyms:  None

``:discussion:``
^^^^^^^^^^^^^^^^

   A discussion of a Dylan language element.

   :Syntax:    ``:discussion: DISCUSSION``
   :Synonyms:  ``:description:``

``:seealso:``
^^^^^^^^^^^^^

   A set of items that are related to the current element.

   :Syntax:    ``:seealso: OTHER ELEMENTS``
   :Synonyms:  None

``:example:``
^^^^^^^^^^^^^

   An example of the use of a binding. This doc field may appear multiple times.

   :Syntax:    ``:example: EXAMPLE``
   :Synonyms:  None

``:supers:``
^^^^^^^^^^^^

   A superclass of a class. This doc field may appear multiple times.

   :Syntax:    ``:supers: DESCRIPTION``
   :Synonyms:  ``:superclasses:``, ``:super:``, ``:superclass:``

``:keyword:``
^^^^^^^^^^^^^

   An init-keyword of a class. This doc field may appear multiple times.

   :Syntax 1:    ``:keyword NAME: DESCRIPTION``
   :Syntax 2:    ``:keyword required NAME: DESCRIPTION``
   :Synonyms:  ``:init-keyword:``

   See `dylan:class::`_ for an example.

``:slot:``
^^^^^^^^^^

   A slot of a class. This doc field may appear multiple times.

   :Syntax:    ``:slot NAME: DESCRIPTION``
   :Synonyms:  ``:getter:``

``:operations:``
^^^^^^^^^^^^^^^^

   A list of methods or functions applicable to a class.

   :Syntax:    ``:operations: LIST``
   :Synonyms:  ``:methods:``, ``:functions:``

``:param:``
^^^^^^^^^^^

   A parameter of a generic function or method. This doc field may appear
   multiple times.

   :Syntax 1:  ``:param NAME: DESCRIPTION``
   :Syntax 2:  ``:param #key NAME: DESCRIPTION``
   :Syntax 3:  ``:param #rest NAME: DESCRIPTION``
   :Synonyms:  ``:parameter:``

   See `dylan:generic-function::`_ and `dylan:method::`_ for examples.

``:value:`` (1)
^^^^^^^^^^^^^^^

   A return value of a generic function or method. This doc field may appear
   multiple times.

   :Syntax 1:  ``:value NAME: DESCRIPTION``
   :Syntax 2:  ``:value #rest NAME: DESCRIPTION``
   :Synonyms:  ``:return:``, ``:retval:``, ``:val:``

   See `dylan:generic-function::`_ and `dylan:method::`_ for examples.

``:type:``
^^^^^^^^^^

   The type of a variable or constant.

   :Syntax:    ``:type: EXPRESSION``
   :Synonyms:  None

``:value:`` (2)
^^^^^^^^^^^^^^^

   The initial value of a variable or constant.

   :Syntax:    ``:value: EXPRESSION``
   :Synonyms:  ``:val:``

``:signature:``
^^^^^^^^^^^^^^^

   The signature of a function.

   :Syntax:    ``:signature: TEXT``
   :Synonyms:  ``:sig:``

   Example::

      .. function:: error

         :signature: ``error`` *condition* => *will never return*
         :signature:
            ``error`` *string* ``#rest`` *arguments* => *will never return*

``:macrocall:``
^^^^^^^^^^^^^^^

   The syntax of a macro call.

   :Syntax:    ``:macrocall: BODY``
   :Synonyms:  ``:call:``, ``:syntax:``

   Example::

      .. macro:: variable-definer

         :macrocall:
            .. parsed-literal::
               define { `adjective }* variable `variables` = `init`

``:conditions:``
^^^^^^^^^^^^^^^^

   A discussion of conditions signaled by a function or by a class's make or
   initialize.

   :Syntax:    ``:conditions: DISCUSSION``
   :Synonyms:  ``:exceptions:``, ``:signals:``, ``:throws:``, ``:condition:``,
               ``:exception:``

``:supertypes:``
^^^^^^^^^^^^^^^^

   A supertype of a type. This doc field may appear multiple times.

   :Syntax:    ``:supertypes: DESCRIPTION``
   :Synonyms:  ``:supertype:``, ``:super:``, ``:supers:``

``:equivalent:``
^^^^^^^^^^^^^^^^

   The equivalent of a type.

   :Syntax:    ``:equivalent: DESCRIPTION``


Directive options
-----------------

Directive options appear immediately after the directive with no intervening
blank lines.

``:library:``
^^^^^^^^^^^^^

   Sets the current library, also affecting documentation following the
   directive. Mostly for automatically-generated documentation; hand-written
   documentation can use `dylan:current-library::`_.

   :Syntax: ``:library: NAME``

``:module:``
^^^^^^^^^^^^^

   Sets the current module, also affecting documentation following the
   directive. Mostly for automatically-generated documentation; hand-written
   documentation can use `dylan:current-module::`_.

   :Syntax: ``:module: NAME``

``:specializer:``
^^^^^^^^^^^^^^^^^

   A way to distinguish one method from another -- generally a list of the types
   of its required parameters. It cannot contain parentheses. This option is
   required in `dylan:method::`_ directives.

   :Syntax: ``:specializer: EXPRESSION, EXPRESSION, ...``

   See `dylan:generic-function::`_ and `dylan:method::`_ for examples.

``:open:``
^^^^^^^^^^

   Indicates an open class or generic function. Synonymous with ``:adjectives:
   open``.

   :Syntax: ``:open:``

``:primary:``
^^^^^^^^^^^^^

   Indicates a primary class. Synonymous with ``:adjectives: primary``.

   :Syntax: ``:primary:``

``:free:``
^^^^^^^^^^

   Indicates a free class. Synonymous with ``:adjectives: free``.

   :Syntax: ``:free:``

``:abstract:``
^^^^^^^^^^^^^^

   Indicates an abstract class. Synonymous with ``:adjectives: abstract``.

   :Syntax: ``:abstract:``

``:concrete:``
^^^^^^^^^^^^^^

   Indicates a concrete class. Synonymous with ``:adjectives: concrete``.

   :Syntax: ``:concrete:``

``:sealed:``
^^^^^^^^^^^^

   Indicates a sealed generic function, method, or class. Synonymous with
   ``:adjectives: sealed``.

   :Syntax: ``:sealed:``

``:instantiable:``
^^^^^^^^^^^^^^^^^^

   Indicates an instantiable class. Synonymous with ``:adjectives:
   instantiable``.

   :Syntax: ``:instantiable:``

``:uninstantiable:``
^^^^^^^^^^^^^^^^^^^^

   Indicates an uninstantiable class. Synonymous with ``:adjectives:
   uninstantiable``.

   :Syntax: ``:uninstantiable:``

``:adjectives:``
^^^^^^^^^^^^^^^^

   Adjectives to a binding. You may use this to display implementation-specific
   adjectives.

   :Syntax: ``:adjectives: ADJECTIVES``

``:statement:``
^^^^^^^^^^^^^^^

   Indicates a statement macro. Synonymous with ``:macro-type: statement``.

   :Syntax: ``:statement:``

``:function:``
^^^^^^^^^^^^^^

   Indicates a function macro. Synonymous with ``:macro-type: function``.

   :Syntax: ``:function:``

``:defining:``
^^^^^^^^^^^^^^

   Indicates a defining macro. Synonymous with ``:macro-type: defining``.

   :Syntax: ``:defining:``

``:macro-type:``
^^^^^^^^^^^^^^^^

   Describes the type of a macro, in a general sense. Free-form.

   :Syntax: ``:macro-type: TYPE``


Roles
-----

   All cross-referencing roles except `:dylan:meth:`_ have the same syntax. This
   syntax is similar to the syntax of cross-referencing roles for other
   languages, but if you use the ``!`` or ``~`` marks, you must enclose the
   target in ``< >``, and the ``~`` mark does not have any effect.

   :Syntax 1: ``:dylan:role:`LIBRARY:MODULE:NAME```
   :Syntax 2: ``:dylan:role:`TEXT <LIBRARY:MODULE:NAME>```
   :Syntax 3: ``:dylan:role:`MARK <LIBRARY:MODULE:NAME>```
   :Syntax 4: ``:dylan:role:`MARK TEXT <LIBRARY:MODULE:NAME>```

   - You may omit *LIBRARY* or *MODULE* to use the current library or module or
     link to a uniquely-named binding or module.
   - *MARK* may be ``!`` to avoid making a hyperlink, or ``~`` which does not
     have an effect at the moment.

   Examples::

      .. current-library:  io
      .. current-module:   streams

      Be sure to call `~ <dylan:dylan:copy-sequence>`:gf: to avoid
      unintentionally changing the values of the sequence.

      See `the <stream> class <<stream>>`:class: for more information.

``:dylan:lib:``
^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:library::`_ directive.

``:dylan:mod:``
^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:module::`_ directive.

``:dylan:class:``
^^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:class::`_ directive.

``:dylan:gf:``
^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:generic-function::`_ directive.

``:dylan:meth:``
^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:method::`_ directive.

   The syntax is similar to other roles.

   :Syntax 1: ``:dylan:meth:`LIBRARY:MODULE:NAME(SPECIALIZER)```
   :Syntax 2: ``:dylan:meth:`TEXT <LIBRARY:MODULE:NAME(SPECIALIZER)>```
   :Syntax 3: ``:dylan:meth:`MARK <LIBRARY:MODULE:NAME(SPECIALIZER)>```
   :Syntax 4: ``:dylan:meth:`MARK TEXT <LIBRARY:MODULE:NAME(SPECIALIZER)>```

   - The *SPECIALIZER* component matches a method directive's `:specializer:`_
     option. It cannot contain nested parentheses.
   - You may omit *LIBRARY* or *MODULE* to use the current library or module or
     link to a uniquely-named binding or module.
   - *MARK* may be ``!`` to avoid making a hyperlink, or ``~`` which does not
     have an effect at the moment.

``:dylan:func:``
^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:function::`_ directive.

``:dylan:prim:``
^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:primitive::`_ directive.

``:dylan:const:``
^^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:constant::`_ directive.

``:dylan:var:``
^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:variable::`_ directive.

``:dylan:type:``
^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:type::`_ directive.

``:dylan:macro:``
^^^^^^^^^^^^^^^^^

   Creates a cross-reference to a `dylan:macro::`_ directive.

