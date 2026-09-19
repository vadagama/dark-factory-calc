---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:percent-of-pending-addition
type: scenario
title: Percent of the pending operand of an addition and of a subtraction
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Percent of the pending operand of an addition and of a subtraction

The user asks for "200 plus 10 %" and for "200 minus 10 %" without computing the percentage by hand.

## Script

1. Press `C`, `2`, `0`, `0`, `+`, `1`, `0`, `%`. Read the display. Press `=`. Read the display.
2. Press `C`, `2`, `0`, `0`, `-`, `1`, `0`, `%`. Read the display. Press `=`. Read the display.

## Observable outcome

- Step 1: display after `%` reads `20`; display after `=` reads `220`.
- Step 2: display after `%` reads `20`; display after `=` reads `180`.

## Traces

- `req:example-product:calculator:percent-pending-operation#AC-1`
- `req:example-product:calculator:percent-pending-operation#AC-2`
