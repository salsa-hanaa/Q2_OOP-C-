# Q060 — Your answer

## (a) The trace

| # | Step | handled | label |
|---|------|---------|-------|
| 0 | start | 0 | (no quote yet) |
| 1 | type 4.2 | 0 | (no quote yet) — typing fires no event, so nothing runs |
| 2 | click Quote | 1 | WF 4.5 kg: Rp31,500, ready in 48 h |
| 3 | tick Express | 2 | WF 4.5 kg express: Rp47,250, ready in 12 h |
| 4 | choose WDI | 3 | WDI 4.5 kg express: Rp67,500, ready in 18 h |
| 5 | type abc | 3 | WDI 4.5 kg express: Rp67,500, ready in 18 h — unchanged, see (b) |
| 6 | press Enter | 4 | invalid weight |
| 7 | untick Express | 5 | invalid weight |
| 8 | choose IR | 6 | invalid weight |
| 9 | type 0.4, click | 7 | IR 2.0 kg: Rp10,000, ready in 24 h |

Each listener fires exactly once per user action that actually delivers an event: typing
alone (steps 1 and 5) delivers none, so `handled` does not move; every click, Enter, combo
selection and check-box toggle delivers exactly one event, so `handled` increases by 1 per
step from step 2 onward.

## (b) The stale label

Typing a character into a `JTextField` does **not** fire an `ActionEvent` — Swing only
delivers an `ActionEvent` from a text field when the user presses Enter (or when
`postActionEvent()` is called programmatically). So step 5 ("type abc") changes the text
the field *holds*, but delivers no event to any listener: `refresh` never runs, `handled`
stays at 3, and the label keeps showing the quote computed back in step 4 from the old
text "4.2" — "WDI 4.5 kg express: Rp67,500, ready in 18 h" — even though the field now
reads "abc". The label and the field have become inconsistent purely because nothing told
the controller the text had changed.

To keep the label in sync with every keystroke (not just Enter), the fix is to listen to
the field's **document**, not to its action event: attach a `DocumentListener` to
`kg.getDocument()` (its `insertUpdate`/`removeUpdate`/`changedUpdate` callbacks) and call
`refresh(false)` from there. A `DocumentListener` fires on every edit to the field's
content, so the label would immediately go stale/re-quote (or show the error text) as
soon as "a" is typed, instead of only when Enter or another control is next triggered.
