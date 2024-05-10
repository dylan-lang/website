Glossary
========

.. glossary::

    *abstract class*
        A class that cannot have direct instances. To define an abstract class,
        you provide the *abstract* class adjective in the ``define class`` form.
        All superclasses of an abstract class must also be abstract.

    *allocation*
        The allocation of a slot determines where the storage for the slot’s
        value is allocated, and determines which instances share the value of
        the slot. There are four kinds of allocation: instance, class,
        each-subclass, and virtual.

    *ambiguous methods*
        Methods that cannot be ordered as more specific or less specific than
        one another, in the method dispatch.

    *assignment*
        The act of setting the value of an existing variable or slot, or of
        setting an element of a collection. The assignment operator is ``:=``.

    *binding*
        An association between a name and an object. For example, there is a
        binding that associates the name of a constant and the object that is
        the value of the constant. The names of functions, module variables, and
        local variables are also bindings.

    *body*
        A region of program code that delimits the scope of all local variables
        declared inside it. Bodies can be nested. An body is begun implicitly
        with ``define method``, and is ended by the corresponding ``end``. You can
        define a body explicitly by using ``begin`` to start it and ``end`` to
        finish it. A local variable has scope extending from its declaration to
        the end of the smallest body that surrounds it.

    built-in class
        A class provided by Dylan, such as ``<object>``, ``<integer>``, or
        ``<string>``.

    *class*
        A definition of a type of other objects, which are called its instances.
        A class defines the slots of its instances. Dylan provides built-in
        classes, and users can define new classes. When you define a class, you
        specify its name, its direct superclasses, and its slots.

    *class precedence list*
        For a particular class, a list of the class and all its superclasses,
        ordered from most specific (the class itself) to least specific (the
        ``<object>`` class).

    *closure*
        A method that closes over some local variables. The closure can access
        the local variables which existed when the closure was created. The
        ability to dynamically create and return closures that can access
        lexical state is one of the important dynamic aspects of Dylan.

    *collection*
        A kind of container that can hold zero or more objects. Dylan provides
        the usual kinds of collections, including arrays, vectors, strings,
        singly linked lists, queues, hash tables, and so on. In Dylan, a
        collection is an instance of a class. For example, the ``<array>`` class
        represents arrays, and the ``<vector>`` class represents vectors.

    *concrete class*
        A class that can have direct instances. By default, a class is concrete.

    *condition*
        An instance (direct or indirect) of the ``<condition>`` class, that
        represents a problem or unusual situation encountered during program
        execution.

    *constant*
        An unchanging binding whose scope is its module. You define a constant
        explicitly with ``define constant``, and implicitly with ``define class``,
        ``define generic``, ``define macro``, or possibly ``define method``. You
        must initialize the value of a constant, and you cannot assign another
        value to a constant during the execution of a Dylan program.

        (Also called :term:`module constant`.)

    constituent
        A definition, a local declaration, or an expression.

    constructor
        A function that creates an instance. A constructor provides a shorthand
        means for calling ``make``. For example, you can call the constructor
        function ``vector`` to create a vector, and to initialize that vector with
        data.

    contract
        An agreement between a generic function and its methods. The generic
        function defines the terms of the contract, and the methods must obey
        the contract; particularly, the methods’ parameters and value
        declarations must be congruent with the generic function’s parameters
        and value declarations.

    definition
        A declaration of a piece of program structure, such as a library,
        module, class, generic function, or method. A definition usually
        establishes a module variable or constant. Definitions include ``define
        variable``, ``define class``, and ``define method``.

    development environment
        A collection of tools for Dylan programmers that can include an editor
        custom-tailored for Dylan code, a browser, a compiler, a debugger, and a
        listener that enables you to enter expressions and to see their values.
        The features of any development environment are defined by the
        implementation, rather than by Dylan itself.

    direct instance
        An object is a direct instance of class A if the object’s class is class
        A. You can use ``object-class`` to find out the class of which an object
        is a direct instance.

    direct subclass
        A class is the direct subclass of all its direct superclasses. “Direct”
        means there is no class intervening between the class and its subclass
        in the inheritance graph.

    direct superclass
        The direct superclasses of a class appear in the ``define class`` form for
        that class. Direct means that there is no class intervening between the
        class and its superclass in the inheritance graph.

    ``dylan`` library
        A library that contains modules that contain the elements of the core
        Dylan language.

    ``dylan`` module
        A module that contains the elements of the core Dylan language.

    ``dylan-user`` module
        The special bootstrapping module in which you define the modules and
        libraries that make up your program.

    exception
        An unexpected event that occurs during program execution.

    expression
        A piece of code that, when executed, can return (zero or more) values
        and can have side effects. Expressions include (among others) literals,
        references to variables or constants, function calls, and statements
        (such as ``if``, ``while``, and ``case``).

    ``#f``
        The canonical false value. This object is the only object that
        represents false in Dylan.

    general instance
        A member of a class. An object is a general instance of a class if it is
        either a direct or an indirect instance of that class. The term
        *instance* is equivalent to the term *general instance*.

    generic function
        A kind of function. A generic function defines an interface, and
        contains methods that implement that generic function. When a generic
        function is called, it chooses the method to call based on the types of
        its required
        arguments.

    *getter*
        A method that retrieves the current value of a slot in an object. Each
        slot in a class automatically has a getter defined for it. The getter’s
        name is the same as the name of the slot.

    handler
        A function that can potentially resolve an exceptional situation.

    implicit generic function
        A generic function created by Dylan if a method is defined by ``define
        method`` or (for a slot getter or setter) by ``define class`` and if no
        generic function of the same name exists. An implicit generic function
        has the most general parameter and result types that are compatible with
        the method.

    indirect instance
        An object is an indirect instance of class A if the object’s class has
        class A as a superclass.

    infix function
        A function whose calling syntax has the function appearing between the
        arguments. The arithmetic functions ``+``, ``-``, ``*``, ``/``, ``<``, ``>``,
        and so on are infix functions, as is the assignment operator, ``:=``. An
        example of the calling syntax is: ``3 + 2``.

    information hiding
        A principle of minimizing the information that is passed among
        components in a system; it reduces the interdependencies of components.

    inheritance
        The ability to arrange for classes that are logically related to one
        another to share the behaviors and data attributes that they have in
        common. Each class inherits from one or more other classes, called its
        superclasses. If no other class is an appropriate superclass, the class
        inherits from the class ``<object>``.

    init expression
        A technique for initializing slots. An init expression provides an
        expression that yields a default value. Every time that an instance is
        made and the slot needs a default value, this expression is evaluated,
        and its value is used as the default. The slot receives its default
        initial value when no init keyword is defined, or when the caller does
        not supply the init-keyword argument to ``make``.

    init function
        A function of zero arguments that is to be called to return a default
        initial value for the slot. The function is called every time that an
        instance is created if no init keyword is defined, or if the caller does
        not supply the init keyword argument to ``make``. To define an init
        function for a slot, use the ``init-function:`` slot option in the class
        definition.

    init keyword
        A keyword that can be given to ``make`` to provide an initial value for a
        slot. To define an init keyword for a slot, you use the ``init-keyword:``
        or ``required-init-keyword:`` slot option in the class definition.

    init value
        A default initial value for a slot, obtained by evaluating an expression
        once, before the first instance of the class is made. To define an init
        value for a slot, use the ``init-value:`` slot option in the class
        definition.

    initialize
        To provide an initial value for something that you are creating, such as
        a slot or a variable.

    initialize method
        A method for the ``initialize`` generic function. The purpose of
        initialize methods is to initialize an instance before that instance is
        returned by ``make``.

    instance
        A member of a class. An object is an instance of a class if it is either
        a direct or an indirect instance of that class. The term *instance* is
        equivalent to the term *general instance*.

    instantiable class
        A class that can be used as the first argument to ``make``. All concrete
        classes are instantiable. You can make an abstract class be instantiable
        by defining a ``make`` method for the class; the ``make`` method must return
        an instance of a concrete subclass of the abstract class.

    interchange format
        A format that all Dylan implementations accept for publishing and
        exchanging source code by means of files. In this format, each file
        contains a single source record. The file must have a header at the
        front, consisting of pairs of keywords and values. One required keyword
        is ``module:``; its value is the name of the module in which the source
        record of the file resides.

    keyword
        A symbol name followed by a colon, such as ``total-seconds:``.

    keyword argument
        An optional argument to a function consisting of a keyword followed by
        that keyword’s value. You can give keyword arguments in any order.
        Keyword arguments can be useful for functions that take many arguments —
        when you call the function, you do not need to remember the order of the
        arguments. Keyword parameters enable a method to accept optional
        arguments that are keyed to a name. Keyword parameters appear after
        ``#key`` in the parameter list.

    library
        A Dylan library defines a software component, which is a separately
        compilable unit that can be either a stand-alone program or a component
        (library) of a larger program. A library contains modules.

    library-interchange definition (LID) file
        A file that enumerates all the files that make up a library. Most Dylan
        implementations support LID files, but these files are not required to
        by the core language.

    *limited type*
        A type that is a more restricted version of its base type. For example,
        a limited-integer type is based on ``<integer>``, but has a given minimum
        or maximum value. Another example of a limited type is a
        limited-collection type, which is a collection type that specifies the
        type of elements, and/or the size of the collection. Limited types are
        created via ``limited``.

    listener
        A tool that enables you to enter Dylan expressions, executes the
        expressions, and displays any values and output produced by them.

    literal constant
        An object whose contents are known completely at compile time.

    local declaration
        A declaration that establishes a local variable, local method, or local
        condition handler. Local declarations include ``let``, ``local``, and ``let
        handler``.

    local variable
        A binding whose scope extends from its definition to the end of the
        smallest body that surrounds it. You establish and use local variables
        within a body. Once the program exits the body, the local variables are
        no longer defined, and an attempt to access them is an error.

    macro
        A word or phrase that stands for another phrase (usually longer, but
        built of simpler components). Macros can be used for abbreviation,
        abstraction, simplification, or structuring. The primary use of macros
        in programming is to extend or adapt the language to allow a more
        concise or readable solution for a particular problem domain.

    method
        A kind of function that can belong to a generic function. Although
        methods are independent of classes, they operate on instances of
        classes. A method states the kinds of objects that it handles by the
        types of its required arguments.

    *module*
        A unit that contains a portion of the definitions of a library. Each
        module specifies an independent namespace for Dylan constants and
        variables, and controls the visibility of the names within a module from
        outside the module. You can use modules both to do information hiding
        and to prevent name clashes between constants and variables.

    *module constant*
        See :term:`constant`.

    *module variable*
        A binding whose scope is its module. A module variable is much like a
        global variable in other languages. You define a module variable with
        ``define variable``. When you define a module variable, you must
        initialize it (that is, provide an initial value for it). If a module
        variable is not exported from the module that defines it, then it is
        accessible only within the module. If the module variable is exported by
        the module that defines it, and is imported or used by another module,
        then it is accessible within that other module as well.

    multiple inheritance
        Inheritance of a class from more than one direct superclass.

    ``<object>`` class
        The class from which all classes inherit, either directly or indirectly.

    object
        An individual datum. Also called an *instance*.

    *parameter list*
        A list of specifications for the arguments to a function. A parameter
        list can specify required and optional arguments. The optional arguments
        can be keyword arguments, each of which is passed to the function as a
        keyword followed by a value. Each parameter has a name, which is bound
        to the corresponding argument within the function’s body when the
        function is called. Required parameters and a method’s keyword
        parameters can include type constraints. The parameter lists of a
        generic function and all its methods must be congruent.

    *parameter specializer*
        The type of a required parameter of a method.

    *predicate*
        A function that returns true or false. False is always represented as
        ``#f``. True is represented by the canonical true value, ``#t``, and by
        any value other than ``#f``.

    protocol
        The interface definition of a software component. The purpose of
        establishing protocols is to define a uniform interface that clients can
        use, even if the implementation of a component is enhanced or modified.

    recursion
        A technique in which a function calls itself.

    required parameter
        A parameter corresponding to an argument that must be provided in the
        call to the function. Required parameters appear before any rest or
        keyword parameters in a parameter list. Required parameters are ordered,
        and the required arguments must be given in the same order.

    rest parameter
        Parameters that enable a method to accept any number of optional
        arguments. Any arguments provided in the call after the required
        arguments are collected in a sequence, which is the value of the rest
        parameter. A rest parameter, if one exists, appears after ``#rest`` in the
        parameter list.

    restart
        A special condition that represents an opportunity to recover from an
         exception.

    restart handler
        A function used to implement the particular recovery action for a
        restart condition.

    root
        The starting point of Dylan class inheritance — the class ``<object>``,
        from which all Dylan classes inherit, either directly or indirectly.

    *setter*
        A method that stores a value in a slot. By default, each slot in a class
        has a setter defined for it automatically.

    signature
        The parameter list and the values declaration of a function.

    *singleton type*
        A type whose only member is one particular instance. Singleton types are
        created via ``singleton``.

    single inheritance
        Inheritance in a class that has only one direct superclass.

    slot
        A unit of data associated with an instance. A slot is like a structure
        member or a field in other languages. Information about a slot is
        specified in the definition of the instance’s class. The location of
        storage for the slot is determined by the slot’s allocation. A program
        retrieves the value of a slot by calling that slot’s getter generic
        function, and, unless the slot is constant, it sets the value by calling
        the slot’s setter generic function.

    slot option
        An option that specifies a characteristic of a slot, such as the default
        initial value or the init keyword. Slot options appear in the ``define
        class`` form.

    source record
        A unit that organizes a portion of the Dylan source code for a program.
        Different Dylan implementations divide code into source records
        differently, and store the source records differently. For example, an
        implementation might store source records in a database. Many
        implementations store source records in files, and typically each file
        contains one source record.

    subclass
        The subclasses of a class include the class itself, and all classes that
        inherit from the class (all the class’s direct subclasses, and all their
        direct subclasses, and so on).

    subtype
        The subtypes of a type include the type itself, and all types that
        inherit from the type, directly or indirectly.

    superclass
        The superclasses of a class include all that class’s direct
        superclasses, and all their direct superclasses, and so on, all the way
        to the root of class inheritance, which is the ``<object>`` class. You can
        use ``all-superclasses`` to find all the superclasses of a class.

    supertype
        The supertypes of a type include all the types from which the type
        inherits, directly or indirectly.

    symbol
        An instance of the ``<symbol>`` type. Symbols are much like strings. There
        are two reasons to use symbols in certain cases where you might consider
        strings. First, symbol comparison is not case sensitive. Second,
        comparison of two symbols is much faster than is comparison of two
        strings, because symbols are compared by identity, and strings are
        usually compared element by element., There are two equivalent syntaxes
        for referring to symbols: ``north:`` is an example of the keyword syntax,
        whereas ``#"north"`` is an example of the hash syntax.

    ``#t``
        The canonical value of true. Note that any value other than ``#f`` is
        considered a value of true.

    type
        An object that describes the structure and behavior of its members. All
        classes are types, but not all types are classes. You can define new
        nonclass types with ``limited``, ``singleton``, and ``type-union``.

    type constraint
        A type associated with a binding or slot that ensures that the value of
        that binding or slot can hold only objects of that type.

    *union type*
        A type whose members include all the members of one or more base types.
        Union types are created via ``type-union``.

    user-defined class
        A class defined by a Dylan user, and not provided by Dylan itself.

    value declaration
        A list of the values returned by a function, and of the types of the
        values. The name of a return value is used purely for documentation
        purposes. When you provide a value declaration for a function, Dylan
        signals an error if the function tries to return a value of the wrong
        type. The compiler can check receivers of the results of the method for
        correct type, and can usually produce more efficient code. The value
        declarations of a generic function and all that function’s methods must
        be congruent.

    virtual slot
        A slot that does not occupy storage; instead, its value is computed.
        When you define a virtual slot, you need to define a getter method to
        return the value of the virtual slot, and you can optionally define a
        setter method to set the value of the virtual slot.
