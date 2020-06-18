Modules
=======


A good module system should address a number of interconnected concerns;

- Encapsulation (atomics, exports),

- Organization (composition via namespacing, separation of concerns & compilation),

- Composition (namespacing),

- Dependency Management & Repeatability (content-addressed imports).

Architecture
------------

The Calipto module system is implemented entirely in normal Calipto code. It does not require any special features or priviliages from the runtime.

However it does apply additional restrictions on code within the modules which it hosts.

Qualified Symbols
-----------------

Module names can just be symbols. Probably also a good idea to (optionally?) namespace by organisation / user so as to avoid collisions.

Same goes for any other kind of name.

if symbols are normally unqualified, ``lambda``, ``system``, etc. then how can we protect some qualified names as atoms? Inevitably we must EITHER protect some single unqualified symbol from being used by users, OR make cons partial! These both seem like an undesirable restrictions.

So it seems we do need qualified symbols ... but these don't appear to be sufficient for our purposes! We want to be able to add extra qualifying context info to a symbol so that e.g. two unrelated and distinct instances of a symbol within a module can be disambiguated.

Maybe symbols can be qualified any number of times? There can be operators to cons and des symbols, but they don't have to be total and they only accept names.

We can manually qualify one symbol with another, producing a new symbol. This function may be partial over some inputs so as to protect authorship of data to trusted sources::

  (qualify-symbol 'a 'b)
  > a:b

  (qualify-symbol 'a 'b:c)
  > a:b:c

  (qualify-symbol 'a:b 'c)
  > a:b:c

  (qualify-symbol 'system 'in)
  > Failed to qualify symbol, insufficient authority.

We can introspect the name and qualifier of a symbol. These functions are total over all symbols::

  (symbol-name 'a:b:c)
  "c"

  (symbol-qualifier 'a:b:c)
  > a:b

  (symbol-name (symbol-qualifier 'a:b:c))
  "b"

An unqualified symbol is implicitly qualified with ``nil``. The ``nil`` symbol is itself qualified with ``nil``, recursively::

  (symbol-qualifier 'nil:a)
  > nil

  (symbol-qualifier 'a)
  > nil

  (eq 'a 'nil:nil:nil:a)
  > true

Alternatively::

  (symbol-qualifier 'a:b:c)
  > a

  (symbol-qualified 'a:b:c)
  > b:c

  (symbol-qualified 'a)
  > a

  (symbol-qualifier 'a)
  > nil

Content Addressing
------------------

Exports and imports are content-addressed by hash.

Symbol Visibility
-----------------

The module system injects a restricted reader implementation into hosted modules such that they are only able to read symbols which are visible to them via their imports.

Atomics
-------

When we share data from one module to be consumed by others, sometimes we want to protect parts of that data from being read or modified. The way to achieve this in Calipto is with atomics.

An atom is for most purposes an *indivisible particle*. In Calipto, "indivisible" means "unable to be destructed", and instead of particles of course we're dealing with data. The obvious example of an atom is a symbol, but the module system also allows a module to export more complex compositions of data as atoms, such that they *appear* indivisible to other modules.

There are two sides to atomicity; consider that ``des`` and ``cons`` should be symmetric, such that for everything we construct via ``cons`` we should also be able to destruct it via ``des``. We have already defined an ``atom`` to be a datum which cannot be destructed outside of its owning module, and so in turn this must imply that an atom cannot be constructed outside of its owning module.

There are two ways we could protect a datum from construction; by modifying ``cons`` to be a partial function, or by restricting access to one of the components. Calipto takes the latter option, so as to preserve the desirable property that ``cons`` is total::

  (module my-atoms)

  (define a (form-atom (protons neutrons)))

  (print a)
  > (atom my-atoms (protons neutrons))

  (des a)
  > error, attempt to destruct atom

  (cons atom (protons neutrons))
  > error, unable to read symbol 'atom'

  (print (split-atom a))
  > (protons neutrons)

To achieve this, every file which is read as a module is injected with a modified version of ``des``, along with the modified version of ``read-symbol`` which protects lookup of certain symbols.

Every atom is self-evaluating?

.. todo::

  Every atomic cons cell must be identified by a "marker symbol" in the car position. Should this marker symbol be the same every time (module:atom?). This makes things easier for two reasons;

  - Faster to check whether something is an atom (and we can des) against only a single "blessed" symbol.

  - There is only one symbol which we must protect from being created or exported and it is owned by the module system.

  It would be nice if we could designate any symbol to be an atomic marker, but this would slow down des checking (special casing to put an "atomic" bit on the data could alleviate this, but its nice to have performance *without* special-casing where possible. It should however be possible to statically eliminate this check in many places so perhaps that is sufficient). 
  
  This would also mean that the module system must ensure that the marker symbol is not exported from a module, and also that it can never escape by being returned from functions, etc. etc. This could be hard, but there are probably tricks to get there by modifying cons etc. to detect changes.

  Alternatively there is a possible middle ground, to say that any symbol in the ``atom`` namespace is an atomic marker and also inaccessible, e.g. ``atom:int``, ``atom:builtin``, etc. But do symbols even *have* namespaces or is a qualified name just another abstraction composed into an atom?

Leaking Protected Data via Equality/Hash
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Atomics may allow information to be leaked via ``eq`` and ``hash`` by default. For instance consider a user atom with a protected password::

  (define (make-user name password)
    `(user (name ,name) (password ,password)))

When calling the function from another module, the internal data will be hidden, as we would expect::

  (let jane (users:make-user "Alex Kid" "hunter2"))
  alex
  > (users:user ...)

However, if we pass this datum to untrusted modules they can still brute-force the password of the user if they know the name::

  (for-each password password-list
    (if (eq (users:make-user "Alex Kid" password) jane)
      password
      nil))
  > "hunter2"

To prevent this, the user datum should include an identity field::

  (define (make-user name password)
    `(user (generate-uid) (name ,name) (password ,password)))

Alternatively atoms don't provide any data hiding at all, and sensitive data such as passwords should be hidden in data structures internal to the module, i.e. a table mapping users to passwords.

Previous Notes
--------------

.. todo::
 
  Organise this document!

To protect structured data is more difficult. There are two possible approaches.

For files/URLs the symbol name of the function could be an actual URL. Perhaps we could even reuse the protocol as the namespace since the formatting is similar, e.g. ``file:///data.txt\0`` is a symbol with the namespace ``file`` and the name ``///data.txt\0`` and is formatted as a file URL appended with a dash and the operation count. This way operations for protocols are automatically properly namespaced. But collisions between protocols and unrelated modules seem likely.

``uri:|https://calipto.org|``

``uri:|https://calipto.org|``

``uri:|connection https://calipto.org 0|``

``uri:|in https://calipto.org 0 0|``

``uri:|out https://calipto.org 0 0|``

``uri:scheme``

Perhaps there can be some mechanism by which to export structures AS symbols, where a symbol must map uniquely to a structure and vice versa. This would be a way to hide access to a structure without perverting the data model.

This could be done by embedding an s-expression in the symbol name, but that seems a little awkward as it has to deal with escapes.

``uri:|((uri:scheme https) (uri:authority (uri:host "calipto.org")) (uri:path "/docs"))|``

``(uri:uri (uri:scheme https) (uri:authority (uri:host "calipto.org")) (uri:path "/docs"))``

Though more generally using the scanner/reader/macro system to parse the string would allow other syntax.

``uri:|https://calipto.org/docs|``

``(uri:uri (uri:scheme https) (uri:authority (uri:host "calipto.org")) (uri:path "/docs"))``

.. todo::

  The main problem with this approach appears to be with the reverse process of printing the string. Do we *need* to be able to do this? If our only use-case is pretty printing output then perhaps we can relax our standards of normalisation and uniqueness of string representations.

.. todo::

  Another problem is that it might be possible for an external module to ``cons`` together a protected form and then be unable to ``des`` it. These operations should always be symmetric, and we need to think of a way to preserve this property!

This is total encapsulation and means that no information can escape unless there is API on the module for it. Typechecking the internals of an atom is still possible if there are functions in the defining module to facilitate the checks, though it can't be done via destructuring on the client side. This is a good thing! We shouldn't be able to typecheck over hidden data such as passwords.

