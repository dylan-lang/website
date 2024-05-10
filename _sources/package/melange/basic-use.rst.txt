Basic Use
=========

Although the ``define interface`` form provides a fairly
rich sublanguage for specifying interfaces, it is often
sufficient to use just the "minimal" form. For example, if
``gc.h`` contained the following code:

.. code-block:: c

    typedef char bool;
    typedef struct obj obj_t;
    typedef char *str;
    extern obj_t alloc(obj_t class, int bytes);
    extern void scavenge(obj_t *addr);
    extern obj_t transport(obj_t obj, int bytes);
    extern void shrink(obj_t obj, int bytes);
    extern void collect_garbage(void);
    extern bool TimeToGC;
    #define ForwardingMarker ((obj_t)(0xDEADBEEF))

then you could import it by creating a file named
``class.intr`` which includes arbitrary Dylan code and the
following:

.. code-block:: dylan

    define interface
       #include "gc.h";
    end interface;

You would then run ``melange class.intr class.dylan``
which would produce a file of Dylan code which contains appropriate
definitions for the classes ``<bool>``, ``<obj>``, ``<obj_t>``, and
``<str>``; the variable ``TimeToGC``; and the functions ``alloc``,
``scavenge``, ``transport``, ``shrink``, and ``collect_garbage``.
(The constant ``ForwardingMarker`` will be excluded because it
is not a simple literal.)

.. code-block:: dylan

    if (TimeToGC() ~= 0)
       collect_garbage();
    end if;

This code fragment points out some of the hazards of
"simple" imports. Melange has no way of knowing that ``bool``
should correspond to Dylan's ``<boolean>`` class, so you are
stuck with a simple integer. Likewise, the system wouldn't be
able to guess that ``char *`` should correspond to the Dylan class
``<c-string>``. We will explain in later sections how "map:"
or "equate:" options may be used to provide this information to
Melange.
