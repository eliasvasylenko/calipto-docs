Base Language
=============

At its core, Calipto is a very small purely-functional language, with only two primitive data types and a handful of primitive functions. This doesn't make for a very ergonomic programming experience by itself, but don't let that scare you off! Most users will never program directly in the core language and will instead use a number of standard language extensions. That said, the base language is still designed to be human readable and comprehensible, and understanding this design can be a solid foundation upon which to learn the rest of the language.

Primitives
----------

Data
~~~~

There are only two fundamental types of datum, the symbol and the cons cell. All data is immutable with no exceptions, though mutability can easily be simulated with clever macros, for instance using the effect system.

A symbol is a unique namespace-qualified name. They are typically formatted with the namespace before the name, separated with `:`, though in some circumstances the namespace may be inferred from context in which case the `:` should be omitted. Some examples are `system:true`, `false`, `nil`, `product-domain.com:important-thing`. If two symbols share the same name they are equal, otherwise they are not. The symbol `system:nil` may also be represented as `()`.

A cons cell is a pair of data. Each element may be a symbol or another cons cell. In the Lisp tradition, the left element of the pair is referred to as the car, and the right element is referred to as the cdr. They are typically formatted with the car following the cdr, separated with `.` and surrounded by `(` and `)`. The separating dot must be separated from other characters by whitespace, though whitespace around the brackets is optional. Some examples are `(true . false)`, `(hello . (world . ()))`, `(system:true . system:false)`.

List are typically represented as linked lists. The empty list is represented by `system:nil`, and a list with at least one element is represented by a cons cell with its car being the first element and its cdr being the remainder of the list. Lists have a special formatting convention, whereby a list of elements can be  #########

It may seem like a huge performance problem that there are no built in types for numerics, lists, maps, or other data structures, but this is not the case. Runtimes are required to know how to lay out important data structures efficiently in memory, and will provide intrinsics for functions which operate on them.

Functions
~~~~~~~~~

Continuation-Passing/Direct Style
---------------------------------

Most Calipto programs are written in the direct style, as programmers are generally familiar with. That means that every expression and every function returns a value (even if it's a value of a unit type like void), and expressions can be composed by passing arguments to functions. The base language, however, is restricted to the continuation-passing style (CPS), in which no functions or expressions resolve to a value. Instead of control automatically returning to the caller when a function completes, each function explicitly passes along control by calling another function.


