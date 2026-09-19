---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:percent-key-present
type: scenario
title: Keypad shows the percent key alongside every existing key
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Keypad shows the percent key alongside every existing key

The user opens the calculator. The keypad shows the same keys as before, in the same relative order,
plus a key labelled `%`.

## Script

1. Record `before_labels` and `before_order` from the pre-change build: read every keypad label in
   row-major order (left to right, top to bottom).
2. On the changed build, read every keypad label in row-major order → `after_labels`, `after_order`.
3. Assert `after_labels − before_labels == { "%" }`.
4. Assert `before_labels − after_labels == ∅`.
5. Assert `[k for k in after_order if k in before_labels] == before_order`.
6. Assert the number of keys labelled `%` is 1.

## Observable outcome

Exactly one new key, labelled `%`; every pre-change key is present, keeps its label and keeps its
relative position.

## Traces

- `req:example-product:calculator:percent-key#AC-1`
- `req:example-product:calculator:percent-key#AC-2`
- `req:example-product:calculator:percent-key#AC-3`
