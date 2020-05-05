Modules
=======


The module system is designed for a number of important purposes.

- Encapsulation (atomics, exports)

- Organization (composition via namespacing, separation of concerns & compilation)

- Composition (namespacing)

- Dependency Management & Repeatability (content-addressed imports)

The Calipto module system is implemented entirely in normal Calipto code. It does not require any special features or priviliages from the runtime.

However it does apply additional restrictions on code within the modules which it hosts.

Symbol Visibility
-----------------

The module system injects modified 

Atomics
-------

When we share data from one module to be consumed by others, sometimes we want to protect parts of that data from being read or modified. The way to achieve this in Calipto is with atomics.

An atom is an *indivisible particle*. In Calipto, "indivisible" means "unable to be destructured", and instead of particles of course we're dealing with data. The obvious example of an atom is a symbol, but the module system also allows a module to export more complex compositions of data as atoms, such that they *appear* indivisible to other modules.

There are two sides to atomicity; consider that ``des`` and ``cons`` should be symmetric, such that for everything we construct via ``cons`` we should also be able to destruct it via ``des``. We have already defined an ``atom`` to be a datum which cannot be destructed, and so in turn this must imply that an atom cannot be constructed.

For the system to be consistent both of these invariants must be enforced. The module system achieves this by injecting modified versions of ``des``

To achieve the necessary restrictions on 

To achieve this, every file which is read as a module is injected with modified versions of ``des`` and ``atom`` (and possibly ``cons``) which respect the atomicity of data exported from other modules.

Previous Notes
--------------

.. TODO::
 
  Organise this document!

To protect structured data is more difficult. There are two possible approaches.

- Our first option is to disallow arbitrary structured data from being constructed by adding an extra ``fail`` continuation to the signature of ``cons`` so that we can reject attempts to cons with protected symbols.
  
- Our second option is to disallow arbitrary structured data from being destructured, so that we can reject attempts to des with protecte symbols. This has the unfortunate effect of breaking the semantics that we can determine whether something is a cons cell by attempting to des it. It also has the unfortunate (/fortunate?) effect that we can nolonger expect to be able to reflect over all data. 

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

.. TODO::

  The main problem with this approach appears to be with the reverse process of printing the string. Do we *need* to be able to do this? If our only use-case is pretty printing output then perhaps we can relax our standards of normalisation and uniqueness of string representations.

.. TODO::

  Another problem is that it might be possible for an external module to ``cons`` together a protected form and then be unable to ``des`` it. These operations should always be symmetric, and we need to think of a way to preserve this property!

This is total encapsulation and means that no information can escape unless there is API on the module for it. Typechecking the internals of an atom is still possible if there are functions in the defining module to facilitate the checks, though it can't be done via destructuring on the client side.

