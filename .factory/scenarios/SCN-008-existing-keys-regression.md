---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:existing-keys-regression
type: scenario
title: Existing key sequences behave exactly as before
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Existing key sequences behave exactly as before

A user who never presses `%` must not notice that the keypad changed.

## Script

1. On the pre-change build run the baseline suite and record every display string:
   `12 + 34 =`, `5 × 6 =`, `7 ÷ 2 =`, `9 - 4 =`, `C` then `12 + 3 =`, `1 ÷ 3 =`, `0 + 0 =`.
2. Run the same sequences on the changed build and record every display string.
3. Compare the strings pairwise, character by character.

## Observable outcome

Every baseline display string is character-identical between the two builds; no sequence produces a
different digit string, an error state or a blank display.

## Traces

- `req:example-product:calculator:percent-key#AC-4`
