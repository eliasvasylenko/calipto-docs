Text
====

When discussing text in the context of programming there are a few different views we can take at different levels of abstraction, so let's define some terms.

- *Literal* - A piece of text as it appears in source code.

- *String* - A sequence of abstract characters which defines a piece of text.
  
- *Character* - The indivisible unit from which text is composed.

- *Representation* - The s-expressions which represent strings and characters at runtime. Every string must be encodable in Unicode, and all Unicode text must be representable as a string.

- *Layout* - A scheme for storing a representation in computer memory. A typical implementation may choose to lay out strings according to a standard encoding such as UTF-8 or UTF-32, but this is not a requirement.

String Literals
---------------

Basic string literals in Calipto are delimited by quotes ``"``::

  "For example, this is a string."

Unlike in most languages, literals in Calipto do not support escape sequences. Instead, if a string is to contain quotes, the delimiters must be modified by way of surrounding the string with backslashes ``\``::

  \"Thus any quotes within the string are "disambiguated" from the delimiters."\

And in the same way, if a string is to contain backslashes, the delimiters must be extended with a sequence of backslashes which exceeds the longest sequence within the string. The number of backslashes before the opening delimiter must match the number after the closing delimiter::

  \\\"And now we can put one \ or two \\ backslashes in our string."\\\

In languages which support escape sequences, programmers are forced to visually parse an embedded DSL just to understand what the contents of a string will be at runtime, but in Calipto the contents are always exactly as they appear.

The choice not to support escape sequences or backslashes is justified by the ease with which strings can be concatenated::

  (cat "Hello, string." \n)

  (cat multiplier " times 2.5 is " (#decimal 2.5 multiplier))

We can also use our escape notation to help make our opening and closing delimiters "heavier" in long concatenations, and to disambiguate them visually::

  (cat \"Thingy is at point ["\ x ", " y "]")

It's also possible for string literals to span multiple lines::

  (strip-indent (cat \"
    This is the first line...
    ... and this is the second!
    Here's a value in quotes: ""\ (inline-value) \""!
    And now for the final line."\)))

Characters
----------

If a string is a sequence of characters, we need to define exactly what constitutes a "character" and how this notion is represented at runtime. This turns out be be a tricky choice with a number of subtle tradeoffs, as the word character is heavily overloaded.

In the Unicode standard there are a few candidates for what could be considered a character.

- A *glyph* is a specific written representation of a grapheme, for instance the way a grapheme is rendered according to a choice of font.

- A *grapheme* is what a human reader would typically consider a single character. It may be composed of multiple code points.

- A *code point* is the unit of information of text in the abstract. It is independent of encoding and may be composed of multiple code units.

- A *code unit* is the unit of encoding of text as a series of bytes. It is dependent on the chosen encoding format (i.e. UTF-8, UTF-16, UTF-32, etc.)

In Calipto we choose code points as our unit of representation, and hereafter we will interchangeably refer to code points as characters. The justification for this choice is multi-faceted.

Code units are unsuitable to be the character unit, as they are an artifact of the choice of encoding which is generally considered to be an implementation detail. Glyphs, too, are unsuitable to be the character unit, as they are concerned with the manner of representation of text rather than the abstract text itself.

Graphemes are a more attractive target, but unfortunately they are sometimes defined to be locale-dependent in Unicode, and it is also sometimes useful to manipulate text at the sub-grapheme level.

So for these reasons, code points are left as the best option to be our character, and our unit of representation. This has a few other benefits, for instance it aligns with the Unicode standard, which assigns "Abstract Characters" to single code points. It also simplifies laying characters out in memory in some circumstances, as all code points can fit into the same fixed amount of space.

However, many text manipulation tasks should operate over graphemes and not "characters", and for this reason many of the operations defined over text are parametric with a strategy for "clustering" graphemes.

Graphemes
---------

TODO discuss grapheme clustering.
Normalisation
TODO discuss normalisation & equivalence (Unicode specifies 4 normal forms!)

Normalisation probably shouldn't be done automatically, as this prohibits round-tripping from external formats.

Newlines are just \n on all platforms by default.

Scanning
--------

Characters in strings cannot be directly indexed, they must be reached and counted by iterating over them. This is to make it easier for language implementations to choose compressed storage strategies such as UTF-8 or UTF-16.

Graphemes can also not be directly indexed or counted. TODO explain why.

In other words, most useful operations over strings are defined iteratively. It turns out that iterative operations are more widely applicable than just to static strings, for instance we may also be able to iterate efficiently over streaming network sources, or text generated dynamically. For this reason we abstract all our important string processing operations into a more general text scanning API.
