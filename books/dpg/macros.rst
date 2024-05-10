Macros
======

.. index:: ! macro

The term *macro*, as used in computer programming, originally stood for
*macro-instruction*, meaning an instruction that represented a
sequence of several machine (or micro) instructions. Over time, the term
has evolved to mean any word or phrase that stands for another phrase
(usually longer, but built of simpler components). Macros can be used
for abbreviation, abstraction, simplification, or structuring. Many
application programs, such as word processors or spreadsheets, offer a
macro language for writing scripts or subroutines that bundle a number
of simpler actions into one command.

Many computer languages support a macro facility for creating shorthand
notations for commonly used, longer phrases. They range from simple,
text-based abbreviations to full languages, permitting computed
replacements. Macros are processed before the program is compiled by
*expanding* each macro into its replacement phrase as that macro is
encountered until there are no more macros. You can use macros to extend
the base language by defining more sophisticated phrases in terms of
simpler, built-in phrases.

The primary use of macros in programming languages is to extend or adapt
the language to allow a more concise or readable solution for a
particular problem domain. A simple program rarely needs macros. More
complicated programs, including the implementation of a Dylan compiler
and run-time system, will use macros often. Macros have no visible
run-time cost or effect — they are transformations that take place
during the compilation of a program (hence, they *can* increase
compilation time). Although macros may take the form of function calls,
they are not functions — they cannot be passed as functional arguments,
and they cannot be invoked in a run-time image as a function can.
Although macros may have parameters, they do not take arguments the way
functions do. The arguments to a macro are not evaluated; they are
simply program phrases that can be substituted in the replacement
phrase.

Dylan provides a macro facility that is based on pattern matching and
template substitution. This facility is more powerful than is a simple
textual substitution facility, but is simpler than a *procedural-macro*
facility, which allows arbitrary computations to construct replacement
phrases. Dylan’s macro facility is closely integrated with the Dylan
language syntax, and permits most macro needs to be satisfied. Dylan
designers have also planned for a full procedural macro capability, so
that it can be added compatibly at a later time if there is sufficient
demand.

.. topic:: Comparison with C and C++:

   C and C++ macros are text substitutions, performed by a preprocessor.
   The preprocessor has no understanding of the language; it simply
   splices together text fragments to create replacement phrases.

   Dylan macros are written in terms of Dylan language elements; the macros
   choose their transformation by pattern matching, and they substitute
   program fragments.

   Language-based macros are more powerful than — and avoid a number of
   common pitfalls of — text-substitution macros. These pitfalls are
   described in later comparisons in this chapter.

.. index::
   single: patterns
   single: macro; patterns
   single: templates
   single: macro; templates
   single: macro; rules

Patterns and templates
----------------------

A Dylan macro consists of a set of *rules*. Each rule has two basic
parts: a *pattern* that is matched against a fragment of code, and a
*template* that is substituted for the matched fragment, perhaps
including pieces of the original fragment. When a macro is invoked, each
rule is tried in order until a matching pattern is found. When a match
is found, the macro is replaced by the matching template. If no match
can be found, an error occurs.

Dylan macros are recognized by the compiler because they fit one of
three possible formats: the function macro, the statement macro, and the
defining macro. The macro format determines the overall fragment that is
matched against the macro’s rules at each macro invocation.

.. index::
   single: macro; function macro
   single: function macro

The simplest macro format that the compiler can match is that of a
function call. A *function macro* is invoked in exactly the same way
that a function is invoked. The name of the macro is a module variable
that can be used anywhere a function call can occur. Typically, it is
simply the name followed by a parenthesized list of arguments, but
recall that slot-style abbreviations and unary and binary operators are
also function calls.

.. index::
   single: macro; delay of evaluation of arguments

The most important use of function macros is to rearrange or delay
evaluation of arguments. The fragment that is matched against the
function macro’s rules is the phrase that represents a function’s
arguments. The function macro can then rearrange the function arguments,
perhaps adding code. When a macro rearranges its arguments, its action
has the effect of delaying the evaluation of the arguments (as opposed
to a function call, where the argument expressions are evaluated and
then passed to the function).

One simple use of delaying evaluation is to write a function-like
construct similar in spirit to C’s ``?:`` operator:

.. code-block:: dylan

    define macro if-else
      { if-else (?test:expression, ?true:expression, ?false:expression) }
        => { if (?test) ?true else ?false end }
    end macro if-else;

We could not write ``if-else`` as a function, because both the true and
false expressions would be evaluated before the function was even
called:

.. code-block:: dylan-console

    ? define variable *x* = 0;

    ? define variable *y* = 0;

    ? *y* := if-else(*y* == 0, *x* := 1, *x* := -1);
    => 1

    ? *y*;
    => 1

    ? *x*;
    => 1

If we had defined ``if-else`` as a function, ``*x*`` would have been ``-1``,
rather than ``1``, because both assignments to ``*x*`` would have been
evaluated, before ``if-else`` was called. When a macro is used, the
assignments are just substituted into the template ``if``, which
evaluates the first clause only when the condition is true.

.. index::
   single: macro; pattern variables
   single: pattern variables

Looking at the macro definition of ``if-else``, we can infer basic ideas
about macros. A macro is introduced by ``define macro``, followed by the
*macro name* — in this case, ``if-else``. The definition of the macro
is a *rule* that has two parts: a *pattern* enclosed in braces, *{}*,
that mimics the fragment that it is to match, and a *replacement*.
Macro parameters, called *pattern variables*, are introduced in the
pattern by ``?``. They match fragments with particular *constraints* — in
this case, ``:expression``. They are delimited by punctuation — in this
case, the open and close parentheses, ``()``, and the comma, ``,``.

The replacement part of the rule, the *expansion*, is indicated by ``=>``
and is defined by a *template*, also enclosed in braces. The template
is in the form of a code fragment, where pattern variables are used to
substitute in the fragments they matched in the pattern. Note that
matching and replacement are language based, so required and optional
whitespace is treated exactly as in Dylan. We have used optional
whitespace to improve the legibility of the macro definitions presented
here.

Most Dylan development environments provide a way to view code after all
macros have been expanded. This view can be helpful in debugging macros
that you write. For example, showing the expanded view of an expression
like

.. code-block:: dylan

    *y* := if-else(*y* == 0, *x* := 1, *x* := -1);

might yield

.. code-block:: dylan

    *y* := if (*y* == 0) *x* := 1 else *x* := -1 end;

The exact format of the expanded view of the macro depends on the
particular development environment. Here, we show the code that comes
from the macro template in *underlined italic*, whereas the fragments
matched by the pattern variables and substituted into the template are
presented in our conventional *code font*. Note that the ``if-else``
macro we have defined is just syntactic sugar — Dylan’s built-in ``if``
statement is perfectly sufficient for the job.

.. index::
   single: macro; delay of evaluation of arguments

Another reason to delay evaluation is to change the value of an argument
— for example, to implement an operator similar in spirit to C’s ``++``
and ``+=`` operators:

.. code-block:: dylan

    define macro inc!
      { inc! (?place:expression, ?by:expression) }
        => { ?place := ?place + ?by; }
      { inc! (?place:expression) }
        => { ?place := ?place + 1; }
    end macro inc!;

This macro might be used as follows:

.. code-block:: dylan-console

    ? define variable *x* = 0;

    ? inc!(*x*, 3);
    => 3

    ? *x*;
    => 3

    ? inc!(*x*);
     4

    ? *x*;
     4

In this macro, it is important to delay the evaluation of the first
argument because we want to be able to assign to the variable or slot it
is stored in, rather than simply to manipulate the value of the variable
or slot.

The ``inc!`` macro demonstrates the use of multiple rules in a macro. They
are tried in order until an appropriate match is found. This allows the
``inc!`` macro to have two forms. The one-argument form increments the
argument by 1. The two-argument form allows the increment amount to be
specified.

.. index::
   single: macro; hygiene

Macro hygiene
-------------

Displaying the code fragments inserted by the macro in *underlined
italics* both helps to show exactly what the macro has done to our code,
and draws attention to an important feature of Dylan macros — they are
hygienic macros. A *hygienic* or *referentially transparent* macro
system is one that prevents accidental collisions of macro variables
with program variables of the same name. Consider the following macro,
which is used to exchange the values of two variables:

.. code-block:: dylan

    define macro swap!
      { swap! (?place1:expression, ?place2:expression) }
        => { let value = ?place1;
             ?place1 := ?place2;
             ?place2 := value
           }
    end macro swap!;

The local variable ``value`` is created by the macro. There is a
possibility that this variable could conflict with another variable in
the surrounding code. Consider what might happen if we were to expand
``swap!(value, x)``:

.. code-block:: dylan

    let value = value;
    value := x;
    x := value

With simple textual substitutions, ``swap!`` would have no effect in this
case. Dylan’s hygienic macros solve this problem by differentiating
between the ``value`` introduced by the macro and any other ``value``
that might appear in the original code.

.. topic:: Comparison with C:

   Because C (and C++) macros are simply text substitutions performed
   by a preprocessor that has no understanding of the C language, they
   are inherently unhygienic. C macro writers reduce this problem by
   choosing unusual or unlikely names for local variables in their
   macros (such as ``_swap_temp_value``), but even this workaround
   can be insufficient in complex macros. Dylan macros in effect
   automatically rename macro variables on each expansion to guarantee
   unique names.

.. index::
   single: macro; evaluation in

Evaluation in macros
--------------------

Dylan’s template macros do no evaluation. In particular, the pattern
variables of a macro are unlike function parameters. They name fragments
of code, rather than naming the result of the evaluation of a fragment
of code.

If we were trying to write an operation like C’s ``||`` (one that would
evaluate expressions and would return the value of the first nonzero
expression without evaluating any subsequent expressions), we could not
write it as a function:

.. code-block:: dylan

    define method or-int (arg1, arg2) if (arg1 ~= 0) arg1 else arg2 end end;

When a function is invoked, all its arguments are evaluated first, which
defeats our purpose. If we model our macro on our function idea,
however, we will not get the ideal result either:

.. code-block:: dylan

    define macro or-int
      { or-int (?arg1:expression, ?arg2:expression) } =>
        { if (?arg1 ~= 0) ?arg1 else ?arg2 end }
    end macro or-int;

The expansion of ``or-int (x := x + 1, y := y - 1)`` is probably not what
we want:

.. code-block:: dylan

    if (x := x + 1 ~= 0) x := x + 1 else y := y - 1 end

We see a common macro error — the expression ``x := x + 1`` will be
evaluated twice when the resulting substitution is evaluated, leaving
``x`` with an incorrect (or at least unexpected) value. There is no magic
technique for avoiding this error — you just have to be careful about
repeating a pattern variable in a template. Most often, if you are
repeating a pattern variable, you should be using a local variable
instead, so that the fragment that the pattern represents is evaluated
only once:

.. code-block:: dylan

    define macro or-int
      { or-int (?arg1:expression, ?arg2:expression) }
        => {
             let arg1 = ?arg1;
             if(arg1 ~= 0) arg1 else ?arg2 end
           }
    end macro or-int;

Another potential pitfall arises if the pattern variables appear in an
order in the template different from the one in which they appear in the
pattern. In this case, unexpected results can occur if a side effect in
one fragment affects the meaning of other fragments. In this case, you
would again want to use local variables to ensure that the fragments
were evaluated in their natural order.

These rules are not hard and fast: The power of macros is due in a large
part to the ability of macros to manipulate code fragments without
evaluating those fragments, but that power must be used judiciously. If
you are designing macros for use by other people, those people may
expect function-like behavior, and may be surprised if there are multiple
or out-of-order evaluations of macro parameters.

.. topic:: Comparison with C:

   Because it is more difficult to introduce local variables in C macros
   than it is in Dylan macros, most C programmers simply adopt the
   discipline of never using an expression with side effects as an
   argument to a macro. The problem of multiple or out-of-order
   evaluations of macro parameters is inherent in all macro
   systems, although some macro systems make it easier to handle.

.. index::
   single: macro; constraints

Constraints
-----------

.. index::
   single: macro; statement macro
   single: statement macro

So far, in our macros, we have seen the constraint *expression* used for
the pattern variables. Except for a few unusual cases, pattern variables
must always have a constraint associated with them. Constraints serve
two purposes: they limit the fragment that the pattern variable will
match, and they define the meaning of the pattern variable when it is
substituted. As an example, consider the following *statement macro*,
which we might find useful for manipulating the decoded parts of
seconds:

.. code-block:: dylan

    define macro with-decoded-seconds
      {
        with-decoded-seconds
          (?max:variable, ?min:variable, ?sec:variable = ?time:expression)
          ?:body
        end
      }
        => {
             let (?max, ?min, ?sec) = decode-total-seconds(?time);
             ?body
           }
    end macro;

The preceding macro might be used as follows:

.. code-block:: dylan

    define method say (time :: <time>)
      with-decoded-seconds(hours, minutes, seconds = time)
        format-out("%d:%s%d",
                   hours, if (minutes < 10) "0" else "" end, minutes);
      end;
    end method say;

A statement macro can appear anywhere that a ``begin`` / ``end;`` block can
appear. A statement macro introduces a new *begin word* — in this case,
``with-decoded-seconds`` — and is matched against a fragment that extends
up to the matching ``end``.

The pattern and the constraints on the pattern variables limit what the
macro will match; they define the syntax of this particular statement.
In the case of ``with-decoded-seconds``, the syntax of this statement
begins with a parenthesized list of

- Three *variable* expressions (that is, ``name :: <type>``, where the
  type is optional)
- The literal token ``=``
- An *expression* (any Dylan expression yielding a value)

After the parenthesized list comes a ``body`` (any sequence of expressions
separated by ``;``, just as would be valid in a ``begin`` / ``end;`` block).
Note the use of the abbreviation ``?:body``, to mean ``?body:body`` (a
pattern variable, ``body``, with the constraint ``body``).

The constraints are similar to type declarations on variables: They
limit the acceptable values of the pattern variables, and they help to
document the interface of the macro. The constraints also serve a second
purpose: Once the compiler has recognized a fragment under a particular
constraint, it will ensure the correct behavior of that fragment when
that fragment is substituted in a template. For example, suppose that we
define a function macro:

.. code-block:: dylan

    define macro times
      { times (?arg1:expression, ?arg2:expression ) } =>
        { ?arg1 * ?arg2 }
    end macro times;

We might use the macro as follows:

.. code-block:: dylan

    times(1 + 3, 2 + 5);

Here is the expanded macro:

.. code-block:: dylan

    1 + 3 * 2 + 5

We can see that, if the macro were a simple text-substitution macro, the
result would be 12, rather than the 28 we were expecting. But because,
in Dylan, the constraint is maintained when a pattern variable is
substituted (that is, the expression that makes up each of the pattern
variables remains a single expression), the result is as though the
macro automatically inserted parentheses, and the expansion were

.. code-block:: dylan

    (1 + 3) * (2 + 5)

Some development environments may display the implicit parentheses of an
expression constraint. Thus, the macro will yield the expected result of
28.

.. topic:: Comparison with C:

   Because C macros are simple textual substitutions, the macro writer
   must be sure to insert parentheses around every macro variable when
   it is substituted, and around the macro expansion itself, to prevent
   the resulting expansion from taking on new meanings.

More complex rules
------------------

.. index::
   single: macro; defining macro
   single: defining macro

The macros shown so far have all been simple: a single pattern
transformed into a single template. To get a flavor of the full power of
the Dylan macro system, consider this *defining macro*:

.. code-block:: dylan

    define macro aircraft-definer
      { define aircraft ?identifier:name (?type:name) ?flights end }
        => { register-aircraft(make("<" ## ?type ## ">", id: ?#"identifier"));
             register-flights(?#"identifier", ?flights) }
    flights:
      { }
        => { }
      { ?flight; ... }
        => { ?flight, ... }
    flight:
      { flight ?id:name, #rest ?options:expression }
        => { make(<flight>, id: ?#"id", ?options) }
    end macro aircraft-definer;

We might use the macro ``define aircraft`` as follows:

.. code-block:: dylan

    define aircraft UA4906H (DC10)
      flight UA11, from: #"BOS", to: #"SFO";
      flight UA12, from: #"SFO", to: #"BOS";
    end aircraft UA4906H;

.. index:
   single: macro; auxiliary rules

This macro shows a number of the more esoteric features of Dylan macros.
First, notice the pattern variable ``?flights``, which has no constraint,
but rather is called out as an *auxiliary rule*. When the compiler
matches this macro, it will try each of the auxiliary rule’s patterns
listed under ``flights:`` for a match. When it finds a match, it will
assign the pattern variable ``?flights`` to the fragment resulting from
the matching pattern’s template substitution. In effect, auxiliary rules
give a way of writing new constraints, combined with the effect of a
subroutine for matching and substitution.

In this particular case, we use the auxiliary rule to map yet another
auxiliary rule, ``flight``, over a sequence of flight descriptions that
look similar to the slot descriptions in a class. The mapping is
signaled by the points of ellipsis (``...``) which means that the rule
should be applied recursively (that is, the current rule is matched
again to the fragment that matches ``...``). Note that ``flights`` must
have a rule to cover the case of there being no flight; that rule also
handles the end of the recursion when the final flight has been matched.

The ``flight`` rule simply converts each flight name and its options into
the appropriate call to ``make``, to create the flight. We could extend
this rule to allow a more natural specification for flight origin,
destination, and time.

We do the work of defining an aircraft by calling the helper functions
``register-aircraft`` and ``register-flights`` (which are not given here),
but the macro takes care of getting the arguments in order. The
substitution ``"<" ## ?type ## ">"`` turns the name ``DC10`` into the name
``<DC10>`` by using *concatenation*, allowing a more concise format for our
definer while maintaining our convention for naming types. The substitution
``?#"identifier"`` turns the name ``UA1306`` into the symbol ``#"UA1306"`` by
using *coercion*; the program can use the symbol ``#"UA1306"`` to look up
an aircraft in the registry by name. The template for ``flights`` collects
all the individual flights into a comma-separated list that is passed to
``register-flights`` as a ``#rest`` argument.

.. index::
   single: macro; hygiene

More hygiene
------------

We shall make one more note about hygiene: In a textual substitution
macro, there is a chance that the global variables that the macro uses
(in this case, the helper function ``register-aircraft``) could be confused
with a surrounding local variable of the same name where the macro is
called. This confusion does not happen in a Dylan macro. The global
variables used in a Dylan macro always denote what they denoted at the
time that the macro was defined, rather than at the time that the macro
is called. It is as though the variables were automatically renamed so
that conflicts will be avoided.

You will also notice this feature if you export a macro from a module.
Only the macro needs to be exported. Its global references still refer
to the proper (module-private) values that they had at the time the
macro was defined, just as occurs when a function exported from a module
calls module-private subroutines.

Occasionally, you will want to circumvent macro hygiene. You may want to
define a macro that creates a variable that *is* visible at the macro
call. Here is a simple statement macro that repeats its body until you
ask it to ``stop!``:

.. code-block:: dylan

    define macro repeat
      { repeat ?:body end }
       => { block (?=stop!)
              local method again() ?body; again() end;
              again();
            end }
    end macro repeat;

The term ``?=stop!`` says that the local variable ``stop!``, which is the
block exit variable, will be visible when the macro is called exactly as
``stop!``; there will be no hygienic renaming. Here is an example that
uses the macro to count to 100:

.. code-block:: dylan

   begin
     let i = 0;
     repeat
       if (i == 100) stop!() end;
       i := i + 1;
     end;
   end;

Note that the ``body`` constraint invokes the Dylan parser to match the
code properly between the ``repeat`` and the corresponding ``end``. It is
not confused by the ``end`` of the ``if`` statement, as a text-based macro
might be. The expanded view of the preceding code might look like this:

.. code-block:: dylan

    begin
      let i = 0;
      block (stop!)
        local method again()
          if (i == 100) stop!() end;
          i := i + 1;
          again()
        end;
        again();
      end;
    end;

Note that we have shown the local variable ``stop!`` introduced by the
macro ``block`` in *code font* rather than in *underline italic*, because
it is visible to the body and is exactly the ``stop!`` called in the ``if``
to stop the repetition. The local variable ``again``, on the other hand,
is not visible to the body code. We could use ``again`` instead of ``i`` as
our repetition count without a problem.

.. topic:: Comparison with C:

   All C macros have the syntax of function calls, making it impossible to
   write language extensions such as ``repeat``.  By using language-based
   constraints, such as the ``body`` constraint used here, Dylan macros can
   match language forms, and thus can create extensions that are consistent
   with the base language.

Note that we would have to document how ``repeat`` works for other users,
or they might be surprised if they tried to use ``stop!`` instead of ``i``
in the example.

.. index::
   single: auxiliary macros
   single: macro; auxiliary macros

Auxiliary macros
----------------

One difficulty with the aircraft macro that we defined in
`More complex rules`_ is this: suppose that we want each
flight object to know the type of equipment used, rather than our having
to look up the type in the aircraft registry. What looks like the
obvious approach does not work:

.. code-block:: dylan

    define macro aircraft-definer
      { define aircraft ?identifier:name (?type:name) ?flights end }
        => { register-aircraft(make("<" ## ?type ## ">", id: ?#"identifier"));
             register-flights(?#"identifier", ?flights) }
    flights:
      { }
        => { }
      { ?flight; ... }
        => { ?flight, ... }
    flight:
      { }
        => { }
      { flight ?id:name, #rest ?options:expression }
        => { make(<flight>, equipment: ?"type", id: ?#"id", ?options) }
    end macro aircraft-definer;

When we are processing the ``flight`` auxiliary rules, we would like to be
able to reference the pattern variable ``?type`` (coercing it to a string)
from the main rules, but it is not *in scope* — it is inaccessible
to the auxiliary rules. We could have ``register-flights`` set the
``equipment`` slot after the flight is created, but we would prefer to
initialize the slot at the time we create the ``<flight>`` object. There
is a workaround, an *auxiliary macro*:

.. code-block:: dylan

    define macro aircraft-definer
      { define aircraft ?identifier:name (?type:name) ?flights:* end }
        => { register-aircraft (make("<" ## ?type ## ">", id: ?#"identifier"));
             define flights (?#"identifier", ?"type")
               ?flights
             end }
    end macro aircraft-definer;

    define macro flights-definer
      { define flights (?craft:name, ?equipment:name) end }
        => { }
      { define flights (?craft:name, ?equipment:name) ?flight ; ?more:* end
      }
        => { register-flights
               (?craft, make(<flight>, equipment: ?equipment, ?flight)) ;
             define flights (?craft, ?equipment) ?more end }
    flight:
      { }
        => { }
      { flight ?id:name, #rest ?options:expression }
        => { id: ?#"id", ?options }
    end macro flights-definer;

Here, we have essentially broken out the work that used to be done by
the auxiliary rule ``flights`` into a separate definition macro. Where
``flights`` used points of ellipsis to walk over each flight, the
definition macro uses a *wildcard* constraint ``?more:*``, explicitly
calling itself again (that is, the macro appears in the substitution,
and will be expanded again), as long as there are more flights to be
processed.

Here is an example use of the ``flights-definer`` macro:

.. code-block:: dylan

    define aircraft UA4906H (DC10)
      flight UA11 from: #"BOS", to: #"SFO";
      flight UA12 from: #"SFO", to: #"BOS";
    end aircraft UA4906H;

Expanding that code would result in the following:

.. code-block:: dylan

  register-aircraft (make(<DC10>, id: #"UA4096H"));
  register-flights (#"UA4096H",
                    make(<flight>, equipment: "DC10",
                         id: #"UA11" from: #"BOS", to: #"SFO");
  register-flights (#"UA4096H",
                    make(<flight>, equipment: "DC10",
                         id: #"UA12" from: #"SFO", to: #"BOS");

(Note that this example is a hypothetical one used to illustrate macro
expansion. The ``define aircraft`` statement cannot be compiled in the
airport example.)

Summary
-------

In this chapter, we introduced macros by explaining their purpose as a
language-extension tool, and by showing a range of Dylan macros. Macros
can be useful when you want to tailor the language to express a
particular problem domain more concisely.

:ref:`pattern-constraints` summarizes how constraints control pattern-variable
matches.

.. index::
   single: macro; constraints
   single: macro; pattern variables
   single: pattern variables

.. _pattern-constraints:

.. table:: Pattern constraints.

   +----------------+-------------------------------------------------------------------------+
   | Constraint     | Matches                                                                 |
   +================+=========================================================================+
   | ``token``      | a lexeme (a Dylan word), including literal strings, symbols, and        |
   |                | numbers and punctuation                                                 |
   +----------------+-------------------------------------------------------------------------+
   | ``name``       | a Dylan identifier, including reserved identifiers, such as ``define``, |
   |                | ``end``, and operators such as ``+``, or ``*``                          |
   +----------------+-------------------------------------------------------------------------+
   | ``variable``   | either ``variable`` or ``variable :: <type>``, useful for macros that   |
   |                | mimic variable binding (automatically drops the ``:: <type>``, as       |
   |                | appropriate on substitution)                                            |
   +----------------+-------------------------------------------------------------------------+
   | ``expression`` | a well-formed Dylan expression –- a constant, such as ``37``; a         |
   |                | variable, such as ``*my-position*``; a function call, such as           |
   |                | ``get-current-time()``; a statement, such as                            |
   |                | ``if (test) 12 else try() end``; or a binary operand series, such as    |
   |                | ``x + y * z``                                                           |
   +----------------+-------------------------------------------------------------------------+
   | ``body``       | a well-formed Dylan body — a sequence of semicolon-separated            |
   |                | constituents, each constituent being either a definition, local         |
   |                | declaration, or expression                                              |
   +----------------+-------------------------------------------------------------------------+
   | ``case-body``  | a Dylan ``case`` statement body                                         |
   +----------------+-------------------------------------------------------------------------+
   | ``*``          | any sequence of Dylan tokens and parsed forms                           |
   +----------------+-------------------------------------------------------------------------+

