.. _melange-concrete-example:

A Concrete Example
==================

In order to get a feel for using Melange, it is probably
best to start with a concrete example.  This section contains a
complete program which will use native C libraries to list the
contents of some directories. For now, you should simply skim
this example to get a general overview of Melange's
capabilities. These will be described in more detail in later
sections.

We will first begin with an "interface file" which
contains a mixture of basic Dylan code and ``define interface``
forms which will be processed by Melange. We will name this file
``dirent.intr``.

.. code-block:: dylan

    module: Junk
    synopsis: A poor imitation of "ls"

    define library junk
      use dylan;
      use streams;
    end library junk;

    define module junk
      use dylan;
      use extensions;
      use extern;
      use streams;
      use standard-io;
    end module junk;

    define interface
      // This clause is more complex than it needs to be, but it does
      // demonstrate a lot of Melange's features.
      #include "/usr/include/sys/dirent.h",
        equate: {"char /* Any C declaration is legal */ *" => <c-string>},
        map: {"char *" => <byte-string>},
        // The two functions require callbacks, which we don't support.
        exclude: {"scandir", "alphasort", "struct _dirdesc"},
        seal-functions: open,
        read-only: #t,
        name-mapper: minimal-name-mapping;
      function "opendir", map-argument: {#x1 => <string>};
      function "telldir" => tell, map-result: <integer>;
      struct "struct dirent",
        prefix: "dt-",
        exclude: {"d_namlen", "d_reclen"};
    end interface;

    define method main (program, #rest args)
      for (arg in args)
        let dir = opendir(arg);
        for (entry = readdir(dir) then readdir(dir),
             until entry = $null-pointer)
          write-line(entry.dt-d-name, *standard-output*);
        end for;
        closedir(dir);
      end for;
    end method main;

We will then process this file through Melange to produce
a file of pure Dylan code.  Melange can be invoked like
this::

    melange dirent.intr dirent.dylan

This command will process ``dirent.intr`` and write a file
named ``dirent.dylan``.

You can compile ``dirent.dylan`` normally within a Dylan library.
