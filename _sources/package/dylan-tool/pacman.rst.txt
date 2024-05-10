.. highlight:: json

******
pacman
******

Pacman is the Dylan package manager library. It knows how to find packages in
the `catalog`_, install them, and how to resolve dependencies between them.

This documentation describes the package model and how versioned dependencies
are resolved. Users generally manage workspaces and packages via `the dylan
command`_.


Packages
========

A package is blob of data with an associated version which can be downloaded
from the network and unpacked into a directory of files. All packages must have
a ``dylan-package.json`` file in their top-level directory to specify
dependencies and other package metadata.

.. note:: In this beta version of pacman packages must be git repositories,
   downloadable with the ``git clone`` command. In the future pacman will
   support downloading and installing arbitrary compressed file bundles so that
   it isn't tied to a specific VCS.


.. _package-versions:

Package Versions
----------------

New versions of a package are released from time to time and each release has a
`Semantic Version`_ associated with it. When specifying a package dependency we
identify a particular release we want to depend on with the syntax
``"<package>@<version>"``.

For example, ``"abc@1.2.3"`` identifies the release of package ``abc`` having
`Semantic Version`_ ``1.2.3`` (major version 1, minor version 2, and patch
version 3).

.. note:: Pacman doesn't support pre-release and build identifiers yet. For
   example, in "abc@1.2.3-alpha1+build1". Support will be added in the future.

How the package name and version are used to locate the package depends on the
"package transport". Git is currently the only transport, and for any given
semantic version ``1.2.3`` there must be a corresponding Git tag ``v1.2.3`` in
the package's Git repository. Ensure that you creat such a tag when publishing
a numbered release of your package. (This can be done easily via the GitHub
UI, or by using the Git command.)

It is also possible to use other Git refs when specifying a dependency:

=================   ==============================
Spec                Meaning
=================   ==============================
``abc@<semver>``    Use the version specified by ``<semver>``. For example
                    "abc@1.2".  See `Dependency Resolution`_ for details of
                    how competing dependencies are resolved.
``abc``             Same as ``abc@latest``.
``abc@latest``      Use the latest numbered, non-pre-release version.
``abc@<ref>``       Use the branch/tag/ref specified by ``<ref>`` instead of a
                    semantic version.
=================   ==============================

When a package is published to the `pacman catalog`_, its dependencies must be
specified with `Semantic Versions`_ so that user builds will be
reproducible. ``abc@latest`` and ``abc@<ref>`` are prohibited in the catalog
and are primarily intended for use during development.


.. _dylan-package.json:

The Package File - dylan-package.json
-------------------------------------

Packages are described by a ``dylan-package.json`` file in the package's
top-level directory. This file contains the name, description, dependencies,
and other metadata for the package. Let's look at the ``dylan-package.json``
file for ``dylan-tool`` itself::

    {
        "name": "dylan-tool",
        "version": "0.6.0",
        "category": "language-tools",
        "contact": "dylan-lang@googlegroups.com",
        "description": "Manage Dylan workspaces, packages, and registries",
        "keywords": ["workspace", "package"],
        "dependencies": [
            "command-line-parser@3.1.1",
            "json@1.0",
            "logging@2.1",
            "regular-expressions@1.0",
            "uncommon-dylan@0.2"
        ],
        "dev-dependencies": [
            "testworks@2.0"
        ],
        "url": "https://github.com/dylan-lang/dylan-tool"
    }

Required Package Attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~~

name
  The package name. This name may differ from the containing directory and/or
  from the package repository URL, although it's usually less confusing if
  they're the same.

description
  A brief description of the package intended to be displayed to users who are
  searching for the packages they need. In some contexts (for example the
  :ref:`dylan-list` command) this may be truncated to only display the first
  sentence, or even less, so special care should be used when writing this part
  of the description.

url
  URL of the Git repository for the package.

version
  A string designating the `Semantic Version`_ of the package.

Optional Package Attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~~

category
  *Reserved for future use.*

contact
  A string giving users a way to contact you about the package. Usually an
  email address or an issue tracker URL.

dependencies
  A list of package dependencies. Each dependency is a string identifying a
  specific release of another package. These dependencies are transitive; any
  package that depends on your package necessarily has your dependencies as
  well as the ones they list explicitly in *their* package file. See the
  example above and see `Dependency Resolution`_ for how conflicts are handled.

dev-dependencies
  A list of package dependencies that are only needed for development purposes,
  such as testing. These dependencies are not propagated to other packages that
  depend on this package. Put another way, these dependencies are not
  transitive.  See `Dependency Resolution`_.

keywords
  A list of strings with additional keywords that might be useful to help users
  find the package if they don't already occur in the "description" attribute.

license
  A string indicating the license under which the package's software is
  released.

license-url
  A URL with the location of the license text. (This attribute is planned to be
  merged with "license", which will become a map with various subattributes.)

Dependency Resolution
=====================

When `the dylan command`_ is asked to update a workspace it asks ``pacman`` to
resolve the dependencies specified in the ``dylan-package.json`` file and to
install the resolved versions of those packages. So how does ``pacman`` do the
package resolution, especially if two packages depend on different versions of
a third package?

The long answer is that ``pacman`` uses `minimal version selection`_ (MVS). To
read more than you ever wanted to know about this subject unless you're Russ
Cox, check out https://research.swtch.com/vgo. In particular, check out the
`principles`_ post in that series, for motivation. What follows is a very brief
summary of minimal version selection and certain aspects that are specific to
``pacman``.

Unlike most traditional package systems, in which when you specify version 1.2
you are really saying "give me the *latest* version that is at least 1.2", with
MVS you are saying "give me the *lowest* version that is at least 1.2". Why
would you want this?  Isn't it a feature to get the latest *compatible*
software when you build?  Well, in fact, a much better feature is to get a
*repeatable build* each time. That is what MVS provides.

If the latest versions are preferred, then building your code today may very
well result in a different binary, with different bugs, than when you build
your code tomorrow.

Example
-------

Let's say you build an application that depends on (and you have tested with)
``strings@2.5`` and ``http@1.3``, and that ``http@1.3`` itself depends on
``strings@2.4.2``.  Further, let's assume that there are three patch versions
of ``strings@2.5``: ``strings@2.5.0``, ``strings@2.5.1``, and
``strings@2.5.2``. Which version of ``strings`` should ``pacman`` install?

The answer is ``strings@2.5.0`` because that is the minimum version that is
compatible with *both* ``strings@2.5`` (which is the same as ``strings@2.5.0``)
and ``strings@2.4.2`` based on `SemVer 2.0`_ rules.

What if ``http@1.3`` instead depended on ``strings@3.0.1``? In this case
``pacman`` would signal an error because ``strings@2.5`` is not compatible with
``strings@3.0.1`` since they have different major versions.

You could say that MVS uses the maximum (compatible) specified minimum version.

Dev Dependencies
----------------

In addition to the primary set of dependencies for each package there may be a
set of "dev dependencies" to pull in software that is used only during
development.  The canonical example of a dev dependency is the test framework
library `testworks`_, which itself depends on several other packages.

When resolving dependencies for a package, dev dependencies may or may not be
considered, depending on context. When updating a development workspace they
are resolved along with the primary dependencies.

.. note:: Currently, updating a workspace is the *only* context, so in practice
   dev dependencies are always considered. When/if we integrate pacman into the
   Dylan build process it will be necessary to have both a dev build and a
   production build. The prod build will exclude dev dependencies.

So how do dev dependencies interact with the main dependencies? If there is a
package that is depended on by both a main and a dev dependency then the main
dependency is always preferred, even if it wouldn't normally be chosen based on
`minimal version selection`_ rules. The reason for this is simple: we want to
use the same software when developing as would be used when running in
production; otherwise, *we're testing the wrong software*.

    Example:

    Most Dylan libraries have a dev dependency on `testworks`_. Testworks
    itself depends on `strings`_. Let's say our main library transitively
    depends on ``strings@1.0`` and `testworks`_ depends on ``strings@1.1``.  In
    this case ``strings@1.0`` is used even though it's older than what
    Testworks requests, because Testworks is only a dev dependency.

Note that dev dependencies are never transitive. That is, if package ``A``
depends on package ``B`` and package ``B`` has a dev dependency on ``C`` this
does not mean that ``A`` depends on ``C``. (``A`` may depend on ``C`` via some
other path, but not via ``B``'s dev dependency.)


Index and Search
================

* :ref:`genindex`
* :ref:`search`

.. _minimal version selection: https://research.swtch.com/vgo-mvs
.. _principles:        https://research.swtch.com/vgo-principles
.. _the dylan command: https://opendylan.org/documentation/dylan-tool/
.. _Semantic version:  https://semver.org/spec/v2.0.0.html
.. _Semantic versions: https://semver.org/spec/v2.0.0.html
.. _SemVer 2.0:        https://semver.org/spec/v2.0.0.html
.. _catalog:           https://github.com/dylan-lang/pacman-catalog.git
.. _pacman catalog:    https://github.com/dylan-lang/pacman-catalog.git
.. _strings:           https://github.com/dylan-lang/strings.git
.. _testworks:         https://github.com/dylan-lang/testworks.git
