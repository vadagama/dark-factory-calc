---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:percent-of-pending-multiplication
type: scenario
title: Percent of the pending operand of a multiplication
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Percent of the pending operand of a multiplication

The user wants 200 multiplied by 10 %. Instead of working out `200 × 10 ÷ 100` by hand, they press
`%` and let the calculator do it.

## Script

1. Press `2`, `0`, `0`, `×`, `1`, `0`.
2. Press `%`. Read the display.
3. Press `=`. Read the display.
4. As the reference route, press `C`, then `2`, `0`, `0`, `×`, `1`, `0`, `÷`, `1`, `0`, `0`, `=`.
   Read the display.

## Observable outcome

- After step 2 the display reads `20`; this is character-identical to the reference route of step 4.
- After step 3 the display reads `4000`.

## Traces

- `req:example-product:calculator:percent-pending-operation#AC-3`
- `req:example-product:calculator:precision-parity#AC-1`
