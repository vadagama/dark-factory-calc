---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:percent-of-pending-division
type: scenario
title: Percent of the pending operand of a division
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Percent of the pending operand of a division

The user wants 200 divided by 10 %, i.e. by one tenth of the pending operand.

## Script

1. Press `2`, `0`, `0`, `÷`, `1`, `0`, `%`. Read the display.
2. Press `=`. Read the display.

## Observable outcome

- After step 1 the display reads `20`.
- After step 2 the display reads `10`.

## Traces

- `req:example-product:calculator:percent-pending-operation#AC-4`
