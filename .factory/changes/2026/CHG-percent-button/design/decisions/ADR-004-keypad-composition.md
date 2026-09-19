---
schema: dark-factory.dev/adr/v1
id: adr:example-product:0004
type: adr
title: Keypad composition of the percent key
product: example-product
status: proposed
change: chg:example-product:2026:0002
impact: [ui, public_api]
---

## Контекст

`REQ-001` requires that the rendered keypad contains exactly one key labelled `%` (`AC-1`), that every
other label keeps the same multiplicity and is neither missing nor renamed (`AC-2`), and that the
relative order of the pre-existing keys is unchanged — filtering the row-major keypad order down to
the pre-change labels must reproduce the pre-change order element by element, with no transposition
(`AC-3`). `AC-4` additionally requires that no pre-existing key changes behaviour, checked through the
baseline regression suite of `SCN-008`.

The intent's constraint is the same statement from the operator's side: no existing key may be
removed, renamed or reordered; `%` is an addition. `REQ-001` leaves the *position* of the `%` key
explicitly unconstrained, and keyboard/OS-level input mapping as well as localization of the label are
out of scope. `SCN-001` inspects the keypad by reading every label in row-major order, so the
placement decision must be expressible in that same ordered, inspectable structure.

## Решение

1. **Keep the keypad declarative and ordered.** The keypad is described as an ordered list of key
   descriptors `{key_id, label, command, cell}`. The renderer iterates the list row-major and the
   input layer maps a pressed `key_id` to a core command; the label list read in `SCN-001` is exactly
   the descriptor order.
2. **Add exactly one descriptor.** `{key_id: percent, label: "%", command: percent (ADR-002)}`, placed
   in a keypad cell that no pre-existing key occupies — appended after the existing keys (a new cell
   at the end of the grid). Appending cannot shift, split or transpose an existing row.
3. **Do not edit existing descriptors.** Every pre-existing descriptor keeps its `key_id`, its label,
   its command and its relative position byte for byte; this change adds a list entry and one binding,
   and modifies nothing else.
4. No OS-keyboard mapping and no localized label are added (out of scope); the label is the literal
   `%`.

The result is that the keypad's label multiset after the change is `before ∪ {%}` and the filtered
row-major order equals the pre-change order — i.e. `AC-1`, `AC-2` and `AC-3` hold as properties of the
list rather than as layout coincidences.

## Обоснование

- `AC-2` and `AC-3` are mechanical consequences of append-only editing: `after − before = {percent}`
  and `[k for k in after_order if k in before_labels] == before_order` follow directly, with no
  dependence on the grid geometry.
- `AC-4` / `SCN-008` follow from the same property: no pre-existing descriptor and no pre-existing
  binding is touched, so no pre-existing key path can change behaviour.
- Appending is the lowest-risk placement precisely because `REQ-001` does not constrain the position:
  a middle-of-row insertion would move pre-existing keys and make `AC-3` depend on how the layout
  algorithm reflows, which is avoidable risk for no requirement gain.
- Keeping the keypad data-driven rather than hard-coding the new button keeps the descriptor list the
  single place where label sets and order are inspected, matching the acceptance checks.

## Альтернативы

| Вариант | Плюсы | Минусы | Почему не выбран |
| --- | --- | --- | --- |
| Insert `%` inside an existing row, shifting later keys | Compact layout; familiar `%` placement next to `÷` | Moves pre-existing keys; `AC-3` then depends on the layout/reflow algorithm, and a transposition would be a silent regression | Unnecessary constraint risk; `REQ-001` does not constrain the position, so there is nothing to buy |
| Overload an existing key (long-press, secondary label, gesture) | No new cell; the keypad keeps its size | A pre-existing key acquires new behaviour; the label count for `%` becomes ambiguous; `AC-2`/`AC-4` and the intent constraint ("no existing key renamed") are directly at risk | Violates the change constraint, which is the one hard limit in the intent |
| Render `%` outside the keypad grid (toolbar button, floating control, menu item) | No keypad layout change at all | `REQ-001` AC-1…AC-3 enumerate the keypad's labels, so a control outside the keypad is not a keypad key; the inspection in `SCN-001` would not see it | Fails the requirement's framing of `%` as a keypad key |
| Add the key behind a feature flag or configuration toggle | Flexible rollout; easy A/B | Makes the acceptance checks environment-dependent and the label set non-deterministic; adds config surface with no requirement behind it | Over-engineering; `AC-1` expects exactly one `%` key unconditionally |
| Reorder the whole keypad into a new, "more standard" layout including `%` | Modern layout; `%` in a natural place | Directly violates the constraint and `AC-2`/`AC-3` | Explicitly out of scope and forbidden by the intent |

## Последствия

- The keypad becomes fully data-driven: one new descriptor plus one binding; the new `key_id` and
  command name are the only additions to the (internal) public surface.
- UI baseline artifacts (recorded label lists and key order from the pre-change build) must be
  regenerated to expect the extra `%` label while `SCN-001` step 5 and `SCN-008` confirm nothing else
  moved.
- Verification: `SCN-001` and `SCN-008`; criteria `REQ-001` AC-1…AC-4.
- A later change of placement is an edit of the descriptor's cell only; `key_id: percent` and the id
  `adr:example-product:0004` stay stable.
