*************************************
Using the Melange Interface Generator
*************************************

The Melange interface generator provides a mechanism for
providing access to native C code. It is modeled upon Apple
Computer's Creole, and shares Creole's goals of automatically
providing full support for a foreign interface based upon existing
interface descriptions. It also, like Creole, provides mechanisms for
explicitly adapting these interfaces to provide a greater match
between C and Dylan data models.

Melange, however, differs from Creole in a number of
significant ways. This document, therefore, provides a gentle
introduction to Melange without attempting to build upon any existing
descriptions of Creole.

.. toctree::
   :maxdepth: 2

   introduction
   concrete-example
   basic-use
   importing-headers
   specifying-object-names
   type-definitions
   translating-object-representations
   other-file-options
   function-clauses
   struct-union-clauses
   pointer-clauses
   constant-clauses
   variable-clauses
   low-level-support
   known-limitations
   proposed-modifications
   historical-notes
   copyright
