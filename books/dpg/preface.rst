Preface
=======

Dylan
-----

Dylan (DYnamic LANguage) is a new programming language invented by
Apple Computer and several partners. Dylan is dynamic, is
object-oriented, and delivers efficient applications.

The Dylan language is defined by *The Dylan Reference Manual*,
written by Andrew Shalit, and published by Addison-Wesley (1996).
That manual is the definitive reference on Dylan. *The Dylan
Reference Manual* is available on the World Wide Web; see
:doc:`environ`, for details.

Dylan is up and running. You can get it from Harlequin, Carnegie
Mellon University, Apple Computer, Digitool, and other organizations.
Dylan implementations run on most of the popular computer platforms.
Full-fledged implementations provide both a compiler and a
development environment. You can obtain public-domain
implementations. See :doc:`environ`.

Audience
--------

This book is written for application programmers who have experience
working in a conventional language, such as C, Pascal, COBOL,
FORTRAN, or BASIC, or in an object-oriented language, such as C++,
Java, Smalltalk, or Common LISP with CLOS. Familiarity with
object-oriented programming and dynamic languages is not required. We
do compare Dylan to C, C++, and Java in this book, but you can read
and understand the book without any knowledge of C, C++, or Java.

Goals of this book
------------------

The primary goals of this book are to teach you how to program in
Dylan, and how to write programs in an object-oriented style. Along
the way, we hope to convince you to use Dylan. It is intended to be
a practical, elegant, and fun language to use. This book is a tutorial on
programming in Dylan, and it does the following:

- Begins with the most basic use of Dylan, and gradually expands to
  show the more powerful and advanced techniques.
- Gives the flavor of working with the Dylan language in a typical
  Dylan environment.
- Shows how to define classes and methods that work together to solve a
  problem.
- Shows how to use many of Dylan’s classes, functions, and features to
  good effect within the context of an example application.
- Introduces the more advanced features of Dylan, including multiple
  inheritance, performance, exceptions, and macros.

This book does not attempt to be as complete as *The Dylan Reference
Manual*, and does not provide the following kinds of material:

- Complete descriptions of all classes and functions provided by Dylan
- Complete descriptions of the detailed mechanisms in Dylan
- To make full use of Dylan, programmers need *The Dylan Reference
  Manual*, as well as this book.

Organization of this book
-------------------------

We have divided the chapters of this book into four parts:

- :doc:`part1`, introduces the object-oriented and dynamic
  nature of Dylan.
- :doc:`part2`, provides more details about Dylan’s
  object-oriented techniques, and covers collections (that is, how to
  use strings, vectors, lists, and other kinds of collections), control
  flow, libraries, and modules.
- :doc:`part3`, contains a complete working application that
  illustrates the topics covered in :doc:`part1` and
  :doc:`part2`.
- :doc:`part4`, covers four areas that are sophisticated and
  powerful: multiple inheritance, performance versus flexibility,
  exceptions, and macros. The chapters in :doc:`part4` show
  how we can improve the example shown in :doc:`part3` by
  applying advanced techniques.

Program examples
----------------

This book includes many program examples. Our approach is to show how
evolutionary programming might work by presenting an example simply
at first, and then expanding it gradually.

In :doc:`part1`, we develop an example of a simple library
that represents time and position. That library is needed for the
sample airport application that we develop in :doc:`part3`.
The airport application simulates airplanes, runways, gates, flights,
and airports. Its goal is to schedule gates for arriving and
departing aircraft. To do scheduling, we need the library that
represents and manipulates time and position.

Harlequin and Addison-Wesley provide World Wide Web pages containing
the source code of the program examples. See :doc:`source-code`.

Dylan’s core language is lean. It does not include input–output
facilities, support for a user interface, or interfaces for
communicating with programs written in other languages. These
features are available in libraries supplied by vendors or in the
public domain. We want this book to be applicable to the widest
possible range of Dylan implementations, so we focus on the core
Dylan language, and use only those library interfaces that are widely
available.

Conventions used in this book
-----------------------------

- We use boldface when we introduce new terms, such as *library*.

- We use bold typewriter font for code examples and names of Dylan
  functions and objects, such as ``define method``. Code comments appear
  in oblique typewriter font — for example,

  .. code-block:: dylan

      // Method that says a greeting
      define method say-greeting (greeting :: <object>);
        format-out("%s\n", greeting);
      end;

- Many Dylan environments provide a *listener*, which enables you to
  type in expressions and to see their return values and output. We use
  a hypothetical Dylan listener to show the result of evaluating Dylan
  expressions:

  .. code-block:: dylan-console

     ? say-greeting("hi, there");
     => hi, there

  In our hypothetical listener, the Dylan prompt is the question mark, ``?``.
  The *bold typewriter font* shows what the user types. The
  *bold-oblique typewriter font* shows what the listener displays.

- We use boxes to give information about Dylan’s naming conventions,
  cautions, performance implications, comparisons to other languages such
  as C or C++, environment notes, and automatic-storage-management notes.
  Here is an example:

.. topic:: Environment note:

   Our hypothetical development environment does not represent any
   particular Dylan development environment. Also note that the Dylan
   language does not require a development environment, so any given
   implementation may not provide one.

An image of Dylan
-----------------

Jonathan Bachrach designed the image on the cover of this book. He
played with the meaning that Dylan has for him by creating colorful
tiles that appear to take off and fly. Each tile has its own vibrant
color, unique personality, and individual strength. The tiles fly
independently, but tend to flock with other tiles to achieve harmony
within a community. Each tile could represent a Dylan component, or a
Dylan programmer. Once Bachrach was satisfied with the still image,
he took the next step, and built an animation of the tiles flying
gracefully through space, flocking together, and creating a dynamic
new world.

Bachrach wrote the animation and physical-modeling portions of the
program in Dylan, using Open GL as the three-dimensional rendering
substrate. Steve Rowley provided the physics equations. Bachrach
demonstrated his animation at the Apple Worldwide Developers
Conference in 1995.

Acknowledgments
---------------

We are fortunate to have at Harlequin a great pool of Dylan talent
and expertise, including original inventors of the language, compiler
gurus, and environment designers. A core group of Dylan experts and
two expert C programmers gave us valuable technical advice and
encouragement from the first to the final days of our project:
Freeland Abbott, Jonathan Bachrach, Kim Barrett, Paul Butcher, Paul
Haahr, Tony Mann, and Keith Playford. Other people reviewed our
drafts along the way: Roman Budzianowski, Bob Cassels, Edward Cessna,
Bill Chiles, Christopher Fry, David Gray, Eliot Miranda, Scott McKay,
Nosa Omorogbe, Mike Plusch, and Andy Sizer. We are grateful to
Harlequin people whose expertise lies in programming languages other
than Dylan, for giving us their perspectives on our book: Judy
Anderson, Wesley Dunnington, David Jones, Andy Latto, Peter Norvig,
Kent Pitman, Steve Rowley, Craig Swanson, Jason Trenouth, Helen
Vickers, and Evan Williams.

Andrew Shires carefully tested all our program examples. Brent
Tennefoss gave us a great deal of help with graphics. Gary Palter
shared his Macintosh expertise, and Leah Bateman shared her Windows
expertise. Richard Brooksby let us steal time from other projects to
write this book. Anne Altherr, Sharon Van Gundy, Clive Harris, and
Sang Lee helped us to navigate the legal and business issues. Ken
Jackson helped us to get the ball rolling, and gave it an extra push
when needed. Jo Marks is one of Dylan’s biggest fans — he urged us to
write this book as a way to explain the power of Dylan to a wider
audience.

We are grateful to Dylan experts outside of Harlequin who gave us
thoughtful and thorough reviews of the book: Scott Fahlman, Robert
Futrelle, David Moon, and Andrew Shalit.

Our editors at Addison-Wesley cheerfully and capably steered us
through the process and helped to shape our book. We are grateful to
Sarah Hallet Corey, Lyn Dupré, Nancy Fenton, and Helen Goldstein.
Eileen Hoff designed the cover using Bachrach’s image. It was, once
again, a great pleasure to work with Peter Gordon.

We thank the people at Apple Computer who combined their vision of
the future with hard work to make Dylan a reality. We thank the
people at Carnegie Mellon University and Harlequin who continue to
move Dylan forward with insight and creativity.

