Dylan Object Model for C and C++ Programmers
============================================

In this appendix, we discuss certain areas where Dylan’s object model
differs significantly from the object model of C and C++.

The concept of pointers
-----------------------

If you are familiar with a language with explicit pointers, such as C,
you may be confused initially by Dylan’s object model. Although there is
no “pointer-to” operation in Dylan, there are pointers in the
implementation. If you are trying to imagine how Dylan objects are
implemented, it is better to think in terms of always manipulating a
pointer to the object: A Dylan variable (or slot) stores a pointer to an
object, rather than a copy of the object’s slots. Similarly, assignment,
argument passing, and identity comparison are in terms of pointers to
objects.

Even characters and numbers can be *considered* as objects that are
pointed to (objects with an unmodifiable value slot), making the object
model uniform. But compilers optimize away the indirection for these
built-in classes.

Note that ``=`` comparison defaults to pointer comparison, but can be
customized by class. There are sensible customizations built-in for
characters, numbers, collections, sequences, and lists. You can add your
own customizations for classes that you create.

Consider this Dylan code:

Dylan object example.

.. code-block:: dylan

    define class <color> (<object>)
      slot red :: <integer> = 0, init-keyword: red:;
      slot green :: <integer> = 0, init-keyword: green:;
      slot blue :: <integer> = 0, init-keyword: blue:;
    end class <color>;

    define constant black = make(<color>);

    define constant white
      = make(<color>, red: 2 ^ 24 - 1, green: 2 ^ 24 - 1, blue: 2 ^ 24 - 1);

    define method whiteness-test(color :: <color>)
      if (color = white) format-out("It’s white!\n") end;
    end method whiteness-test;

    define variable color = black;

    color := white;
    whiteness-test(color);

The equivalent C code is as follows:

C equivalent of Dylan object example.

.. code-block:: c

    typedef struct _color
      { int red, green, blue; }
      Color;

    static Color _black = {0, 0, 0};
    Color* const black = &_black;

    static Color _white = {16777215, 16777215, 16777215};
    Color* const white = &_white;

    void whitenessTest(Color* const color) {
      if (color == white) { printf("It’s white!\n"); }
    }

    void main () {
      Color* color = black;
      color = white;
      whitenessTest(color);
    }

The benefit of the Dylan model is that the final two statements are a
single pointer assignment and a passing of a single pointer as a
parameter. The comparison in ``whitenessTest`` is a single pointer
comparison. Another possible C implementation — one more typical of C
style, but *not* equivalent to the Dylan implementation — is as follows:

C-style example, without pointers.

.. code-block:: c

    typedef struct _color
      { int red, green, blue; }
      Color;

    Color const black = {0, 0, 0};
    Color const white = {16777215, 16777215, 16777215};

    void whitenessTest(Color const color) {
      if (color.red == white.red &&
          color.green == white.green &&
          color.blue == white.blue)
        { printf("It’s white!\n"); }
    }

    void main () {
      Color color = black;
      color = white;
      whitenessTest(color);
    }

In the C-style example, without pointers, the final two statements
consist of three integer assignments (as the *Color* structure is
copied), and a passing of a three-slot structure (the equivalent of
three arguments) as an argument. The comparison in whitenessTest is
three integer comparisons (as the two *Color* structures are compared,
slot by slot).

The drawback of the Dylan object example is shown here:

.. code-block:: dylan

   color.blue := 0;

The preceding call makes *white* yellow! In the C-style example, without
pointers, you would make only *color* yellow. You can prevent people
from changing defined colors to other colors in Dylan by not allowing
the slots of ``<color>`` objects to be modified once they are initialized
— in other words, by making ``<color>`` objects *immutable*:

Dylan object example, with immutable objects.

.. code-block:: dylan

    define class <color> (<object>)
      constant slot red :: <integer> = 0, init-keyword: red:;
      constant slot green :: <integer> = 0, init-keyword: green:;
      constant slot blue :: <integer> = 0, init-keyword: blue:;
    end class <color>;

    define constant black = make(<color>);

    define constant white
      = make(<color>, red: 2 ^ 24 - 1, green: 2 ^ 24 - 1, blue: 2 ^ 24 - 1);

    define variable color = black;

    define method whiteness-test(color :: <color>)
      if (color = white) format-out("It’s white!\n") end;
    end method whiteness-test;

    color := white;
    whiteness-test(color);

You can consider Dylan as always using pointers, even to objects such as
integers and characters. Integers and characters are, by definition,
immutable objects: There are no slots that you can change in an integer
or character object. Thus, there is no danger of setting 6 to 9.
Built-in immutable objects can have their pointers optimized away by the
compiler: The compiler just has to arrange that ``6 = 6`` and ``9 = 9``,
whether there is only one 6 object pointed to by all the variables with
the value 6, or copies of 6 are stored in each of those variables (saving
the need for a pointer).

Another difficulty in the Dylan model is this potentially embarrassing
situation:

.. code-block:: dylan

    color := make(<color>, red: 2 ^ 24 - 1, green: 2 ^ 24 - 1, blue: 2 ^ 24 - 1);
    if (color = white) format-out("It’s white!\n") end;

The preceding expression might not say “It’s white!”, because ``make``
might return a new object with white RGB values, and that object would
not be ``=`` to the object named ``white``. The equivalent C code would be:

.. code-block:: c

    Color* make_color(int r, int g, int b) {
      Color* c = (Color*)malloc(sizeof(Color));
      c->red = r; c->green = g; c->blue = b;
      return c;
    }

    static Color _white = {16777215, 16777215, 16777215};
    Color* const white = &_white;

    Color* color = make_color(16777215, 16777215, 16777215);
    if (color == white) { printf("It’s white!\n"); };

Because the preceding code is comparing the pointer stored in ``white`` to
the pointer stored in ``color``, it will clearly not say “It’s white!”.
The default implementation of ``=`` in Dylan is to compare pointers.

There are several solutions to this difficulty in Dylan. One is to
customize the ``=`` comparison operator for our class to do a comparison
more thorough than the default comparison:

.. code-block:: dylan

    define method \= (o1 :: <color>, o2 :: <color>)
      o1.red = o2.red & o1.green = o2.green & o1.blue = o2.blue;
    end method \=;

Now, using ``=`` will compare colors by checking their individual RGB
components, and our whiteness test will work.

Note that Dylan also provides the ``==`` comparison operator, which always
compares pointers. This comparison is useful when you want to check
object identity. But, as we have seen, it is not always the appropriate
default for comparison of equality of objects. The compiler can avoid
calling our ``=`` method altogether if the same object is compared to
itself. It can do so because, with the exception of IEEE NaNs
(nonnumbers), values that are ``==`` must also be ``=``.

.. index:: make; methods

Another approach that you can use if your objects are immutable is to
make sure that they are unique. The ``make`` function is not required to
return a new object each time, as shown in the Dylan object example,
with unique, immutable objects.

This advanced use of ``make`` and tables ensures that there is always only
one instance of each color. Thus, when we make another white, it will
always be *the* white, and our whiteness test will work with the default
``=`` comparison. The choice of solution depends on whether you will be
doing more making or more comparing.

Dylan object example, with unique, immutable objects.

.. code-block:: dylan

    define class <color-table> (<table>)
    end class <color-table>;

    define method table-protocol(<color-table>)
      local method color-hash(color :: <color>)
        let (red-id, red-state) = object-hash(color.red);
        let (grn-id, grn-state) = object-hash(color.green);
        let (blu-id, blu-state) = object-hash(color.blue);
        let (merge-id, merge-state) =
          merge-hash-codes(red-id, red-state,
                           grn-id, grn-state, ordered: #t);
          merge-hash-codes(merge-id, merge-state,
                           blu-id, blu-state, ordered: #t);
      end;
      local method color-test(o1 :: <color>, o2 :: <color>)
        o1.red = o2.red & o1.green = o2.green & o1.blue = o2.blue;
      end;
      values(color-test, color-hash)
    end method table-protocol;

    define variable color-table = make(<color-table>);

    define method make(class == <color>, #key red, green, blue)
      let prototype = next-method();
      element(color-table, prototype, default: #f) |
        (color-table[prototype] := prototype);
    end method make;

.. _c-comparisons-concept-of-classes:

The concept of classes
----------------------

If you are familiar with the class concepts of C++, you may be confused
by Dylan’s class model. In Dylan, all base classes are effectively
virtual base classes, with “virtual” data members. When a class inherits
another class more than once (because of multiple inheritance), only a
single copy of that base class is included. Each of the
multiple-inheritance paths can contribute to the implementation of the
derived class. The Dylan class model favors this mix-in style of
programming.

Here is an example of such a program, followed by the equivalent C++:

Mix-in example in Dylan.

.. code-block:: dylan

    define class <window> (<object>)
      slot width :: <integer>;
      slot height :: <integer>;
    end class <window>;

    define class <border-window> (<window>)
      slot border-width :: <integer>;
    end class <border-window>;

    define method width(window :: <border-window>)
      next-method() - 2 * window.border-width;
    end method width;

    define method height(window :: <border-window>)
      next-method() - 2 * window.border-width;
    end method height;

    define class <label-window> (<window>)
      slot label-height :: <integer>;
      slot label-text :: <string>;
    end class <label-window>;

    define method height(window :: <label-window>)
      next-method() - window.label-height;
    end method height;

    define class <border-label-window>
      (<border-window>, <label-window>, <window>)
    end class <border-label-window>;

The example is a greatly simplified sketch of a computer-display
windowing system, where a window may have a border (outline decoration),
or a title (such as the title bar of a window), or both. (We omit any
further detail, such as scroll bars.) One chore in such a system is to
compute the available display area of a window from that window’s
overall size and from the sizes of the window’s components.

Note that calling ``height`` on an instance of ``<border-label-window>``
will automatically perform the actions appropriate for a window with a
border and a label. First, the method for ``<border-window>`` will be
called, subtracting out the border width; when it calls ``next-method``,
to get the underlying window width, the method for ``<label-window>`` will
be called, subtracting out the label height; finally, when it calls
``next-method``, the method for getting the value of the ``height``
slot in the underlying window will be called.

This example is a classic one of the mix-in style — the full
functionality of the ``<border-label-window>`` class is the result of the
combination of the individual pieces of ``<border-window>`` and
``<label-window>`` functionality.

C++ equivalent of the mix-in example.

.. code-block:: c++

    class Window {
      private:
      int _width;
      int _height;
      public:
      virtual int width() { return _width; }
      virtual int height() { return _height; }
    };

    class BorderWindow : public virtual Window {
      private:
      int _border_width;
      public:
      virtual int border_width() { return _border_width; }
      virtual int width();
      virtual int height();
    };

    int BorderWindow::width() {
      return Window::width() - 2 * border_width();
    }

    int BorderWindow::height() {
      return Window::height() - 2 * border_width();
    }

    class LabelWindow : public virtual Window {
      private:
      int _label_height;
      char *_label_text;
      public:
      virtual int label_height() { return _label_height; }
      virtual char* label_text() { return _label_text; }
      virtual int height();
    };

    int LabelWindow::height() {
      return Window::height() - label_height();
    }

    class BorderLabelWindow :
      public virtual BorderWindow,
      public virtual LabelWindow,
      public virtual Window {
      public:
      virtual int height();
    };

    // Have to generate "combined" method by hand in C++
    int BorderLabelWindow::height() {
      return Window::height() - 2 * border_width() - label_height();
    }

It may be helpful for C++ programmers to consider that:

- Dylan base classes are always virtual.
- In Dylan, data members are accessed through virtual functions, so it
  is always possible to override access to a data member in a derived
  class, and to modify the returned value (or, by overriding the
  setter, to modify the value to be stored).
- Dylan’s *next-method* allows you to use automatic method combination
  when you are programming in a mix-in style.

Note that the C++ equivalent of the mix-in example is incomplete. It is
intended only as a guide to how you can think of Dylan classes. In
particular, we have not modeled the slot setter virtual functions that
Dylan classes define automatically, and we have not gone into how
instances of the classes are constructed. In Dylan, we would simply give
init-keywords for each of the slots, and the automatically generated
constructor would fill them in for any of the derived classes. In
contrast, constructors for virtual base classes are a particularly
difficult aspect of C++: They make it hard to model what is done in
Dylan accurately. In general, the mix-in style of programming is more
difficult to do in C++, because that language’s support for it is quite
limited.

Note also that the C++ code is provided only as a model of Dylan
execution, so that you can understand the semantics of Dylan classes in
C++ terms. Good Dylan compilers use library compilation, type
inferencing, and partial evaluation to optimize out the overhead
normally associated with virtual classes and virtual functions, while
preserving the dynamic execution semantics.
