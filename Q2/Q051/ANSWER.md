# Q051 — Your answer

## (d) HashMap order vs Python dict order

A `HashMap` gives no ordering guarantee at all: it stores keys in buckets chosen by
`key.hashCode()` modulo the table size, so the iteration order depends on the hash codes,
the number of buckets (which grows as entries are added, triggering rehashing), and the
resolution order of any collisions. `String.hashCode()` is specified and stable across
machines and JVMs, but the *bucket layout* built on top of it is an implementation detail
that can differ between JVM/JDK versions (and even between runs, if the table resizes at a
different point because entries were added in a different order). So `map.keySet()` on a
`HashMap` can print `[SH, IR, WDI, WF]` on one machine/JDK and a different permutation on
another — it happens to look sorted here only by coincidence of the hash values chosen for
these particular short strings.

Python's `dict` makes a real guarantee, not a coincidence: since Python 3.7 a `dict`
preserves **insertion order** — keys are reported in the order they were first inserted,
regardless of hash values, table resizes, or Python version. That guarantee is *not*
alphabetical order; it only matches the alphabetical ranking in this program because the
log's first occurrences of `WF, IR, WDI, SH` happen to be inserted in that same relative
order here. To get a guaranteed **alphabetical** order in Java you need a `TreeMap` (sorted
by key) or a `LinkedHashMap` (insertion order, like Python's dict) instead of a `HashMap`.
