---
schema: dark-factory.dev/requirement/v1
id: req:example-product:percent-button:percent-key
type: requirement
title: Percent key is available in the existing keypad
product: example-product
status: draft
change: chg:example-product:2026:percent-button
---

# REQ-001 — Percent key is available in the existing keypad

The keypad exposes exactly one key labelled `%`. The key is added without
removing, renaming or moving any pre-existing key (constraint C-1) and is
active in the calculator's default state.

Baseline used by the criteria below (capture B-1): the keypad label set `L0`
and the rendered order `O0` of the build that precedes this change.

## Acceptance criteria

### AC-1 — Exactly one `%` key is present

Statement: In the default state the rendered keypad contains exactly one key
whose visible label is `%`.

Verification: render the default keypad and count keys labelled `%`; the count
must be 1.

Traceability: SCN `keypad-layout-preserved`.

### AC-2 — No pre-existing key is removed or renamed

Statement: The multiset of key labels after the change equals `L0` plus one
`%` label. In particular no label of `L0` is missing and no pre-existing key's
label has changed.

Verification: diff the post-change label multiset against `L0`; the only
difference is the addition of one `%`.

Traceability: SCN `keypad-layout-preserved`.

### AC-3 — Pre-existing keys keep their relative order

Statement: The ordered sequence of labels of pre-existing keys in the rendered
keypad is identical to `O0`; the `%` key is inserted at one position between or
beside existing keys and does not reorder any other key.

Verification: take the post-change label sequence, remove the `%` entry, and
compare the result with `O0`; the sequences must be identical.

Traceability: SCN `keypad-layout-preserved`.

### AC-4 — The `%` key is active in the default state

Statement: In the default state the `%` key is enabled (not visually disabled
or inert) and reacts to activation: pressing `5`, then `0`, then `%` changes
the display from `50` to `0.5`.

Verification: drive the keypad with the sequence `5 0 %` and assert the
display reads `0.5`.

Traceability: SCN `bare-percent`.

### AC-5 — Physical-keyboard parity

Statement: For every operator key of the pre-existing keypad that is bound to
a physical key in the baseline binding map (capture B-2), the change adds a
binding for the `%` character of the active keyboard layout that produces
exactly the same state change as activating the keypad `%` key. If B-2 is
empty, the criterion is satisfied vacuously.

Verification: for each entry of B-2, press the corresponding physical `%`
character with the calculator focused and compare the resulting display and
calculator state with the result of activating the keypad `%` key; the two
must be equal.

Traceability: SCN `keyboard-binding-parity`.
