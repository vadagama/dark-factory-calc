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
if such a mechanism exists, and otherwise the minimal contract change must be justified. The answer
therefore has three parts — *what counts as the keypad's public API?*, *does such a mechanism exist?*,
and *what exactly would the fallback change?* — and all three are settled before the proposal is
stated.

*What counts as the keypad's public API* is fixed by `ADR-005` and consumed here rather than
re-derived: the keypad's exported/consumed surface — the descriptor schema and the meaning of its
fields, the render/bind/query entry points and their signatures, the semantics of `key_id` and the
supported value domain of the descriptor fields. The *values* of the shipped layout data, the internal
construction of that data and the rendered result are explicitly **not** part of that API. The
distinction that carries this decision is therefore *extensional* versus *intensional* change: a
keypad whose descriptor list contains one more entry has changed extensionally — that is exactly what
`AC-1` demands and what any implementation of this change must do — whereas schema, signatures and
semantics are the intensional part, which this ADR must leave untouched.

*Does a layout extension mechanism exist?* Nothing in the context of this ChangeSet declares one. The
baseline decisions contain a single placeholder (`adr:example-product:0001`,
`.factory/decisions/ADR-000-example.md`, asynchronous welcome-email delivery) and no keypad, layout,
registry, plugin or configuration surface is referenced by the baseline requirements, the product
description or the scenarios of this ChangeSet. The implementation's source tree is not part of the
context this phase can inspect, so the architecture cannot certify the presence or the absence of
such a mechanism. The decision must therefore hold under both realities instead of assuming one — a
contingent answer is affordable here precisely because, as the decision below shows, both branches
produce the same observable keypad and the same (unchanged) public API.

*What the fallback changes.* The smallest edit that produces the additional descriptor value: the two
values named in point 1 are written at the construction site of the built-in layout data, using only
constructs that site already uses. It is not a contract change: it introduces no exported name, no
descriptor field, no function parameter, no callable, no callback and no configuration key, so by
`ADR-005` points 1–3 it is a data change with an empty API delta — the "minimal contract change" the
rework order asks to justify is, on inspection, a zero contract change.

## Решение

1. **Add the key as data, never as new keypad code and never as a new keypad contract.** The
   structural delta is exactly two additive values: one descriptor
   `{key_id: percent, label: "%", command: percent}` in a keypad cell that no pre-existing key
   occupies, and one binding entry `key_id: percent → command id percent` (`ADR-002`). Both are
   values in structures the keypad already consumes; neither introduces a field, a type, a callback,
   a parameter, an exported name or an entry point.
2. **Use the layout's own extension point when the implementation declares one.** If the keypad layout
   is assembled through a registration hook, a pluggable descriptor list, a layout profile/override
   file or an equivalent mechanism, both values are registered through that point and the built-in
   layout definition is not edited at all. The mechanism's own semantics apply; the keypad's public
   API is untouched.
3. **Otherwise place the two values at the construction site of the built-in layout data, using only
   constructs that are already there** — one more entry in the internal ordered descriptor list, or
   one more call to the existing builder/registration that the built-in layout itself uses. The edit
   stays inside the layout *data*: no new exported name, descriptor field, function parameter,
   callable, callback, configuration key or module boundary is added, so by `ADR-005` this branch also
   has an empty contract delta. Appending at the end of the row-major order cannot shift, split or
   transpose an existing row, and `REQ-001` does not constrain the position, so this fallback needs
   no layout rule to be re-derived and no reflow logic to be trusted.
4. **Exhaustive list of what does not change in either branch**, i.e. the elements that must be
   identical before and after (the freeze check of `ADR-005` point 5): descriptor schema and the
   meaning of its fields; the render, bind and query entry points and their signatures; the semantics
   of `key_id`; the `key_id`, label, command and cell of every pre-existing descriptor; the relative
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
   arithmetic, so the keypad's API cannot grow with percent's semantics. The command-surface delta of
   percent belongs to `ADR-002`, whose subject is the core (and whose own alternatives explain why
   pushing percent into the keypad instead would change the *keypad* API for no requirement gain).
7. **Escalation rule.** If, in the implementation, the extra descriptor value cannot be introduced
   through branch 2 or branch 3 without adding an element of the keypad's public API as defined in
   `ADR-005`, the implementation stops and the question returns to the operator. This design does not
   authorise growing the keypad's API in order to place one data value; a keypad that can only accept
   its keys through a public API change would make the requirement's framing, not this ADR, the thing
   to revisit.

## Обоснование

- **The rework's requested variant is satisfied by construction, in both branches.** The instruction
  asks for a variant without a keypad public-API change. Under the boundary fixed in `ADR-005`,
  point 4 is an exhaustive list of the API elements and none of them changes; the change is confined
  to the values of two structures the keypad already reads. The condition "if such a mechanism
  exists" is handled by making it a branch of the decision (point 2) rather than an assumption:
  whichever branch applies, the contract claims of point 4 are the same, so no requirement, criterion
  or scenario of this ChangeSet becomes branch-dependent.
- **The fallback is minimal, and its smallest possible form is exactly what it does.** `AC-1` demands
  one additional rendered label, so at least one descriptor must exist that did not exist before; the
  design cannot be smaller than one descriptor plus its binding. Points 2 and 3 place those two values
  wherever the implementation already keeps layout values, and nowhere else. By `ADR-005` point 3 the
  fallback is *not* a contract change — the contract delta is zero and the internal-data delta is two
  values — which is the justification the rework order asks for and the reason no requirement or
  scenario changes shape because of it.
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
  state (`a`, `op`) that lives in the core, would force every consumer of the descriptor schema to be
  re-verified, and would turn a data placement into an API growth with a single consumer — the
  outcome the rework order excludes.

## Альтернативы

| Вариант | Плюсы | Минусы | Почему не выбран |
| --- | --- | --- | --- |
| Register the two values through an existing layout extension point (registration hook, pluggable descriptor list, layout profile/override, extra-keys configuration) | Zero edits to the built-in layout data; the addition lives where the implementation already expects additions; no schema, signature or semantic change | Exists only if the implementation already declares such a point; the entries then inherit that mechanism's semantics and ordering rules, which must be checked against `AC-3` | Not rejected — it is the prescribed branch when the mechanism exists (Решение, п. 2). It cannot be the only branch, because no such mechanism is declared anywhere in the baseline available to this phase, so the design must also be well defined without it |
| Append the two values to the built-in layout data at the construction site, at a free cell at the end of the row-major order | Smallest possible delta — two data values; contract delta is empty by `ADR-005`; `AC-2`/`AC-3` hold mechanically; independent of the layout algorithm | Touches the shipped layout data (one added entry); the file that holds the data is edited | This is the fallback branch (Решение, п. 3), used only when no extension point exists, and it is the answer to "justify the minimal contract change": the contract change is zero, the data change is two values |
| Assume the extension mechanism exists (declare it in the design) and describe only branch 2 | A single, simple implementation path; no contingency in the text | An unverified assumption about code this phase cannot inspect; if false, the design gives no way to place the key and the implementation would have to improvise | Guessing implementation facts is exactly what the architecture phase must not do; the branch-complete form costs nothing because both branches are observationally equivalent |
| Assume no mechanism exists and edit the built-in layout data unconditionally | One concrete path; no conditional text | Discards an extension point that may exist, replacing a registration with an edit of shipped data for no benefit | Loses the rework order's first preference where it is available; the branch in point 2 is the cheaper, safer option when the mechanism exists |
| Change the keypad's public API: a new renderer/registration entry point, or new descriptor fields (e.g. `extra_keys` parameter, `is_extension` flag, `handler` callback) | An explicit, self-documenting extension surface; might be reusable by later changes | An intensional contract change for a change that needs only data; every consumer of the schema and of the entry points must be updated and re-verified; the new element would serve exactly one key | `AC-1`…`AC-4` are all satisfiable with two data values (`ADR-005` point 5), so no requirement justifies the API change; the rework order asks specifically for the variant *without* a keypad API change, and this is the variant it excludes |
| Introduce a declarative layout profile file for the whole keypad and put `%` in it | Fully data-driven; no code change at all | Adds a configuration surface and a loading path that no requirement asks for; the effective layout depends on a file that the checked-in built-in layout can silently drift from | Scope creep with a determinism cost: the label set read by `SCN-001` would become environment-dependent. If the implementation *already* loads its layout from a profile, that profile is the extension point of Решение, п. 2 — no separate introduction is needed |
| Insert `%` inside an existing row, shifting later keys | Compact layout; familiar `%` placement next to `÷` | Moves pre-existing keys; `AC-3` then depends on the layout/reflow algorithm, and a transposition would be a silent regression | Unnecessary constraint risk; `REQ-001` does not constrain the position, so there is nothing to buy |
| Overload an existing key (long-press, secondary label, gesture) | No new cell; the keypad keeps its size | A pre-existing key acquires new behaviour; the label count for `%` becomes ambiguous; `AC-2`/`AC-4` and the intent constraint ("no existing key renamed") are directly at risk | Violates the change constraint, which is the one hard limit in the intent |
| Render `%` outside the keypad grid (toolbar button, floating control, menu item) | No keypad layout change at all | `REQ-001` AC-1…AC-3 enumerate the keypad's labels, so a control outside the keypad is not a keypad key; the inspection in `SCN-001` would not see it | Fails the requirement's framing of `%` as a keypad key |
| Rearrange the whole keypad into a new, "more standard" layout including `%` | Modern layout; `%` in a natural place | Directly violates the constraint and `AC-2`/`AC-3` | Explicitly out of scope and forbidden by the intent |
| Add the key behind a feature flag or configuration toggle | Flexible rollout; easy A/B | Makes the acceptance checks environment-dependent and the label set non-deterministic; adds config surface with no requirement behind it | Over-engineering; `AC-1` expects exactly one `%` key unconditionally |

## Последствия

- **The keypad's public API is untouched, and that is visible in this ADR's metadata**: the impact is
  narrowed to `ui` — what changes is the rendered keypad (one more key with a fixed label) — while the
  keypad's exported surface is frozen (`ADR-005`). There is no `public_api` impact on the keypad in
  either branch; the additive command-surface change of percent is recorded under `public_api` in
  `ADR-002`, whose subject is the core.
- **The rework order's fallback half is answered in the ADR itself**: the minimal contract change of
  branch 3 is a contract change of *zero*, plus one descriptor value and one binding value of internal
  layout data (Решение, п. 3, and `ADR-005` points 3–5). The rework's preferred variant (branch 2) and
  its fallback (branch 3) therefore differ only in where a data value is written, never in whether an
  API changes.
- Implementation duty, explicit and small: locate the layout data; if it is assembled through a
  registration or override point, put the two values there (point 2); otherwise add them at the
  construction site of the built-in data (point 3). Both branches must produce identical rendered
  labels and identical order; the branch taken is noted in the implementation change and is invisible
  to `SCN-001`/`SCN-008`.
- Two factual questions about the implementation's keypad remain open from the previous round (whether
  it exposes a layout extension point at all; and, for a fixed grid with no free cell, whether
  extending the grid geometry is acceptable). They can only be answered from the implementation, and
  this ADR is written so that either answer selects a branch without altering any claim of Решение,
  п. 4: `REQ-001` AC-3 constrains only the relative order of the pre-existing keys, so a new cell at
  the end of the row-major order is compliant. They stay non-blocking.
- Keypad baseline artifacts (recorded label lists and key order from the pre-change build) must be
  regenerated to expect the extra `%` label, while `SCN-001` step 5 and `SCN-008` confirm nothing else
  moved.
- If the implementation's keypad is a fixed grid rather than data, the same two values apply at the
  definition site: one added button in a new cell at the end of the row-major order, which preserves
  the pre-existing order by construction. A grid that admits no new key at all without a public API
  element is the case handled by the escalation rule (point 7): the change returns to the operator
  instead of growing the keypad's API.
- Verification: `SCN-001` and `SCN-008`; criteria `REQ-001` AC-1…AC-4; plus the freeze check of
  `ADR-005` point 5 on the keypad's exported surface.
- A later change of placement edits only the descriptor's cell value (or the extension point's entry);
  `key_id: percent` and the id `adr:example-product:0004` stay stable, and this ADR is revised in
  place rather than renumbered.

## Примечание оператора (M3 live)

Правка после согласования: проверяем, что согласование архитектуры становится неактуальным (ADR-039 п.4).
