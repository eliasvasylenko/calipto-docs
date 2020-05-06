Define
======

The ``define`` macro binds a value to a name in the current lexical context. In the simplest case it is easy to see how this works::

  (define greeting "Hello")

  (print (cat greeting "!"))
  ; Hello!

Here we bind the string value ``"Hello"`` to the symbol ``greeting``, then when we mention the symbol again later it resolves to this value and we are able to print it to the console.

Defines do not have to appear at the root of a source file, they can be nested within functions, or within other defines, and will then only be visible within the lexical scope in which they appear. They are able to capture bindings from the lexical scope in which they appear::

  (define 

.. TODO::
  
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
