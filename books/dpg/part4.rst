Part 4. Advanced Topics
=======================

.. toctree::
   :maxdepth: 1

   inherit
   perform
   exceptions
   macros

:doc:`inherit`, describes how multiple inheritance works in
Dylan. It describes how method dispatch is affected by multiple
inheritance. It gives an example of using the mix-in style of designing
classes with multiple inheritance.

:doc:`perform`, describes the fundamental tradeoff between
performance and flexibility. You can take advantage of Dylan’s dynamic
nature during the initial stages of development. Later on, when your
application is nearing completion, you can optimize the performance of
the program (and sacrifice flexibility, which presumably is no longer
needed).

:doc:`exceptions`, describes how to use Dylan facilities to help
create reliable programs in the face of exceptions — unexpected events
that occur during program execution.

:doc:`macros`, describes how to define macros in Dylan. Macros
can be used for abbreviation, abstraction, simplification, or
structuring. They are also useful for delaying evaluation of arguments.
