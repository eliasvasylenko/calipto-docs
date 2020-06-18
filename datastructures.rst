Persistent Data Structures
==========================

Linear modification
-------------------

When we modify collections linearly (i.e. can statically prove that the previous version of the collection is thrown away) we can use inline mutating versions of operations as an optimisation.

Theoretically we could even detect when mutliple linear operations happen in sequence and optimise them together, for instance to pre-allocate a list when we know how many elements will be added. This is far more complex though.

Maps & Sets
-----------

Unordered: CHAMP, HHAMT

Ordered: Who knows!




If we have collections based on prefixes, i.e. Trie and descendents, we can generalise this to tree data structures not just strings or binary blobs. Probably fastest to do breadth-first and to have special markers for END and SUBLIST, so e.g. ((a b) c) would be ``SUBLIST c END a b END``. This might mean we're less likely to have to chase pointers when going through the prefix, but this is dependent on how things are laid out, and the ordering it produces will probably be unintuitive if that's important.


Lists
-----

RRB Vector https://dl.acm.org/doi/10.1145/2784731.2784739
