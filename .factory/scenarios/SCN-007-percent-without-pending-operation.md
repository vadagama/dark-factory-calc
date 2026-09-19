---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:percent-without-pending-operation
type: scenario
title: Percent with no pending operation or no second operand
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Percent with no pending operation or no second operand

The user presses `%` when there is nothing to take a percentage of: on a plain entry, on a displayed
result, right after a reset, and with a pending operation whose second operand is still missing.

## Script

1. Press `C`, `5`, `0`, `%`. Read the display.
2. Press `C`, `2`, `0`, `0`, `+`, `%`. Read the display. Press `=`. Read the display.
3. Press `C`, `2`, `0`, `0`, `+`, `1`, `0`, `=`. Read the display. Press `%`. Read the display.
4. Press `C`. Read the display. Press `%`. Read the display.
5. On the pre-change build, press `C`, `2`, `0`, `0`, `+`, `=`. Read the display. Compare with the
   step 2 display after `=`.

## Observable outcome

- Step 1: display reads `0.5`.
- Step 2: display after `%` still reads `200`; the display after `=` equals the pre-change display for
  `200 + =`.
- Step 3: `210`, then `2.1`.
- Step 4: `0`, then `0` — no error indicator, no blank display.

## Traces

- `req:example-product:calculator:percent-without-pending-operation#AC-1`
- `req:example-product:calculator:percent-without-pending-operation#AC-2`
- `req:example-product:calculator:percent-without-pending-operation#AC-3`
- `req:example-product:calculator:percent-without-pending-operation#AC-4`
