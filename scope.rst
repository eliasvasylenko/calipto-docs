Scope
=====

Calipto core is lexically scoped. Names are bound to values by way of function application.

Macros in Calipto are hygienic, but it is important to understand how scoping rules interact with macro expansion.

Variable Names
--------------

Variable names in Calipto are not just symbols, they are *lists of symbols*. This allows them to be qualified with enough information to uniquly identify them. The scanner generally adds the necessary qualifiers to raw symbols transparently. This is how we maintain hygiene in Calipto.

.. todo::
  Perhaps qualified names must be atoms with a special namespace, say ``qualified-name``. This way we have a single place where they are created and we can intern them (and generate their hashes ahead of time?) as an optimisation, which means they're no slower to use in practice than normal symbols.

Macros are free to try to create their own qualified names by decorating the reader, but it will be an error if they're not unique within the source file. For instance a default lambda macro may choose to qualify parameters with the function name, source position, and source file. The module system may choose to qualify imports by content hash to guarantee uniqueness.

Way to parse qualified names? ``calipto.org:builtin:stdin`` ``|file:///home/user/script.cal|:(line 1000):x``.

Perhaps we can leave out parts we don't need?

.. todo::
  In the runtime the extra info embedded in the variable name doesn't too-badly affect size or speed as variables can be interned and generally treated as opaque pointers. But what if we want to compile and output IR? A dumb printer will duplicate the full qualified names of everything at every single use-site, which could be a huge increase in footprint. Especially with e.g. giant content hashes sprinkled everywhere!

Define
------

The ``define`` macro binds a value to a name . In the simplest case it is easy to see how this works::

  (define greeting "Hello")

  (print (cat greeting "!"))
  ; Hello!

Here we bind the string value ``"Hello"`` to the symbol ``greeting``, then when we mention the symbol again later it resolves to this value and we are able to print it to the console.

Defines do not have to appear at the root of a source file, they can be nested within functions, or within other defines, and will then only be visible within the lexical scope in which they appear. They are able to capture bindings from the lexical scope in which they appear::

  (define 

.. todo::
  Or should they be static and not permit capture?

The a defined value is not fully resolved until it is used, and is permitted to mention unbound symbols. But these symbols must all be defined before the value is used. This allows for mutual recursion::

  (define even? (lambda (x)
    (eq x 0
      true
      (not (odd? (dec x))))))

  (define odd? (lambda (x)
    (not (even? (dec x)))))

  (even? 10)
  ; true

In cases of recursion, an extra parameter must be injected into each participating function for every function which it recurs into, including itself.
