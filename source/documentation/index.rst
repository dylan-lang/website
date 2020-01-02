.. raw:: html

  <div class="row-fluid">
    <div class="span3 bs-docs-sidebar">
      <ul class="nav nav-list bs-docs-sidenav" data-spy="affix">
        <li><a href="#cheat-sheets"><i class="icon-chevron-right"></i> Cheat Sheets</a></li>
        <li><a href="#learning-dylan"><i class="icon-chevron-right"></i> Learning Dylan</a></li>
        <li><a href="#references"><i class="icon-chevron-right"></i> References</a></li>
        <li><a href="#external-libraries-and-tools"><i class="icon-chevron-right"></i> External Libraries and Tools</a></li>
        <li><a href="#articles"><i class="icon-chevron-right"></i> Articles</a></li>
        <li><a href="#publications"><i class="icon-chevron-right"></i> Publications</a></li>
        <li><a href="#for-open-dylan-developers"><i class="icon-chevron-right"></i> For Open Dylan Developers</a></li>
        <li><a href="#archived-documentation"><i class="icon-chevron-right"></i> Archived Documentation</a></li>
      </ul>
    </div>
    <div class="span9">

*************
Documentation
*************

.. warning:: We are in the process of converting over to a new documentation
   publishing system. Some documents are not entirely correct yet. We've
   retained links to the 'Old HTML' where relevant. These will be going away
   in the near future.
   :class: alert alert-block alert-warning

Cheat Sheets
============

.. raw:: html

     <div class="alert alert-block alert-info">
       <p>Quick one-page sheets for common tasks.</p>
     </div>

.. hlist::

   * `Basics of Dylan Syntax <cheatsheet.html>`_
   * `Iteration <cheatsheets/iteration.html>`_
   * `Conditionals <cheatsheets/conditionals.html>`_
   * `Collections <cheatsheets/collections.html>`_
   * `For Scheme programmers <cheatsheets/scheme.html>`_

Learning Dylan
==============

.. raw:: html

     <div class="alert alert-block alert-success">
       <p>Just getting started with Open Dylan?  We recommend that
       you read the <a href="intro-dylan/">Introduction to Dylan</a>
       to get a feel for the language. After that, you can broaden
       your knowledge with the <a href="../books/dpg/">Dylan Programming</a>
       book.</p>
     </div>

`An Introduction to Dylan <intro-dylan/index.html>`_ [`pdf <intro-dylan/IntroductiontoDylan.pdf>`__] [`epub <intro-dylan/AnIntroductiontoDylan.epub>`__]
    This tutorial is written primarily for those with solid programming
    experience in C++ or another object-oriented, static language. It
    provides a gentler introduction to Dylan than does the Dylan Reference
    Manual (DRM).

`Dylan Programming <http://opendylan.org/books/dpg/>`_ [`pdf <http://opendylan.org/books/dpg/DylanProgramming.pdf>`__] [`epub <http://opendylan.org/books/dpg/DylanProgramming.epub>`__]
    A good, book length Dylan tutorial by several Harlequin employees.

`Getting Started with the Open Dylan Command Line Tools <getting-started-cli/index.html>`_ [`pdf <getting-started-cli/GettingStartedWithTheOpenDylanCLI.pdf>`__] [`epub <getting-started-cli/GettingStartedWithTheOpenDylanCLI.epub>`__]
    Describes development using the Open Dylan command line tools
    and editor integration (like emacs). This is mainly useful for
    people using Open Dylan on Linux, FreeBSD and Mac OS X.

`Getting Started with the Open Dylan IDE <getting-started-ide/index.html>`_ [`pdf <getting-started-ide/GettingStartedWithTheOpenDylanIDE.pdf>`__] [`epub <getting-started-ide/GettingStartedWithTheOpenDylanIDE.epub>`__] [`old HTML <http://web.archive.org/web/20170102232752/http://opendylan.org/documentation/opendylan/env/index.htm>`__]
    Describes Open Dylan's integrated development environment which
    is available for Windows.

`Building Applications Using DUIM <building-with-duim/index.html>`_ [`pdf <building-with-duim/BuildingApplicationsWithDUIM.pdf>`__] [`epub <building-with-duim/BuildingApplicationsWithDUIM.epub>`__] [`old HTML <http://web.archive.org/web/20170102232826/http://opendylan.org/documentation/opendylan/dguide/index.htm>`__]
    Describes how to use DUIM (Dylan User Interface Manager),
    the portable window programming toolkit. This is only useful
    if you are using Open Dylan on Windows.

References
==========

.. raw:: html

     <div class="alert alert-block alert-info">
       <p>These are some lengthier reference materials. While they
       make for dry reading, they're full of invaluable information!</p>
     </div>

`Dylan Reference Manual <http://opendylan.org/books/drm/>`_ (`Errata <http://opendylan.org/books/drm/drm_errata.html>`_)
    The official definition of the Dylan language and standard library.

`Dylan Library Reference <library-reference/index.html>`_ [`pdf <library-reference/DylanLibraryReference.pdf>`__] [`epub <library-reference/DylanLibraryReference.epub>`__]
    Describes the Open Dylan implementation of the Dylan language, a
    core set of Dylan libraries, and a library interchange mechanism.
    The core libraries provide many language extensions, a threads
    interface, and object finalization, printing and output formatting modules,
    a streams module, a sockets module, and modules providing an
    interface to operating system features such as the file system,
    time and date information, the host machine environment, as well
    as a foreign function interface and some low-level access to the
    Microsoft Win32 API.

`DUIM library reference <duim-reference/index.html>`_ [`pdf <duim-reference/DUIMReference.pdf>`__] [`epub <duim-reference/DUIMReference.epub>`__] [`old HTML <http://web.archive.org/web/20170102233258/http://opendylan.org/documentation/opendylan/dref/index.htm>`__]
    Describes the libraries forming DUIM (Dylan User Interface Manager),
    the portable window programming toolkit. It complements
    Building Applications Using DUIM.


External Libraries and Tools
============================

`Binary-Data </documentation/binary-data/>`_ [`pdf </documentation/binary-data/BinaryData.pdf>`__] [`epub </documentation/binary-data/BinaryData.epub>`__]
    The binary-data library provides an extension to the Dylan language for parsing and assembling binary data using high level Dylan objects.

`Concurrency </documentation/concurrency/>`_ [`pdf </documentation/concurrency/ConcurrencyUserGuide.pdf>`__] [`epub </documentation/concurrency/ConcurrencyUserGuide.epub>`__]
    The concurrency utility library.

`HTTP </documentation/http/>`_ [`pdf </documentation/http/HTTPLibraries.pdf>`__] [`epub </documentation/http/HTTPLibraries.epub>`__]
    Dylan provides libraries for writing HTTP clients and servers.

`Melange </documentation/melange/>`_ [`pdf </documentation/melange/MelangeUserGuide.pdf>`__] [`epub </documentation/melange/MelangeUserGuide.epub>`__]
    Generates Dylan code to wrap C libraries.

`Objective C Bridge </documentation/objc-dylan/>`_ [`pdf </documentation/objc-dylan/ObjectiveCBridgeUserGuide.pdf>`__] [`epub </documentation/objc-dylan/ObjectiveCBridgeUserGuide.epub>`__]
    Provides a bridge between Objective C and Dylan, allowing integration with Objective C libraries.

`Statistics </documentation/statistics/>`_ [`pdf </documentation/statistics/StatisticsUserGuide.pdf>`__] [`epub </documentation/statistics/StatisticsUserGuide.epub>`__]
    A collection of libraries for performing statistical analysis.

`Testworks </documentation/testworks/>`_ [`pdf </documentation/testworks/TestworksUserGuide.pdf>`__] [`epub </documentation/testworks/TestworksUserGuide.epub>`__]
    Testworks is Dylan's unit testing framework.

`Tracing </documentation/tracing/>`_ [`pdf </documentation/tracing/TracingUserGuide.pdf>`__] [`epub </documentation/tracing/TracingUserGuide.epub>`__]
    Tracing is an alternative to traditional logging and performance measurements.

Articles
========

.. raw:: html

    <div class="alert alert-block alert-info">
      <p>Featured articles and blog postings.</p>
    </div>

    <h3>Learning Dylan</h3>

`Dylan Macro System <../articles/macro-system/index.html>`_ by Dustin Voss.
    This article holds hard-won knowledge about how the Dylan macro system works
    and how to work around some of the gotchas that may catch a macro writer.

`Procedural Dylan <../articles/procedural-dylan/index.html>`_ by Paul Haahr.
    This essay explores Dylan from the perspective of a programmer used to
    traditional procedural languages, such as Pascal or C.

`Dylan Web in 60 Seconds <https://carlgay.wordpress.com/2010/09/03/dylan-web-in-60-seconds/>`_
    A quick introduction to web development in Dylan.  (Note that this refers
    to the "http-server" library by its original name, "koala".)

.. raw:: html

    <h3>Tools</h3>

`Development inside emacs using DIME <../news/2011/12/12/dswank.html>`_
    An exciting look at using DIME and emacs for Dylan development.

Publications
============

**Extending Dylan's type system for better type inference and error detection** [`pdf <http://www.itu.dk/~hame/ilc2010.pdf>`__] [`bib <../_static/documentation/mehnert2010.bib>`__]
    Whereas dynamic typing enables rapid prototyping and easy
    experimentation, static typing provides early error detection and
    better compile time optimization. Gradual typing provides the best
    of both worlds. This paper shows how to define and implement
    gradual typing in Dylan, traditionally a dynamically typed
    language. Dylan poses several special challenges for gradual
    typing, such as multiple return values, variable-arity methods and
    generic functions (multiple dispatch).

    In this paper Dylan is extended with function types and parametric
    polymorphism. We implemented the type system and a
    unification-based type inference algorithm in the mainstream Dylan
    compiler. As case study we use the Dylan standard library (roughly
    32000 lines of code), which witnesses that the implementation
    generates faster code with fewer errors. Some previously
    undiscovered errors in the Dylan library were revealed.

    https://dl.acm.org/citation.cfm?id=1869643.1869645

**Partial Dispatch: Optimizing Dynamically-Dispatched Multimethod Calls with Compile-Time Types and Runtime Feedback** [`pdf <http://people.csail.mit.edu/jrb/Projects/pd.pdf>`__] [`bib <../_static/documentation/bachrach2000.bib>`__]
    We presented an approach to gaining back complete class hierarchy
    information by delaying the construction of dispatch caches until
    the whole class hierarchy is available at run- time. Run-time
    call-site caches can then be constructed as specialized decision
    trees built from disjointness and concrete- subtype operations on
    actual arguments combined with compile-time inferred types
    injected into the run-time. Unnecessary decision steps can be
    avoided and often run-time dispatch can be completely
    eliminated. We consider this to be a nice half-way house between
    full static compilation and dynamic compilation which mitigates
    the runtime expense of separately compiled components while
    satisfying our implementation constraints of code shareable
    components, multi-threaded runtime, incremental development, “pay
    as you go philosophy”, and interoperability with standard tools.

**D-Expressions: Lisp Power, Dylan Style** [`pdf <http://people.csail.mit.edu/jrb/Projects/dexprs.pdf>`__] [`bib <../_static/documentation/bachrach1999.bib>`__]
    This paper aims to demonstrate that it is possible for a language
    with a rich, conventional syntax to provide Lisp-style macro power
    and simplicity. We describe a macro system and syntax manipulation
    toolkit designed for the Dylan programming language that meets,
    and in some areas exceeds, this standard. The debt to Lisp is
    great, however, since although Dylan has a conventional algebraic
    syntax, the approach taken to describe and represent that syntax
    is distinctly Lisp-like in philosophy.

`See our publications page to find more <publications.html>`_.

For Open Dylan Developers
=========================

.. raw:: html

     <div class="alert alert-block alert-info">
       <p>Notes and materials useful to those working on
       Open Dylan itself or those who have an interest in the low
       level details.</p>
     </div>

`Open Dylan Hacker's Guide <hacker-guide/index.html>`_ [`pdf <hacker-guide/OpenDylanHackersGuide.pdf>`__] [`epub <hacker-guide/OpenDylanHackersGuide.epub>`__]
    A work in progress to help out people who are hacking on Open Dylan itself.

`Dylan Style Guide <style-guide/index.html>`_ [`pdf <style-guide/StyleGuide.pdf>`__] [`epub <style-guide/StyleGuide.epub>`__]
    Notes and thoughts on how to format your Dylan code. This is the style
    guide that we aspire to adhere to in the Open Dylan sources.

`Dylan Enhancement Proposals <../proposals/index.html>`_
    A series of proposals for improvements to the Open Dylan
    implementation and related libraries.

`Open Dylan Release Notes <release-notes/index.html>`_
    Notes on new features and bug fixes in each release of Open Dylan.


Archived Documentation
======================

.. raw:: html

      <div class="alert alert-block alert-warning">
        <p>This is old documentation that we don't plan to
        bring forward. Let us know if there's interest in this
        material.</p>
      </div>

`Developing Component Software with CORBA <http://opendylan.org/documentation/opendylan/corba/index.htm>`_
    A tutorial and reference for CORBA interoperability using the Open Dylan ORB.

`OLE, COM, ActiveX and DBMS library reference <http://opendylan.org/documentation/opendylan/interop2/index.htm>`_
    Describes high and low level interfaces to COM, OLE, and
    ActiveX component technology, and generic DBMS support, through
    SQL with an ODBC backend.

.. raw:: html

      </div>
    </div>
