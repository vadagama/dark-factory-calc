---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:repeated-percent-and-zero-operands
type: scenario
title: Repeated percent presses and zero operands
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Repeated percent presses and zero operands

The user presses `%` twice in a row on the same pending operation, and presses `%` with a zero
operand.

## Script

1. Press `C`, `2`, `0`, `0`, `+`, `1`, `0`, `%`. Read the display. Press `%` again. Read the display.
   Press `=`. Read the display.
2. Press `C`, `2`, `0`, `0`, `+`, `0`, `%`. Read the display. Press `=`. Read the display.
3. Press `C`, `0`, `+`, `1`, `0`, `%`. Read the display. Press `=`. Read the display.

## Observable outcome

- Step 1: `20`, then `40` after the second press (and not `0.2`, which is what a committed operation
  with a follow-up divide-by-100 would show); `=` reads `240`.
- Step 2: `0`; `=` reads `200`.
- Step 3: `0`; `=` reads `0`.
- No error indicator appears in any step.

## Traces

- `req:example-product:calculator:percent-pending-operation#AC-5`
- `req:example-product:calculator:percent-pending-operation#AC-6`
