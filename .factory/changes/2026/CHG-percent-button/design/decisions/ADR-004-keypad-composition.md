---
schema: dark-factory.dev/adr/v1
id: adr:example-product:0004
type: adr
title: Keypad composition of the percent key
product: example-product
status: proposed
change: chg:example-product:2026:0002
impact: [ui]
---

## Контекст

`REQ-001` requires that the rendered keypad contains exactly one key labelled `%` (`AC-1`), that every
other label keeps the same multiplicity and is neither missing nor renamed (`AC-2`), and that the
relative order of the pre-existing keys is unchanged — filtering the row-major keypad order down to
the pre-change labels must reproduce the pre-change order element by element, with no transposition
(`AC-3`). `AC-4` additionally requires that no pre-existing key changes behaviour, checked through the
baseline regression suite of `SCN-008`. `REQ-001` leaves the *position* of the `%` key explicitly
unconstrained; keyboard/OS-level input mapping and localization of the label are out of scope.
`SCN-001` inspects the keypad by reading every label in row-major order, so the composition decision
must be expressible in that same ordered, inspectable structure.

**The rework order `rw_a63e37d99e9b4589`.** The order asks for a variant that does not change the
keypad's public API: the key is to be added through the keypad's existing layout extension mechanism
if such a mechanism exists, and otherwise the minimal contract change must be justified. The question
therefore has two parts — *does such a mechanism exist?* and *what exactly would the fallback
change?* — and both are answered before the proposal is stated.

*Does a layout extension mechanism exist?* Nothing in the context of this ChangeSet declares one. The
baseline decisions contain a single placeholder (`adr:example-product:0001`,
`.factory/decisions/ADR-000-example.md`, asynchronous welcome-email delivery) and no keypad, layout,
registry, plugin or configuration surface is referenced by the baseline requirements, the product
description or the scenarios of this ChangeSet. The implementation's source tree is not part of the
context this phase can inspect, so the architecture cannot certify the presence or the absence of
such a mechanism. The decision must therefore hold under both realities instead of assuming one — a
contingent answer is affordable here precisely because, as the decision below shows, both branches
produce the same observable keypad and the same keypad contract.

*What "the keypad public API" means here*, so that the answer is checkable: the descriptor schema
(field names, types, requiredness) and the meaning of its fields; the render, bind and query entry
points of the keypad together with their signatures; the semantics of `key_id`; and the label, the
invoked command and the relative order of every pre-existing key. The distinction that carries the
decision is *extensional* versus *intensional* change: a keypad whose descriptor list contains one
more entry has changed extensionally — that is exactly what `AC-1` demands and what any implementation
of this change must do — whereas schema, signatures and semantics are the intensional part, which this
ADR must leave untouched.

## Решение

1. **Add the key as data, never as new keypad code and never as a new keypad contract.** The delta is
   exactly two additive entries: one descriptor `{key_id: percent, label: "%", command: percent}` in a
   keypad cell that no pre-existing key occupies, and one binding entry `key_id: percent → command id
   percent` (`ADR-002`). Both are values in structures the keypad already consumes; neither introduces
   a field, a type, a callback or an entry point.
2. **Use the layout's own extension point when the implementation declares one.** If the keypad layout
   is assembled through a registration hook, a pluggable descriptor list, a layout profile/override
   file or an equivalent mechanism, the `%` entry is registered through that point and the built-in
   layout definition is not edited at all. The mechanism's own semantics apply; the keypad contract is
   untouched.
3. **Otherwise make the minimal contract change: append one descriptor to the built-in ordered
   descriptor list**, in a free cell at the end of the grid (row-major). Appending cannot shift, split
   or transpose an existing row, and `REQ-001` does not constrain the position, so this fallback needs
   no layout rule to be re-derived and no reflow logic to be trusted.
4. **Exhaustive list of what does not change in either branch.** Descriptor schema and the meaning of
   its fields; the render, bind and query entry points and their signatures; the semantics of
   `key_id`; the `key_id`, label, command and cell of every pre-existing descriptor; the relative
   order of the pre-existing descriptors; the number of keys carrying each pre-existing label; the
   layout geometry and reflow rules; label localization and OS-keyboard mapping (out of scope in any
   case). Everything this change adds is a value in the two structures named in point 1.
5. **The branches are observationally equivalent and therefore interchangeable.** `SCN-001` and
   `SCN-008` observe labels, orders and displays, not which structure produced them; the label
   multiset after the change is `before ∪ {%}` and the filtered row-major order equals the pre-change
   order in both branches. Which branch was taken is an implementation-location detail with no
   requirement behind it, and it is recorded in the implementation change, not in the observable
   behaviour.
6. **Keep the keypad out of percent semantics.** The new descriptor names a command id that already
   exists in the core command table (`ADR-002`); the keypad gains no field, no callback and no
   arithmetic, so the keypad contract cannot grow with percent's semantics. The command-surface delta
   of percent belongs to `ADR-002`, whose subject is the core (and whose own alternatives explain why
   pushing percent into the keypad instead would change the *keypad* contract for no requirement
   gain).

## Обоснование

- **The rework's requested variant is satisfied by construction, in both branches.** The instruction
  asks for a variant without a keypad public-API change. Point 4 is an exhaustive list of the contract
  elements and none of them changes; the change is confined to the values of two structures the
  keypad already reads. The condition "if such a mechanism exists" is handled by making it a branch of
  the decision (point 2) rather than an assumption: whichever branch applies, the contract claims of
  point 4 are the same, so no requirement, criterion or scenario of this ChangeSet becomes
  branch-dependent.
- **The fallback is minimal and it is also unavoidable in its smallest possible form.** `AC-1` demands
  one additional rendered label, so at least one descriptor must exist that did not exist before; the
  design cannot be smaller than one descriptor plus its binding. Points 2 and 3 place those two values
  wherever the implementation already keeps layout values, and nowhere else.
- **`AC-2` and `AC-3` are mechanical consequences of append-only editing**, not layout coincidences:
  `after − before = {percent}` and `[k for k in after_order if k in before_labels] == before_order`
  follow directly from "add one entry, edit none", independently of grid geometry and of how the new
  cell is positioned.
- **`AC-4` / `SCN-008` follow from the same property**: no pre-existing descriptor and no pre-existing
  binding is touched, so no pre-existing key path can change behaviour.
- **Appending is the lowest-risk placement** because `REQ-001` does not constrain the position; a
  middle-of-row insertion would move pre-existing keys and make `AC-3` depend on the layout/reflow
  algorithm — risk bought for no requirement gain.
- **A richer keypad contract is not merely unnecessary, it is harmful here.** Any variant that adds a
  descriptor field or a keypad-side handler (see alternatives) would make the keypad the owner of
  state (`a`, `op`) that lives in the core, and would force every consumer of the descriptor schema to
  be re-verified — an intensional contract change for a data-only requirement.

## Альтернативы

| Вариант | Плюсы | Минусы | Почему не выбран |
| --- | --- | --- | --- |
| Register the `%` descriptor through an existing layout extension point (registration hook, pluggable descriptor list, layout profile/override, extra-keys configuration) | Zero edits to the built-in layout; the change lives where the implementation already expects additions; no schema, signature or semantic change | Exists only if the implementation already declares such a point; the entry then inherits that mechanism's semantics and ordering rules, which must be checked against `AC-3` | Not rejected — it is the prescribed branch when the mechanism exists (Решение, п. 2). It cannot be the only branch, because no such mechanism is declared anywhere in the baseline available to this phase, so the design must also be well defined without it |
| Append one descriptor to the built-in ordered descriptor list at a free cell | Smallest possible delta; `AC-2`/`AC-3` hold mechanically; independent of the layout algorithm | Touches the built-in layout definition (one added line/entry) | This is the fallback branch (Решение, п. 3), used only when no extension point exists; it is the minimal contract change the rework asks to justify — its delta is enumerated exhaustively in Решение, п. 4 |
| Change the keypad's public API: a new renderer/registration entry point, or new descriptor fields (e.g. `extra_keys` parameter, `is_extension` flag, `handler` callback) | An explicit, self-documenting extension surface; might be reusable by later changes | An intensional contract change for a change that needs only data; every consumer of the schema and of the entry points must be updated and re-verified; the new field would be used by exactly one key | `AC-1`…`AC-4` are all satisfiable with one data entry, so no requirement justifies the API change; the rework order asks specifically for the variant *without* a keypad API change, and this is the variant it excludes |
| Introduce a declarative layout profile file for the whole keypad and put `%` in it | Fully data-driven; no code change at all | Adds a configuration surface and a loading path that no requirement asks for; the effective layout depends on a file that the checked-in built-in layout can silently drift from | Scope creep with a determinism cost: the label set read by `SCN-001` would become environment-dependent. If the implementation *already* loads its layout from a profile, that profile is the extension point of Решение, п. 2 — no separate introduction is needed |
| Insert `%` inside an existing row, shifting later keys | Compact layout; familiar `%` placement next to `÷` | Moves pre-existing keys; `AC-3` then depends on the layout/reflow algorithm, and a transposition would be a silent regression | Unnecessary constraint risk; `REQ-001` does not constrain the position, so there is nothing to buy |
| Overload an existing key (long-press, secondary label, gesture) | No new cell; the keypad keeps its size | A pre-existing key acquires new behaviour; the label count for `%` becomes ambiguous; `AC-2`/`AC-4` and the intent constraint ("no existing key renamed") are directly at risk | Violates the change constraint, which is the one hard limit in the intent |
| Render `%` outside the keypad grid (toolbar button, floating control, menu item) | No keypad layout change at all | `REQ-001` AC-1…AC-3 enumerate the keypad's labels, so a control outside the keypad is not a keypad key; the inspection in `SCN-001` would not see it | Fails the requirement's framing of `%` as a keypad key |
| Rearrange the whole keypad into a new, "more standard" layout including `%` | Modern layout; `%` in a natural place | Directly violates the constraint and `AC-2`/`AC-3` | Explicitly out of scope and forbidden by the intent |
| Add the key behind a feature flag or configuration toggle | Flexible rollout; easy A/B | Makes the acceptance checks environment-dependent and the label set non-deterministic; adds config surface with no requirement behind it | Over-engineering; `AC-1` expects exactly one `%` key unconditionally |

## Последствия

- **The keypad's public contract is untouched, and that is visible in this ADR's metadata**: the
  impact is narrowed to `ui` — what changes is the rendered keypad (one more key with a fixed label),
  not the keypad's API. There is no `public_api` impact on the keypad in either branch; the additive
  command-surface change of percent is recorded under `public_api` in `ADR-002`, whose subject is the
  core.
- Implementation duty, explicit and small: locate the layout definition; if it exposes a registration
  or override point, put the descriptor and the binding there (point 2); otherwise append the single
  descriptor to the built-in list (point 3). Both branches must produce identical rendered labels and
  identical order; the branch taken is noted in the implementation change and is invisible to
  `SCN-001`/`SCN-008`.
- Keypad baseline artifacts (recorded label lists and key order from the pre-change build) must be
  regenerated to expect the extra `%` label, while `SCN-001` step 5 and `SCN-008` confirm nothing else
  moved.
- If the implementation's keypad is a fixed grid rather than data, the same minimal delta applies at
  the definition site: one added button in a new cell at the end of the row-major order, which
  preserves the pre-existing order by construction. A grid with no room for a new cell at all is the
  only case in which this decision needs operator input (a layout-geometry question, not a contract
  question); it is flagged as a non-blocking question in this round.
- Verification: `SCN-001` and `SCN-008`; criteria `REQ-001` AC-1…AC-4.
- A later change of placement edits only the descriptor's cell value (or the extension point's entry);
  `key_id: percent` and the id `adr:example-product:0004` stay stable, and this ADR is revised in
  place rather than renumbered.
