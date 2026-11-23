# Q053 — Your answer

## (a) Choice of underlying collection

`Bag<T>` stores its counts in a `LinkedHashMap<T, Integer>` (Python: a plain `dict`,
which has the same guarantee since 3.7). A `HashMap`/unordered map would work for
`count`, `remove`, `size` and `distinct`, since those only need O(1) lookup by key — but
`mostCommon()` must break ties by "the one added first", which means the map has to
remember *insertion order*. `LinkedHashMap` gives O(1) get/put/remove like `HashMap`, but
also iterates in the order keys were first inserted, so a single linear scan of
`entrySet()` finds the most-common element with the correct tie-break for free, with no
extra timestamp/sequence field needed. A `TreeMap` was rejected because `T` is not
required to be `Comparable` (sock sizes are, but garment names and arbitrary `Object`
union members are not guaranteed to be).

## (d) `Bag<String> g = new Bag<>(); g.add(42);`

In Java this **fails to compile**: `g` has static type `Bag<String>`, so `add` is
specialized to `add(String item, int n)`; `42` is an `int`, not a `String` (and
autoboxing only turns it into `Integer`, which still is not a `String`), so `javac`
reports a type mismatch before the program ever runs. Generics in Java are a
compile-time-only, static check — there is no `Bag<String>` object at runtime that
"knows" its element type (type erasure), but the compiler refuses to emit the bytecode
for this call in the first place.

Python's `Bag[str]` (via `Generic[T]`/`TypeVar`) carries no such enforcement by the
language itself: `g.add(42)` runs perfectly fine at runtime, because Python never checks
type hints — they are pure documentation for humans and for external tools. A type
checker such as `mypy`, run separately (`mypy bag_demo.py`), *would* flag
`g.add(42)` as an error if `g` were annotated `Bag[str]`, but nothing stops the
interpreter itself from executing it and happily storing the integer `42` as a "string"
bag member.
