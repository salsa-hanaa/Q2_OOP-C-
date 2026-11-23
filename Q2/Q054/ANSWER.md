# Q054 — Your answer

## (a) The eight lines

1. `totalKg(shirts)` — **compiles.** `List<Shirt>` matches `List<? extends Garment>`
   because `Shirt` is a `Garment`; the parameter is read-only (producer), which is
   exactly what `? extends` allows.
2. `totalKg(anything)` — **does not compile.** `anything` is `List<Object>`;
   `Object` is not a subtype of `Garment` (it's a supertype), so `List<Object>` is not a
   `List<? extends Garment>`.
3. `addSpareTowels(basket, 2)` — **compiles.** `basket` is `List<Garment>`; `Garment` is
   a supertype of `Towel`, so `List<Garment>` matches `List<? super Towel>` (it is a
   valid place to write `Towel`s into).
4. `addSpareTowels(bathTowels, 1)` — **does not compile.** `bathTowels` is
   `List<BathTowel>`; `BathTowel` is a *subtype* of `Towel`, not a supertype, so
   `List<BathTowel>` does not satisfy `? super Towel` (writing a plain `Towel` into a
   list that is only guaranteed to hold `BathTowel`s would corrupt it).
5. `transfer(bathTowels, towels)` — **compiles.** `T` is inferred as `Towel`:
   `bathTowels` (`List<BathTowel>`) matches `Collection<? extends Towel>`, and `towels`
   (`List<Towel>`) matches `Collection<? super Towel>`.
6. `transfer(towels, shirts)` — **does not compile.** There is no type `T` for which
   both `List<Towel>` is `? extends T` and `List<Shirt>` is `? super T` — `Towel` and
   `Shirt` share no subtype/supertype relationship (only the common ancestor
   `Garment`/`Object`), and the inferred `T` (`Garment` at best) would make `shirts` fail
   `? super Garment` (a `List<Shirt>` cannot safely accept an arbitrary `Garment`).
7. `List<Garment> g = shirts;` — **does not compile.** Generics are invariant:
   `List<Shirt>` is not a `List<Garment>` even though `Shirt` is a `Garment` — if it were
   allowed, `g.add(new Towel(...))` would then silently corrupt `shirts`.
8. `view.add(new Shirt("x"));` — **does not compile.** `view` has type
   `List<? extends Garment>`; the compiler does not know the *exact* captured type (it
   could be `List<Towel>`, as it is here, or any other subtype), so it refuses every
   `add` except `add(null)` — only reads are safe through a `? extends` view.

## (c) The rule

**PECS — Producer Extends, Consumer Super.** If a parameterized type only *produces*
values for the method to read (you call `get`/iterate it but never `add` to it), use
`? extends T`. If it only *consumes* values the method writes into it (you `add` but
never rely on what `get` returns being a specific type), use `? super T`. If a type
parameter is both read and written through the *same* variable, you cannot use a
wildcard there at all — a plain, unbounded type variable is needed instead.

Applied to `transfer(Collection<? extends T> src, Collection<? super T> dst)`: `src` is
pure producer (the method only reads elements out of it to copy them), so it gets
`? extends T`; `dst` is pure consumer (the method only adds elements to it, never reads
them back as a specific type), so it gets `? super T`. `T` itself is introduced as a
method type parameter precisely because `src`'s element type and `dst`'s accepted type
must be tied together across the two wildcards — neither `?` alone could express "the
same T flows from one side to the other."

## (d) Python

With `total_kg(items: Sequence[Garment])`, `add_spare_towels(dst: MutableSequence[Towel], n)`,
`add_spare_towels_anywhere(dst: Sink[Towel], n)` and `transfer(src: Iterable[T], dst: List[T]) -> int`,
`mypy` rejects exactly the same calls Java refuses to compile, plus the final deliberately
unsafe one in `main`:

- `total_kg(anything: List[object])` — rejected: `object` is not a `Garment`, and
  `Sequence` is covariant in its element type but that still requires the element type to
  be a `Garment` subtype.
- `add_spare_towels(bath_towels, 1)` where `bath_towels: List[BathTowel]` — rejected:
  `MutableSequence[Towel]` is invariant (a mutable container can't safely be
  "narrowed" the other way, same reasoning as Java's `? super`), so a
  `List[BathTowel]` is not a `MutableSequence[Towel]`.
- `transfer(towels, shirts)` — rejected: `shirts: List[Shirt]` doesn't match
  `dst: List[T]` once `T` is pinned to `Towel` by the source.
- `g: List[Garment] = shirts` — rejected: `List[T]` (from the generic `list`/`List`) is
  invariant in Python's type system too, exactly like Java's generics.
- `transfer([Towel("rogue")], only_shirts)` (where `only_shirts: List[Shirt]`) — rejected
  by mypy, but Python itself runs it anyway at runtime (hence the `# type: ignore[misc]`
  in the code): this is the concrete illustration that type hints are erased/unchecked at
  runtime, unlike Java's compile-time rejection.

**Expressing `? super Towel` in Python**: there is no `super`/contravariant bound
built into `List`/`Sequence` directly, because those are declared *covariant* (read-only
containers, like `? extends`) or *invariant* (mutable containers, like plain `List[T]`).
To get genuine contravariance — "anything I can call `.append(a_towel)` on" — you declare
your own `Protocol` with a `TypeVar` marked `contravariant=True`, exactly as `Sink` does
here:

```python
T_contra = TypeVar("T_contra", contravariant=True)

class Sink(Protocol[T_contra]):
    def append(self, item: T_contra, /) -> None: ...

def add_spare_towels_anywhere(dst: Sink[Towel], n: int) -> None: ...
```

`Sink[Towel]` then accepts any object whose `append` takes a `Towel` *or a supertype of
`Towel`* — e.g. `Sink[object]` is a `Sink[Towel]` — which is precisely `List<? super Towel>`'s
meaning, just spelled with Python's three variance keywords (`covariant`, `contravariant`,
or neither for invariant) instead of Java's `extends`/`super` wildcards.
