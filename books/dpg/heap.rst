Definition of a New Collection
==============================

In this chapter, we implement a data structure called a *sorted
sequence*. A sorted sequence is a sequence that automatically keeps the
elements of the sequence in a particular order, based on some value
computed from each element. Elements are added and removed from sorted
sequences; however, the sorted sequence determines the key associated
with the element. Thus, it does not make sense to store an element in a
sorted sequence at a specific key, because the sorted sequence will
determine the correct key to satisfy the automatic-ordering constraint.

We use Dylan’s ``forward-iteration-protocol`` to implement the connection
between our new collection class and Dylan’s standard collection generic
functions. Dylan’s forward-iteration protocol is a well-defined
interface that collection implementors and collection-iterator
implementors can use to enable iterators to operate over new
collections, and to enable collections to work with new iterators. Once
the forward iteration protocol is defined on ``<sorted-sequence>``, many
of the standard Dylan collection generic functions that we covered in
:doc:`collect`, will work with instances of the new sequence.

The airport application uses a sorted sequence to keep track of aircraft
transition in time order. See :doc:`nlanding`, for more details.

The ``sorted-sequence.dylan`` file
----------------------------------

The ``sorted-sequence.dylan`` file contains the module constants, classes,
and methods that build on Dylan’s collection framework to define the
structure and behavior of the new ``<sorted-sequence>`` collection.

.. _heap-new-collection-class:

A new collection class
~~~~~~~~~~~~~~~~~~~~~~

The ``sorted-sequence.dylan`` file.

.. code-block:: dylan

    module: sorted-sequence

    define class <sorted-sequence> (<sequence>)
      // The vector that stores the elements of the sorted sequence, in order
      slot data :: <stretchy-vector> = make(<stretchy-vector>, size: 0);
      // The function used to extract the comparison value from an element
      constant slot value-function :: <function> = identity,
        init-keyword: value-function:;
      // The function used to determine whether one comparison value is
      // smaller than another comparison value
      constant slot comparison-function :: <function> = \<,
        init-keyword: comparison-function:;
    end class <sorted-sequence>;

Because is there is a well-defined ordering of the elements of sorted
sequences, we choose ``<sequence>`` to be the superclass of
``<sorted-sequence>``. We use the built-in collection class called
``<stretchy-vector>`` to store the elements of our sorted sequence,
because we want to be able to have the sorted sequence grow to any size
in a convenient way.

The slots ``comparison-function`` and ``value-function`` are constant slots,
because we intend to have clients specify these functions only when they
create the sorted sequence. If we had decided to let clients change the
value of these slots, we would have made the slots virtual, so that we
could reorder the data vector after either function had changed.

Now that we have covered the structure and initialization of the sorted
sequence data structure, we can define basic collection methods.

.. _heap-basic-collection-methods:

Basic collection methods
~~~~~~~~~~~~~~~~~~~~~~~~

The ``sorted-sequence.dylan`` file. *(continued)*

.. code-block:: dylan

    define method size (sorted-sequence :: <sorted-sequence>)
     => (sorted-sequence-size :: <integer>)
      sorted-sequence.data.size;
    end method size;

    define method shallow-copy (sorted-sequence :: <sorted-sequence>)
     => (copy :: <sorted-sequence>)
      let copy
        = make(<sorted-sequence>,
               value-function: sorted-sequence.value-function,
               comparison-function: sorted-sequence.comparison-function);
      // The map-into function replaces the elements of the copy’s data array
      // to be the identical elements of the data array of sorted sequence
      copy.data.size := sorted-sequence.data.size;
      map-into(copy.data, identity, sorted-sequence.data);
      copy;
    end method shallow-copy;

    define constant $unsupplied = list(#f);

    define method element
        (sorted-sequence :: <sorted-sequence>, key :: <integer>,
         #key default = $unsupplied)
     => (element :: <object>);
      if (key < sorted-sequence.data.size)
        sorted-sequence.data[key];
      elseif (default = $unsupplied)
        error("Attempt to access key %= which is outside of %=.", key,
              sorted-sequence);
      else default;
      end if;
    end method element;

In the preceding code, we define methods for determining the number of
elements in the sorted sequence, for copying the sorted sequence (but
not the elements stored in the sorted sequence), and for accessing a
particular item in the sorted sequence. Once we have defined the
``element`` method for sorted sequences, we can use the subscripting
syntax to access particular items in the sorted sequence. Our ``element``
method implements the standard Dylan protocol, which allows the caller
to specify a default value if the key is not contained within the
collection. If the key is not part of the collection, and no default
value is specified, then an error is signaled. Since we do not export
``$unsupplied`` from our library, we can be certain that no one can supply
that value as the ``default`` keyword parameter for our ``element`` method.

Note that the ``element-setter`` method is not defined, because it does
not make sense to store an element at a particular position within the
sorted sequence. The sorted sequence itself determines the correct key
for each item added to the sorted sequence, based on the item being
added and on the value and comparison functions.

Next, we show methods for adding and removing elements from sorted
sequences.

.. _heap-adding-and-removing-elements:

Adding and removing elements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``sorted-sequence.dylan`` file. *(continued)*

.. code-block:: dylan

    // Add an element to the sorted sequence
    define method add!
        (sorted-sequence :: <sorted-sequence>, new-element :: <object>)
     => (sorted-sequence :: <sorted-sequence>)
      let element-value = sorted-sequence.value-function;
      let compare = sorted-sequence.comparison-function;
      add!(sorted-sequence.data, new-element);
      sorted-sequence.data
        := sort!(sorted-sequence.data,
                 test: method (e1, e2)
                         compare(element-value(e1), element-value(e2))
                       end);
      sorted-sequence;
    end method add!;

    // Remove the item at the top of the sorted sequence
    define method pop (sorted-sequence :: <sorted-sequence>)
     => (top-of-sorted-sequence :: <object>)
      let data-vector = sorted-sequence.data;
      let top-of-sorted-sequence = data-vector[0];
      let sorted-sequence-size = data-vector.size;
      if (empty?(sorted-sequence))
        error("Trying to pop empty sorted-sequence %=.", sorted-sequence);
      else
        // Shuffle up existing data, removing the top element from the
        // sorted sequence
        for (i from 0 below sorted-sequence-size - 1)
          data-vector[i] := data-vector[i + 1];
        end for;
        // Decrease the size of the data vector, and return the top element
        data-vector.size := sorted-sequence-size - 1;
        top-of-sorted-sequence;
      end if;
    end method pop;

    // Remove a particular element from the sorted sequence
    define method remove!
        (sorted-sequence :: <sorted-sequence>, value :: <object>,
         #key test = \==, count = #f)
     => (sorted-sequence :: <sorted-sequence>)
      let data-vector = sorted-sequence.data;
      let sorted-sequence-size = data-vector.size;
      for (deletion-point from 0,
           // If we have reached the end of the sequence, or we have reached
           // the user-specified limit, we are done
           // Note that specifying a bound in the preceding clause for
           // deletion-point does not work, because bounds are computed only
           // once, and we change sorted-sequence-size in the body
           until: (deletion-point >= sorted-sequence-size)
                  | (count & count = 0))
        // Otherwise, if we found a matching element, remove it from the
        // sorted sequence.
        if (test(data-vector[deletion-point], value))
          for (i from deletion-point below sorted-sequence-size - 1)
            data-vector[i] := data-vector[i + 1]
          end for;
          sorted-sequence-size
            := (data-vector.size := sorted-sequence-size - 1);
          if (count) count := count - 1 end;
        end if;
      end for;
      sorted-sequence;
    end method remove!;

The ``remove!`` method uses a form of the ``for`` loop that includes an
``until:`` clause, much like the ``my-copy-sequence`` method defined in
:ref:`collect-lists-and-efficiency`. Note that all termination checks are tested
prior to the execution of the body.

Although the ``pop`` method is not used in the airport application, it is
included for completeness. We could make the ``pop`` method faster by
storing the data elements in reverse order; however, that would lead to
either odd behavior or odd implementation of the ``element`` function on
sorted sequences.

The forward-iteration protocol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Dylan’s forward-iteration protocol allows us to connect the usual
collection iteration functions to our new collection class. Connecting
to the forward-iteration protocol is as simple as defining an
appropriate method for the ``forward-iteration-protocol`` generic
function. This method must return two objects and six functions.

The ``sorted-sequence.dylan`` file. *(continued)*

.. code-block:: dylan

    // This method enables many standard and user-defined collection operations
    define method forward-iteration-protocol
        (sorted-sequence :: <sorted-sequence>)
     => (initial-state :: <integer>, limit :: <integer>,
         next-state :: <function>, finished-state? :: <function>,
         current-key :: <function>, current-element :: <function>,
         current-element-setter :: <function>, copy-state :: <function>)
      values(
        // Initial state
        0,

        // Limit
        sorted-sequence.size,

        // Next state
        method (collection :: <sorted-sequence>, state :: <integer>)
          state + 1
        end,

        // Finished state?
        method (collection :: <sorted-sequence>, state :: <integer>,
                limit :: <integer>)
          state = limit;
        end,

        // Current key
        method (collection :: <sorted-sequence>, state :: <integer>)
          state
        end,

        // Current element
        element,

        // Current element setter
        method (value :: <object>, collection :: <sorted-sequence>,
                state :: <integer>)
          error("Setting an element of a sorted sequence is not allowed.");
        end,

        // Copy state
        identity);
    end method forward-iteration-protocol;

If we are to iterate over any collection, we must maintain some state to
help the iterator remember the current point of iteration. For the
forward-iteration protocol, we maintain this state using any object
suitable for a given collection. In this case, an integer is sufficient
to maintain where we are in the iteration process. The first object
returned by ``forward-iteration-protocol`` is a state object that is
suitable for the start of an iteration. The second object returned is a
state object that represents the ending state of the iteration. Since,
in this case, the state object is just the current key of the sorted
sequence, the integer 0 is the correct initial state, and the integer
that represents the size of the collection is the correct ending state.

The third value returned is a function that takes the collection and the
current iteration state, and returns a state that is the next step in
the iteration. In this case, we can determine the next state simply by
adding 1 to the current state.

The fourth value returned is a function that receives the collection,
the current state, and the ending state, and that determines whether the
iteration is complete. In this case, we need only to check whether the
current state is equal to the ending state.

The fifth value returned is a function that generates the current key
into the collection, given a collection and a state. In this case, the
key is the state object.

The sixth value returned is a function that receives a collection and a
state, and returns the current element of the collection. In this case,
the ``element`` function is the obvious choice, since our state is just
the key.

The seventh value returned is a function that receives a new value, a
collection, and a state, and changes the current element to be the new
value. In this case, such an operation is illegal, since the only
rational way to add elements to sorted sequences is with ``add!``.
Because this operation is illegal, an error is signaled.

The eighth and final value returned is a function that receives a
collection and a state, and returns a copy of the state. In this case,
we just return the state, because it is an integer and thus has no slots
that are modified during the iteration process. If we represented the
state with an object that had one or more slots that did change during
iteration, we would have to make a new state instance and to copy the
significant information from the old state instance to the new state
instance.

Once we have defined a ``forward-iteration-protocol`` method for sorted
sequences, we can iterate over them using ``for`` loops, mapping
functions, and other collections iterators described in :doc:`collect`.
Also, if someone defines a new iterator that uses the forward-iteration
protocol, then this new iterator will work with sorted sequences.

Dylan has several other related protocols for backward iteration and for
tables. See the *The Dylan Reference Manual* for details.

The ``sorted-sequence-library.dylan`` file
------------------------------------------

The definitions for the sorted sequence library and module are simple.
The only module variable that we need to export is for the sorted
sequence class itself. All the generic functions that we want clients to
use on sorted sequences are exported by the ``dylan`` module.

The ``sorted-sequence-library.dylan`` file.

.. code-block:: dylan

    module: dylan-user

    define library sorted-sequence
      export sorted-sequence;
      use dylan;
      use definitions;
    end library sorted-sequence;

    define module sorted-sequence
      export <sorted-sequence>;
      use dylan;
      use definitions;
    end module sorted-sequence;

The ``definitions`` library and module are defined in :doc:`nlanding`.

The ``sorted-sequence.lid`` file
--------------------------------

The LID file for sorted sequences is also straightforward. The entire
library is contained within two files (in addition to the LID file
itself). The library and module definitions are in the file
``sorted-sequence-library.dylan``. The definitions of module constants,
classes, and methods are in the implementation file,
``sorted-sequence.dylan``.

The ``sorted-sequence.lid`` file.

.. code-block:: dylan

    library: sorted-sequence
    files:   sorted-sequence-library
             sorted-sequence

Summary
-------

In this chapter, we covered the following:

- We explored how to define our own collection class.
- We showed how to integrate that class into Dylan’s collection
  framework.
- We used several variations of the control structures presented in
  :doc:`collect`.
