Introduction
============

.. warning:: This documentation is a work in progress and not a faithful representation of implemented features; parts of it may be incomplete, outdated, or aspirational in nature.

Calipto is a language-oriented programming language.

At its core it is a tiny functional language which acts as a sort of human-readable IR. By itself this language is flexible, but it is not particularly ergonomic; the magic lies in its *metaprogramming* facilities.

Compiler
--------

The Calipto compiler is essentially nothing more than a simple macro expander, but from this carefully-designed interface we can bootstrap almost any kind of compiler and runtime that can be imagined; from document generators for markup languages, to parser generators for grammar specifications, to fully fledged programming language runtimes. And all without sacrificing performance!

Types
-----

The host language, and by extension all client languages, are furnished with a general-purpose type system based on static analysis and type-recovery. This means that types can be *expressed as runtime checks*, but that those checks are *statically enforced* by the compiler, and erased so that they place no runtime burden. We believe that this is the best of both words, static safety with a single unified programming model.

Out of the box Calipto provides static analysis for advanced features such as dependent and higher-order types, and since they are implemented via macros any language which is bootstrapped from Calipto can supplement or "turn off" these checks wherever they like, or even to substitute their own type systems via entirely different mechanisms.

Effects
-------

An effect system, to realise the power of purely-functional programming within an imperative programming model.

Concurrency
-----------

A deterministic model which can be locally reasoned about; inter-coroutine communication is mediated by a single thread in the form of an effect handler.

Performance
-----------

A reference implementation supporting both native image generation and an interpreted mode with a state-of-the-art JIT compiler, via `GraalVM & Truffle <https://github.com/oracle/graal>`_.
