Base Language
=============

At its core, Calipto is a very small purely-functional language, with only two primitive data types and a handful of primitive functions. This doesn't make for a very ergonomic programming experience by itself, but don't let that scare you off! Most users will never program directly in the core language and will instead use a number of standard language extensions built on the macro system. That said, the base language is still designed to be human readable and comprehensible, and understanding this design can be a solid foundation upon which to learn the rest of the language.

Continuation-Passing/Direct Style
---------------------------------

Most Calipto programs are written in the direct style, as programmers are generally familiar with. That means that every expression and every function returns a value (even if it's a value of a unit type like void), and expressions can be composed by passing arguments to functions. The base language, however, is restricted to the continuation-passing style (CPS), in which no functions or expressions resolve to a value. Instead of control automatically returning to the caller when a function completes, each function explicitly passes along control by calling another function.

Primitive Data
--------------

There are only two fundamental primitive types of datum, the symbol and the cons cell. All data is immutable with no exceptions, though constrained forms of mutability can easily be simulated with macros, for instance using the effect system.

It may seem like a performance concern that there are no built in types for numerics, lists, maps, or other data structures, but this is not the case. Runtimes are required to know how to lay out important data structures efficiently in memory, and should provide intrinsics for functions which operate on them.

A symbol is a namespace-qualified name. They are typically formatted with the namespace before the name, separated with ``:``, though in some circumstances the namespace may be inferred from context in which case the ``:`` should be omitted. Here are some examples of formatted symbols:

- ``bool:true``
- ``false``
- ``nil``
- ``product-domain.com:important-thing``

If two symbols share the same name and namespace then they are equal, otherwise they are not. The symbol ``data:nil`` may also be formatted as ``()``.

A cons cell is an ordered pair of data. Each half may be a symbol or another cons cell. In the Lisp tradition, the left half of the pair is referred to as the car, and the right half is referred to as the cdr. A cons cell is formatted with the car followed by the cdr, separated with ``.``, and surrounded by brackets.

- ``(true . false)``
- ``(hello . (world . ()))``
- ``(bool:true . bool:false)``

If the cdr holds another cons cell, this can be formatted according to a shorthand notation by omitting the dot and the brackets around the cdr. If the cdr is omitted entirely, it is taken to be ``data:nil``. Here are some examples:

- ``(first second third)`` is equivalent to ``(first . (second . (third . ())))``
- ``(single-element)`` is equivalent to ``(single-element . ())``

This reflects how lists are typically represented in Calipto; the empty list is represented by the ``nil`` symbol, and any other list is represented with a cons cell whose car holds the first element and whose cdr holds the remainder of the list. If a list has a symbol other than ``nil`` in the terminal position, it is considered to be "improper", and in fact any symbol other than ``nil`` may be thought of as an improper empty list:

- ``(first second)`` is equivalent to ``(first second . nil)`` is equivalent to ``(first . (second . ()))``, and is a proper list with two elements.
- ``(first second . terminal)`` is equivalent to ``(first . (second . terminal))``, and is an improper list with two elements.

Primitive Functions
-------------------

Primitive functions are those which cannot be implemented on top of other functions and so must be provided by the platform. Some of them are required to be provided by all platforms, some are optional modulo underlying system capabilities and permissions.
:
- ``(data:cons car cdr k)`` Create a cons cell. The value of ``car`` is "consed onto" the value of ``cdr``, and the resulting cons cell is passed to the function ``k``.
- ``(data:des cons k f)`` Extract the data from each half of a cons cell. If the value of ``cons`` is a cons cell, its ``car`` and ``cdr`` are passed to the function ``k``, otherwise the function ``f`` is called with no arguments.
- ``(data:eq d1 d2 t f)`` Make an equality test. Calls ``t`` if ``d1`` and ``d2`` are equal, otherwise calls ``f``.
- ``(system:exit)`` This function terminates the program.

- ``(random:choose-nondet a b)`` Make a non-deterministic choice. One of the two functions ``a`` and ``b`` is chosen at the discretion of the compiler or runtime---with no requirements regarding probability---and then called with no arguments. This may seem like a perculiar inclusion in the set of primitives, but careful application of this function allows a platform with special knowledge of it to make certain optimisations and use certain data-structures that appear non-deterministic, without violating the surface semantics of the language.
- ``(random:choose a b)`` Make a (possibly-pseudo)random choice. One of the two functions ``a`` and ``b`` is chosen at random and then called with no arguments.

Functional Side Effects
~~~~~~~~~~~~~~~~~~~~~~~

IO is inherently stateful and side effecting, so how do we model that with an API which behaves functionally? This is a problem many programming languages have wrestled with in different ways. Some isolate side effects with monads, while some enforce single-use of side-effecting functions with linear types.

But there is already way to achieve functional side effects in CPS which is perfectly natural, and it does not require any complex static type checking or new programming abstractions. Every time a side-effecting builtin function is called, it remembers the arguments it received and its result, then when it is called again it can give the same results for the same arguments. When the function returns, it passes a new version of itself to the continuation, which can be used to perform another side effect against the resulting state.

When we have an input function, such as e.g. ``system:in``, the definition is annotated with its index. When we call it it performs the IO and remembers it what was read, then all subsequent calls to the same function have the same result. But then how do we read the next character? Well this is CPS, so when you call the function it passes a new version of itself at the next index to the continuation. This approach can be generalised to many different kinds of input. It automatically provides a kind of buffering, which may be useful or may be a detriment to performance.

When we have an output function, such as e.g. ``system:out``, the definition is annotated with its index. When we call it it performs the IO and remembers what was written, then all subsequent calls to the same function will have the same result if the same arguments are given, otherwise they will fail. When called, a new version of the function at the next index is passed to the continuation.

What if clients try to manually materialise a version of one of these functions? If they do so for a state which hasn't been reached yet the system won't know what answers to give them. And if they do so for an old state which has been discarded and cleaned up the system will have forgotten what answers it gave last time.

If we are only permitted to reference symbols from a given namespace if they're explicitly exported from the owning module then we can prevent such attempts to circumvent safety. This means that functional IO functions must be represented by a single symbol and their behaviour must be special-cased by the interpreter or runtime.

.. note::

  At a higher level we will introduce a comprehensive system of side effect handlers and performers in order to properly partition pure and impure code. But this system is implemented at the library level and so our base language needs to operate on some other model.

.. todo::

  Do we want to try to enforce safe resource management with the design of these primitives? This typically means that resources must be closed when they leave scope, but that would seem to be difficult to enforce when there is no clearly defined stack by which to determine scope. Perhaps such restrictions should be the domain of language extensions. Can we use the type system to make sure all "exits" from the section in which a resource is acquired proceed via the associated release function? Do we need a way to represent the stack in the type system in order to achieve this? Or linear types?

.. seealso::

  https://pdfs.semanticscholar.org/9543/279e307892681034afcdf9df863bee90eb42.pdf
  https://core.ac.uk/download/pdf/82009715.pdf
  https://www.cs.bham.ac.uk/~hxt/research/LinCP.pdf

Special Forms
-------------

Special forms are not functions and don't behave as such. This means that they do not follow the continuation-passing style; instead they behave like expressions, evaluating directly to their result. This makes them easy to distinguish from functions based on where they appear in code.

- ``(data:quote a)`` Evaluates to the datum ``a``, without attempting to evaluate it.
- ``(data:lambda p b)`` Evaluates to a function accepting arguments for the parameter list ``p`` and evaluating the function call expressed in the body ``b``. A function can also be defined by quotataion, but the lambda expression has the additional purpose of introducing a lexical scope over the parameter list, and of capturing the lexical scope in which it appears.

.. note::

  If a lambda special form cannot be evaluated this is a syntax error. This does not require any control flow management for the failure path as the error can always be detected statically and so it should never be encountered during evaluation.

.. note::

  Readers familiar with the quasiquote macro---which is taken from Lisp---might ask why the quote special form doesn't allow unquoting. After all, the lambda special form already performs a similar trick when it captures local scope. The difference is that quasiquote is not a fundamental primitive; it can be implemented via ``data:quote`` and ``data:cons``. The set of special forms is intended to be minimal and should not be burdened with unecessary complexity.

.. todo::

  Look into options for PRNGs, TRNGs, CSPRNGs. Rather than a global generator, do we want to create handles to generators like resources, that way we can create the same sequence by deliberately reusing the same seed, which is sometimes useful. How much can this be self-hosted? A PRNG can just be implemented in the language, it's perhaps only the choice of seed which is important to be externalised to a primitive.

.. seealso::

  https://ericlippert.com/2019/01/31/fixing-random-part-1/

- ``(system:in k)`` Read from std in
- ``(system:out d k)`` Write to std out
- ``(system:err d k)`` Write to std error

- ``(file:open path k)`` The function ``k`` accepts functions which operate on the file, including one which closes it. ``(open-temp-file k)``
- ``(system:env name k)`` Get an environment variable
- ``(system:args k)`` Get the program arguments

