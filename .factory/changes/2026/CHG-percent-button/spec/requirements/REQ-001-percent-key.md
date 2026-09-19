---
schema: dark-factory.dev/requirement/v1
id: req:example-product:calculator:percent-key
type: requirement
title: Percent key in the keypad
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# REQ-001 — Percent key in the keypad

When the calculator keypad is rendered, a key labelled `%` is present. Every key that existed before
the change keeps its label, its function and its relative order on the keypad.

## Acceptance criteria

- **AC-1** The rendered keypad contains exactly one key whose label is `%`; the count of keypad keys
  whose label is `%` is 1.
  Check: render the keypad, read all key labels, count the labels equal to `%`.
- **AC-2** The set of keypad labels after the change equals the set of keypad labels before the change
  plus the one element `%`; no pre-change label is missing and no pre-change label is renamed.
  Check: `after_labels − before_labels == { "%" }` and `before_labels − after_labels == ∅`, where
  `before_labels` is the keypad label list recorded from the pre-change build.
- **AC-3** The relative order of the pre-change keys is unchanged: filtering the row-major order of the
  rendered keypad down to the pre-change labels reproduces the pre-change key order element by
  element, with no transposition.
  Check: `[k for k in after_order if k in before_labels] == before_order`, read row-major (left to
  right, top to bottom). The position of the new `%` key is unconstrained by this criterion.
- **AC-4** No pre-change key changes behaviour: for every key sequence in the baseline regression
  suite that does not press `%`, the display after the sequence is character-identical to the display
  the pre-change build produces for the same sequence.
  Check: run the baseline suite on both builds; compare display strings, not numeric values
  (e.g. `12 + 34 =` → `46`, `5 × 6 =` → `30`, `7 ÷ 2 =` → `3.5`, `C` then `12 + 3 =` → `15`).

## Out of scope for this requirement

- Which keypad cell the `%` key occupies; only presence and the preservation of the existing order
  are constrained here.

## Traceability

| Criterion | Scenario |
| --- | --- |
| AC-1 | `scenario:example-product:calculator:percent-key-present` |
| AC-2 | `scenario:example-product:calculator:percent-key-present` |
| AC-3 | `scenario:example-product:calculator:percent-key-present` |
| AC-4 | `scenario:example-product:calculator:existing-keys-regression` |
