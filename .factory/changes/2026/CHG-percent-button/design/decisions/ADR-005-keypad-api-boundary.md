---
schema: dark-factory.dev/adr/v1
id: adr:example-product:0005
type: adr
title: Keypad public API boundary and the key-addition rule
product: example-product
status: proposed
change: chg:example-product:2026:0002
impact: [public_api]
---

## Контекст

The rework order `rw_a63e37d99e9b4589` asks for a variant of this design that does **not** change the
keypad's public API: the key is to be added through the keypad's existing layout extension mechanism
if such a mechanism exists, and otherwise the minimal contract change must be justified. Both halves
of that instruction presuppose an answer to a question that no artifact of this ChangeSet answers
today: *what exactly is the keypad's public API, and which edits of this change fall inside it?* The
fallback half in particular ("justify the minimal contract change") cannot even be evaluated until it
is known which of the change's edits are contract edits and which are data edits.

The context around that question:

- `REQ-001` constrains the **rendered** keypad: exactly one key labelled `%` (`AC-1`), unchanged label
  multiset for every other key (`AC-2`), unchanged relative order of the pre-existing keys (`AC-3`),
  and unchanged behaviour of every pre-existing key (`AC-4`, checked by `SCN-008`). It says nothing
  about the keypad's internal structure or about its exported surface.
- The intent's constraint forbids removing, renaming or reordering pre-existing keys; the goal is
  reached by *adding* one key, which by definition changes the set of rendered keys.
- `ADR-004` must place one descriptor and one binding; `ADR-002` must add one additive command id to
  the **core's** command table. Two different surfaces grow (or not) here, and the rework order's
  question is directed at the keypad's only.
- No baseline artifact declares the keypad's exported surface, and this phase cannot inspect the
  implementation's source tree (the only baseline decision, `adr:example-product:0001`, is an
  unrelated placeholder about asynchronous welcome e-mail). The boundary therefore cannot be stated as
  a list of concrete exported symbols; it must be stated as a rule over *categories of change*, so
  that it is checkable by a diff in any language or framework.

Without such a rule, both candidate answers to the rework order are unfalsifiable: "the public API is
unchanged" could always be denied by pointing at the added descriptor, and the "minimal contract
change" of the fallback could always be claimed to be no contract change at all. This ADR fixes the
rule; `ADR-004` applies it to the composition of the `%` key.

## Решение

1. **The keypad's public API is its exported/consumed surface.** Concretely, the following and only
   the following elements:
   - the descriptor schema — field names, field types, requiredness — and the meaning of each field;
   - the keypad's render, bind and query entry points, their signatures (parameters, return types,
     error behaviour);
   - the semantics of `key_id`, and which command ids a binding may name;
   - the supported value domain of the descriptor fields for the pre-existing keys.
2. **Not part of the public API:** the *values* of the shipped/built-in layout data (which
   descriptors exist, their labels, their cells and their order, and the bindings), the internal
   construction of that data, and the rendered result. These are the keypad's *data* and its *output*,
   not its interface.
3. **Key-addition rule.** Adding a key is, by default, a change to the shipped layout data and not a
   change to the keypad's public API. It *is* an API change if and only if it introduces a new
   exported name, descriptor field, function parameter, callable, callback, configuration key or
   module boundary to the keypad. The rule is stated as a test on the change, not as a claim about
   this particular keypad, so it holds for a data-driven descriptor list and for a layout assembled
   by code alike.
4. **This change obeys the rule with an empty keypad API delta.** Its only structural delta on the
   keypad is one descriptor value and one binding value (`ADR-004`); the only other structural delta
   of the change is one additive command id in the **core's** command table (`ADR-002`), which is not
   on the keypad's surface at all.
5. **Freeze check, mechanically verifiable.** Before and after the change: the descriptor schema has
   the same fields with the same types and requiredness; the render/bind/query entry points have the
   same signatures; `key_id` has the same semantics; and no new descriptor field, parameter,
   callback, exported name or configuration key appears anywhere in the keypad. The only permitted
   structural delta is the two values of point 4; the only permitted observable delta is one more key
   labelled `%` (`REQ-001` AC-1…AC-3, read by `SCN-001`/`SCN-008`).
6. **The rule constrains, it does not only permit.** A later change may of course extend the keypad's
   API, but such an extension carries its own requirement and its own ADR. `%` needs no extension of
   that surface, so it must not create one — a public extension point introduced inside this change
   would be an API growth with a single consumer and no requirement behind it.
7. **The rule is what makes the rework order answerable without assuming implementation facts.**
   Whether the two values of point 4 are written into an existing extension point or at the
   construction site of the built-in layout data (`ADR-004`, branches 2 and 3), points 1–3 are
   satisfied identically and the freeze check of point 5 is the same check. The branch is a
   *location* question (where a data value is written), never a *contract* question.

## Обоснование

- **The boundary must be intensional, because the requirement is extensional.** `REQ-001` AC-1
  demands one additional rendered key; the set of keys is thereby guaranteed to change. If the
  "public API" were defined as the rendered keypad or as everything inside the keypad's definition
  file, the rework order's requested variant ("without changing the public API") would be
  unsatisfiable by construction, i.e. the instruction could not be followed by any design. The only
  coherent reading — and the one that matches the constraint in `intent.md`, which is about labels,
  functions and order, not about files — is the exported surface of points 1–2.
- **Definitions must be checkable, or the claim is rhetoric.** Point 5 turns "the keypad's public API
  is unchanged" into a diff of named elements and signatures plus a test for the absence of new
  fields/parameters/configuration keys. That check is cheap, language-independent, and it is the same
  check in both `ADR-004` branches.
- **The fallback of the rework order needs exactly this rule to be evaluable.** "Justify the minimal
  contract change" is answered by point 3: in the fallback the contract delta is *empty*; what the
  fallback changes is one value of internal layout data. This is a stronger and more precise answer
  than "an internal edit that we argue is not a contract change", because point 3 states the test that
  makes the argument decidable.
- **A rule stated once prevents per-change renegotiation.** `ADR-004` can then argue about
  composition and position only, `ADR-002` about the core's command surface only, and the reviewer
  does not have to re-derive the boundary for every future key.
- **The rule keeps the two surfaces apart**, which is the substantive architectural point of this
  round: the core's command surface is the one that legitimately grows (additively, `ADR-002`), while
  the keypad — the surface the rework order protects — stays a data-only consumer of an unchanged
  interface.

## Альтернативы

| Вариант | Плюсы | Минусы | Почему не выбран |
| --- | --- | --- | --- |
| Define the keypad's public API as the **rendered keypad** (label set and their order) | Directly observable; already machine-checked by `SCN-001` | Under this reading this change *does* change the API, so the rework order's requested variant becomes unattainable although `REQ-001` demands exactly that observable change | Puts the output of the keypad on the same footing as its interface; it makes the instruction "add the key without changing the API" self-contradictory, so no design could satisfy it |
| Define the keypad's public API as **everything in the keypad's source/module**, including the built-in layout data | Maximally strict: zero edits inside the keypad module; easy to check by file diff | Forbids satisfying `REQ-001` AC-1 unless an extension point already exists; where none exists, the change would be impossible without an API change | Converts an internal data edit into a forbidden contract edit for no requirement gain; the intent's constraint is about key labels, functions and order, not about which file holds the data |
| Leave the boundary undefined and argue "no API change" case by case per change | No upfront definition needed | The rework order's question cannot be answered verifiably; every review renegotiates the boundary; claims are unfalsifiable | This is the state the rework order reacted to; an undefined boundary is precisely what makes "no public API change" non-auditable |
| Introduce a public keypad extension point inside this change (a documented registration API, `extra_keys` parameter or similar), then place `%` through it | Self-documenting extension surface; potentially reusable by later keys | An API growth with exactly one consumer and no requirement behind it; the change would then *be* an API change and would have to be reviewed as such; still leaves `ADR-004` with no fallback when the mechanism cannot be added safely | The rework order asks for the variant *without* a keypad API change; inventing the extension point converts the requested variant into the rejected one. If a mechanism already exists it is used (`ADR-004`, branch 2); adding one is a separate change with its own requirement |
| Move the addition out of the keypad's layout data into an application-level wrapper (add the button in app code, keep the keypad as it is) | No keypad data edit at all | The wrapper still has to go through the keypad's render path, and if the keypad offers no way to render a key it does not own, the wrapper needs a new entry point — an API change; `REQ-001` frames `%` as a keypad key and `SCN-001` reads keypad labels | It answers the letter (no keypad edit) but not the substance: it either changes the API it claims to avoid or renders `%` outside the structure the requirement and `SCN-001` inspect |
| Freeze the keypad API by versioning it (a contract version raised by key additions) | Explicit, checkable contract version number | Presupposes that adding a key changes the API (contradicting point 3) and adds a version surface that no requirement asks for | Over-modelling: the freeze check of point 5 gives the same guarantee without a new artifact to keep in sync |
| State the boundary as a list of the actual exported symbols of the implementation | Maximal precision for this keypad | Requires reading the implementation's source, which is not part of this phase's context; the list would be stale the moment the module changes | Not available and not durable; a rule over categories of change is both checkable by diff and independent of the concrete implementation |

## Последствия

- `ADR-004` can now state its claim precisely: the composition of `%` is a data change, its contract
  delta is empty in **both** branches, and the rework order's requested variant is the only variant
  this design permits. `ADR-004`'s own impact stays `ui` (what changes is the rendered keypad).
- The `impact: [public_api]` recorded in this ADR marks that its **subject** is the public API surface
  of the keypad, and that its decision is a *freeze*: the change carries **no** `public_api` delta on
  the keypad. The only `public_api` delta of this ChangeSet is the core's additive command id, and it
  is recorded under `ADR-002`.
- Verification gains one mechanical step in addition to `SCN-001`/`SCN-008`: diff the keypad's
  exported surface before/after (schema fields and types, entry-point signatures, `key_id` semantics)
  and assert that no new descriptor field, parameter, callback, exported name or configuration key
  has appeared. Criteria: `REQ-001` AC-1…AC-4.
- Implementation duty: place the descriptor and the binding in the layout data (`ADR-004`) and do not
  introduce any exported element on the keypad to hold them. If the two values cannot be placed
  without adding a public API element, the implementation stops and the question returns to the
  operator (escalation rule, `ADR-004` point 7) — the requirement, not this boundary, would be what
  has to be revisited.
- The rule outlives this ChangeSet: future key additions are data changes by default, and any future
  request to extend the keypad's API arrives with its own requirement and its own ADR.
- Id stability: `adr:example-product:0005` is a comment anchor. A later change to the boundary
  revises this file in place; the numbers 0002–0005 stay as they are.
