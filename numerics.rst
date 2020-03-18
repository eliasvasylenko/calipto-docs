Numerics
========

There many different ways to represent numbers, and they all have their own tradeoffs. Calipto tries to be flexible and provide defaults appropriate for modern computing.

When discussing numbers there are a few different views we can take at different levels of abstraction, so let's define some terms.

- *Literal* - A number as it appears in source code. A literal is written in positional notation, and is directly loaded as a numeral, before being converted to a direct representation.

- *Numeral* - An s-expression which encodes a number of a given base according to positional notation. A numeral can be converted to an exact representation in the form of an integer or a fraction. A numeral can be trivially printed to or scanned from text in a human-readable format.

- *Representation* - An s-expression which encodes a number in a form which can be evaluated and understood by the language runtime. A representation can be exact or inexact. A representation is typically a combination of binary values with purely symbolic operations and constants.

- *Layout* - A scheme for storing a representation in computer memory. The language is designed to facilitate efficient layout, but how it is actually done is entirely implementation dependent. For instance, users can generally expect small integers to be flattened into typical 32 bit or 64 bit representations as appropriate. Implementations should document their exact behaviour.

Exact Representation
--------------------

Inexactness in computation numerics is a huge source of problems. It can be extremely difficult to reason about overflow, imprecision, and inaccuracy when using common integer and floating point representations. And worse, most programming languages hide these problems from the programmer, allowing errors to accrue silently and catastrophically failing in surprising ways when exceeding the limits of the underlying representation.

Some languages attempt to address these problems by using unbounded integers by default, and by representing rationals fractionally, but this still leaves a huge amount of ground uncovered. What about trigonometric operations? Non-integer exponents? Or well-known irrationals like 'pi'?

Expressions
~~~~~~~~~~~

Unlike most other modern languages, any mathematical expression can be represented exactly in Calipto. Operations or constants with irrational values are represented symbolically, and an expression with an irrational value is not fully evaluated but instead simplified. Any expression which is already maximally simplified is therefore self-evaluating.

Here are some examples of expressions which resolve to an exact form.

- (# 5/7)                      ; '(/ (+ 1 0 1) (+ 1 1 1))

- (# 10 pi)                    ; '(* (+ 1 0 1 0) pi)

- (# 1/2 + 2/3 + 3/4)          ; '(/ (+ 1 0 1 1 1) (+ 1 1 0 0))

- (# 2 e cos(0.2 3 tau))       ; '(* (+ 1 0) e (cos (* (/ (+ 1 1) (+ 1 0 1)) tau)))

Exact rational numbers are all expressed in binary as integers or irreducible fractions. The representation never contains leading zeros. These numbers are considered expressions, and are self-evaluating.

Literals
~~~~~~~~

All numeric literals evaluate to exact rational numbers. The literal forms have the following properties:
They are expressed in positional notation.
A radix `.` can optionally be included.
A base specifier can optionally be included by prefixing with `[base]\`, where `[base]` is the number of the base.
The default base is usually 10, and the maximum base is 36 (each decimal digit, followed by each letter of the alphabet).

The following are examples of valid literals, annotated with the expressions they evaluate to.
- 0         ; 'zero
- 0.1       ; '(/ (+ 1) (+ 1 0 1 0))
- 2\\0.1    ; '(/ (+ 1) (+ 1 0))
- 12.5      ; '(/ (+ 1 1 0 0 1) (+ 1 0))
- 16\\FA.DE ; '(/ (+ 1 1 1 1 1 0 1 0 1 1 0 1) (+ 1 0 0 0 0)) 

The parser will only interpret literals as numbers if the whole literal, including the base specifier, begins with a decimal digit. This is not usually a problem, but if the default base is changed numbers can begin with non-decimal-digit characters, in which case they should be prefixed with `0`.

Inexact Representation
----------------------

Though exact representations are useful, they aren't always appropriate. Irrational numbers cannot simplify to a single number in a portable format, and time and space requirements can quickly compound beyond reasonable boundaries, especially for iterative computation. For these reasons, we need to be able to evaluate expressions to inexact representations.

Inexact representations are only evaluated when explicitly requested. This way the programmer never silently and opaquely loses precision, they are always in direct control of how and when rounding occurs.

Rounding
~~~~~~~~

Numbers can be rounded to a given number of significant figures or places, in any base. This doesn't require that the representation reflect the base in question, and in fact all numbers are still stored as binary fractions when this operation is performed.

Conversion
~~~~~~~~~~
