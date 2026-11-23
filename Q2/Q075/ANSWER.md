# Q075 — Your answer

## (a) The trace

| # | Event | Rows shown |
|---|-------|------------|
| 0 | attach | A2 Budi, A4 Dewi, A5 Rahmat, A1 Sari, A6 Citra, A3 Ayu — the whole book, sorted by ready time (11:00→16:00) |
| 1 | filter "ra" | A5 Rahmat (WASHING, 13:00), A6 Citra (DRYING, 15:00) — only customer names containing "ra", still sorted by ready time |
| 2 | sort by customer | A6 Citra, A5 Rahmat — same two rows, reordered alphabetically by customer |
| 3 | add A7 Budi | **no update** — "Budi" doesn't contain "ra", so A7 isn't in the filtered set and the two shown rows are unchanged |
| 4 | advance A5 | A6 Citra, A5 Rahmat — same two orders in the same order, but A5's row text now reads WASHING→DRYING, so the view *is* notified |
| 5 | filter "RA" | **no update** — case-insensitive, so "RA" normalizes to the same filter as "ra"; the result is identical to what's already shown |
| 6 | only DRYING | **no update** — both A5 and A6 are already DRYING, so adding the status filter removes nobody |
| 7 | advance A6 | A5 Rahmat (DRYING, 13:00) — A6 becomes READY and no longer matches the DRYING filter, leaving only A5 |
| 8 | filter cleared | **no update** — the status filter (DRYING) alone still matches only A5, exactly what's already shown |
| 9 | only READY | A2 Budi, A6 Citra, A4 Dewi — every READY order (name filter is empty), sorted by customer (B < C < D) |

Final tally: **6 updates for 10 events** (events 0, 1, 2, 4, 7, 9 change what the view sees;
events 3, 5, 6, 8 recompute internally but leave the shown rows identical, so the view is
never touched).

## (c) Why compare

**Why compare instead of notifying on every event.** The model fires its "something
changed" signal on *every* mutation (an add, an advance), and the view-model recomputes on
every setter too — but many of those recomputations land on the exact same filtered,
sorted, formatted rows as before (events 3, 5, 6, 8 in part (a)). If the view-model told
the view on every underlying event regardless, a widget like a `JList`/`Treeview` would
re-render, lose scroll position, flicker, or drop a user's in-progress selection for a
change that produced no visible difference at all. Comparing `rows` against the
previously-shown `shown` and only calling `view.accept(rows)` when they differ means the
view is touched exactly when there is something new to look at — the view-model absorbs
the "did this actually matter" decision so neither the model nor the view has to.

**Why it's easy to test.** The view-model has no widgets in it at all: `OrderListViewModel`
depends only on `OrderBook` and a plain `Consumer<List<String>>`/callable, both of which a
test can fake trivially (an in-memory book of `Order`s, and a lambda that appends to a
list, exactly as `main` already does for its own trace). There's no `JFrame`, no event
dispatch thread, no pixels to render and nothing to click — a test simply calls
`setFilter`/`setOnlyStatus`/`setSortByCustomer` or mutates the book directly and asserts on
the plain `List<String>` the callback received and on `updates`. Because the "did it
change" comparison is pure data (`List.equals`), the whole class can be exercised and
verified headlessly, the same way `Q060`'s `quote`/`refresh` logic was tested by peeling
the rules away from Swing.

## (d) Chapter 9 ideas

- **Immutable snapshots / defensive copying**: `OrderBook.orders()` returns
  `List.copyOf(orders)` (Python: a `tuple(self._orders)`), so the view-model — or anyone
  else — iterating the book's orders can never see it mutate mid-iteration and can never
  accidentally corrupt the model's internal list; it is a read-only *view* of the data,
  the same idea as `Bag.asMap()` in Q053 and `List<? extends Garment>` in Q054.
- **Comparator composition / a comparator chain**: `BY_READY` and `BY_CUSTOMER` are built
  with `Comparator.comparingInt`/`comparing(...).thenComparing(...)` — the Chapter 9 idiom
  for a primary key plus a tie-break, used again here exactly as in Q051 part (c)'s
  two-key ranking.
- **Generic functional interfaces as collaborators**: the view-model takes a
  `Consumer<List<String>>` rather than any concrete widget type — a generic interface
  used to decouple "what should happen with the computed rows" from "how a UI renders
  them", the same `Consumer`/`Runnable`/functional-interface style Chapter 9 introduces
  for plugging interchangeable behavior into a class.
- **A stream pipeline over `List<T>`**: `filter → filter → sorted → map → toList()` chains
  generic `Stream<Order>`/`Stream<String>` operations whose type parameters flow through
  each stage, rather than hand-written loops with casts — the generic-collection idiom
  Chapter 9 builds up to and Q075 asks you to apply end-to-end.
