Part 1. Basic Concepts
======================

.. toctree::
   :maxdepth: 1

   intro-new
   start
   oo-1
   usr-class
   offset
   multi
   modularity
   time-code


:doc:`intro-new`, describes the goals of Dylan, and tells you
where Dylan fits in the world of programming languages.

:doc:`start`, is a practical guide for getting started using
Dylan. It shows the look and feel of a hypothetical Dylan listener,
introduces the most basic concepts of Dylan, and presents a complete
Dylan program. You can type in these examples and experiment with Dylan.

:doc:`oo-1`, introduces the concepts of methods, built-in
classes, class inheritance, and explains what it means to be an object.

In Chapters :doc:`usr-class` through :doc:`modularity`, we start to develop
an example of a library that represents different kinds of time and
position. A *library* is a complete unit of code that can be used by
many different clients. Our eventual goal in this book is to develop
a sample application that handles the scheduling of aircraft that are
arriving at, and departing from, an airport. For more information, see
:doc:`design`. The airport application will use the time and position
library.

Also in Chapters :doc:`usr-class` through :doc:`modularity`, we show how to
write object-oriented programs in Dylan. We explain class and method
definition, class inheritance, method dispatch, and modularity.

:doc:`time-code` contains the code developed in :doc:`part1` as a
complete working library.
