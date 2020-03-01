Base Language
=============

At its core, Calipto is a very small purely-functional language, with only two primitive data types and a handful of primitive functions. This doesn't make for a very ergonomic programming experience by itself, but don't let that scare you off! Most users will never program directly in the core language and will instead use a number of standard language extensions built on the macro system. That said, the base language is still designed to be human readable and comprehensible, and understanding this design can be a solid foundation upon which to learn the rest of the language.

Continuation-Passing/Direct Style
---------------------------------

Most Calipto programs are written in the direct style, as programmers are generally familiar with. That means that every expression and every function returns a value (even if it's a value of a unit type like void), and expressions can be composed by passing arguments to functions. The base language, however, is restricted to the continuation-passing style (CPS), in which no functions or expressions resolve to a value. Instead of control automatically returning to the caller when a function completes, each function explicitly passes along control by calling another function.

Primitives
----------

Data
~~~~

There are only two fundamental types of datum, the symbol and the cons cell. All data is immutable with no exceptions, though constrained forms of mutability can easily be simulated with macros, for instance using the effect system.

It may seem like a performance concern that there are no built in types for numerics, lists, maps, or other data structures, but this is not the case. Runtimes are required to know how to lay out important data structures efficiently in memory, and should provide intrinsics for functions which operate on them.

A symbol is a namespace-qualified name. They are typically formatted with the namespace before the name, separated with `:`, though in some circumstances the namespace may be inferred from context in which case the `:` should be omitted. Here are some examples of formatted symbols:
- `system:true`
- `false`
- `nil`
- `product-domain.com:important-thing`
  
If two symbols share the same name and namespace then they are equal, otherwise they are not. The symbol `system:nil` may also be formatted as `()`.

A cons cell is an ordered pair of data. Each half may be a symbol or another cons cell. In the Lisp tradition, the left half of the pair is referred to as the car, and the right half is referred to as the cdr. A cons cell is formatted with the car followed by the cdr, separated with `.`, and surrounded by brackets.
- `(true . false)`
- `(hello . (world . ()))`
- `(system:true . system:false)`

If the cdr holds another cons cell, this can be formatted according to a shorthand notation by omitting the dot and the brackets around the cdr. If the cdr is omitted entirely, it is taken to be `system:nil`. Here are some examples:
- `(first second third)` is equivalent to `(first . (second . (third . ())))` 
- `(single-element)` is equivalent to `(single-element . ())`

This reflects how lists are typically represented in Calipto; the empty list is represented by the `nil` symbol, and any other list is represented with a cons cell whose car holds the first element and whose cdr holds the remainder of the list. If a list has a symbol other than `nil` in the terminal position, it is considered to be "improper", and in fact any symbol other than `nil` may be thought of as an improper empty list:
- `(first second)` is equivalent to `(first second . nil)` is equivalent to `(first . (second . ()))`, and is a proper list with two elements.
- `(first second . terminal)` is equivalent to `(first . (second . terminal))`, and is an improper list with two elements.

Functions
~~~~~~~~~

(cons car cdr cont)
(car cons cont)
(cdr cons cont)

