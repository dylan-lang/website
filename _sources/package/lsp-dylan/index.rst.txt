.. highlight:: shell

*********************
Dylan Language Server
*********************

This is an implementation of the `Language Server Protocol
<https://microsoft.github.io/language-server-protocol/>`_ for Dylan.

.. toctree::
   :hidden:

Current Status
==============

As of 2024-04-19, the server implements

* Jump to declaration
* Jump to definition
* Diagnostics (i.e., compiler warnings)
* Hover (i.e., argument lists)

When applied to a symbol which is bound to a generic function, "jump to
definition" will show a list containing the generic function and its specific
methods, whereas "jump to declaration" will jump straight to the generic
function.

Installation
============

1. First install `Open Dylan <https://opendylan.org>`_ 2023.1 or newer.

2. Clone the `lsp-dylan repository <https://github.com/dylan-lang/lsp-dylan>`_::

     $ git clone --recursive https://github.com/dylan-lang/lsp-dylan

3. Update the workspace so that dependencies are installed and the registry is
   created::

     $ cd lsp-dylan
     $ dylan update

4. Build the ``dylan-lsp-server`` binary.  This will install the binary in
   ``${DYLAN}/bin``. If :envvar:`DYLAN` is not defined it defaults to
   ``${HOME}/dylan``.
   ::

     $ make install

   or just run these commands::

     $ dylan build --unify dylan-lsp-server
     $ cp _build/sbin/dylan-lsp-server /usr/local/bin/

5. Make sure ``dylan-lsp-server`` is on your :envvar:`PATH`.


Usage
=====

The LSP server needs to be able to open a project (that is, a Dylan library)
associated with the file you're editing when you turn on LSP in your editor. It
assumes you are using a `dylan-tool
<https://github.com/dylan-lang/dylan-tool>`_ workspace and searches for a
project to open as follows:

1. If there is a :file:`workspace.json` file and that file has a
   `"default-library"` property, the specified library is opened.

2. It uses the :program:`dylan` tool to choose a library defined in the
   workspace. This is `generally
   <https://github.com/dylan-lang/dylan-tool/blob/292b7bf761745c9fa810511c9888f802dd787011/sources/workspaces/workspaces.dylan#L151>`_
   the test suite library, if one exists; otherwise it chooses a project
   arbitrarily.

Normally you shouldn't need to set any environment variables; everything is
derived from the full pathname to the :program:`dylan-compiler` executable,
which must be on your :envvar:`PATH`.

When you open each new file in your editor the LSP client may try to start a
new project if the file isn't part of the same :program:`dylan` workspace
directory. If you want the client to use just one project, use a `multi-package
workspace <https://opendylan.org/package/dylan-tool/index.html#workspaces>`_.

.. note:: Always run ``dylan update`` and ``dylan build -a`` in your workspace
          **before** starting the LSP server, or :program:`dylan-lsp-server`
          may not be able to open your project. (This requirement will be
          removed in a future release.)


Emacs Usage
-----------

1. Make sure the :program:`dylan-lsp-server` executable is on your
   :envvar:`PATH`, or customize the ``lsp-dylan-exe-pathname`` elisp variable. See
   below for more on customization.

2. Install `lsp-mode <https://github.com/emacs-lsp/lsp-mode>`_ and `dylan-mode
   <https://github.com/dylan-lang/dylan-emacs-support>`_. Both of these are
   available from MELPA.

3. When you jump to another :file:`.dylan` file, that file does not
   automatically have LSP enabled so you must use ``M-x lsp`` again. To make
   this automatic, add this to your emacs init file:

   .. code-block:: elisp

      (add-hook 'dylan-mode-hook 'lsp)

4. Start emacs and make sure that :file:`lsp-dylan.el` is loaded. For example::

     emacs --load=/path/to/lsp-dylan/lsp-dylan.el

   You will probably want to modify your Emacs init file to load the file with

   .. code-block:: elisp

      (load "/path/to/lsp-dylan/lsp-dylan.el")

5. Open a Dylan source file and type ``M-x lsp`` to start the client (unless
   you added the hook above, in which case it started automatically).

The client starts the LSP server (the `dylan-lsp-server` executable) and
connects to it.  You should see a message telling you what Dylan project was
opened.

The emacs client has a customization group "lsp-dylan" which is a member of the
"Language Server (lsp-mode)" group, and has the following variables:

* ``lsp-dylan-exe-pathname``
* ``lsp-dylan-extra-command-line-options``
* ``lsp-dylan-log-pathname``
* ``lsp-dylan-open-dylan-release``

These are documented in the customization interface within emacs. Use ``M-x
customize-group`` ``lsp-dylan`` to customize these variables.

.. note:: Emacs lsp-mode saves state in :file:`~/.emacs.d/.lsp-session-v1`. If
          you have a problem with Dylan's LSP support it's a good idea to
          delete this file and try again.


Visual Studio Code Usage
------------------------

These instructions were tested on Linux and macOS.

1.  Install Visual Studio Code and ``npm``.

2.  The VS Code extension is in the folder ``vscode-dylan``. It is necessary to
    run ``npm install`` in this folder before starting the extension for the
    first time, and any time a ``git pull`` updates the dependencies.

3.  Open the ``vscode-dylan`` folder in VS Code.

4.  In the debug viewlet, click the green play arrow (Launch Extension) or
    press ``F5``.

5.  A build process will begin in 'watch mode'; whenever the source is changed,
    the extension will be rebuilt. It is possible to debug the VS Code
    extension in this window, set breakpoints, watch variables and so on.

6.  A new VS Code window will open with the extension running.

7.  Open a folder with a Dylan project in it.

8.  If :program:`dylan-lsp-server` is on the system path, it will be
    found. Otherwise, open the Settings *in the new extension window*, find the
    Dylan section under Extensions, and edit the path to the LSP server. The
    full, absolute pathname to the executable needs to be specified. It is
    usually better to set this in the 'User' scope, otherwise it will only
    apply to that particular project.

9.  It should now be possible to use the extension window to edit Dylan code
    using LSP.

10. If the VS Code extension is changed, it is necessary to restart the
    extension host, or just close and re-open the extension window.


LSP Server Development
======================

This section contains notes for people interested in helping to improve LSP
support for Dylan.

Open Dylan
----------

Unless you're an Open Dylan expert already, you'll probably find that you want
to add debug statements in the Open Dylan IDE code to figure out what's going
on....

1. Turn on the ``--debug-opendylan`` by customizing
   ``lsp-dylan-extra-command-line-options``. This causes extra in the
   ``*lsp-dylan::stderr*`` buffer from calls to the ``debug-out`` macro.

2. Create a multi-package workspace with both the ``lsp-dylan`` repo and the
   ``opendylan`` repo::

     $ mkdir lsp; cd lsp
     $ git clone --recursive https://github.com/dylan-lang/lsp-dylan
     $ git clone --recursive https://github.com/dylan-lang/opendylan
     $ echo '{ "default-library": "dylan-lsp-server" }' > workspace.json

3. Sprinkle calls like ``debug-out(#"lsp", "your message here", ...)`` around
   in the Open Dylan code as needed.

4. Remember to delete :file:`~/.emacs.d/.lsp-session-v1`` as needed. I (cgay)
   usually start a new emacs after rebuilding :program:``dylan-lsp-server``,
   like this::

     $ rm -f ~/.emacs.d/.lsp-session-v1; emacs &

   If you don't start a new emacs, you probably want to at least kill the old
   :program:`dylan-lsp-server` process. I (cgay) usually rebuild like this::

     $ pkill -f bin/dylan-lsp-server; make install


References
==========

* `Intro to LSP from Microsoft
  <https://docs.microsoft.com/en-us/visualstudio/extensibility/language-server-protocol>`_ -
  Besides being a quick introduction, this has links to some other tools that
  would help in developing VS Code integration for Dylan.

* `LSP v3.15 Specification
  <https://microsoft.github.io/language-server-protocol/specifications/specification-3-15/>`_ -
  This is the version we are currently coding to.

* `langserver.org <https://langserver.org/>`_ lists LSP implementations that
  support at least one of the six major LSP features.


Index and Search
================

* :ref:`genindex`
* :ref:`search`
