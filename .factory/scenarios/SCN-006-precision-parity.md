---
schema: dark-factory.dev/scenario/v1
id: scenario:example-product:calculator:precision-parity
type: scenario
title: Percent output matches existing-operation precision and formatting
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# Percent output matches existing-operation precision and formatting

The user compares the characters the `%` key produces with the characters the existing keys produce
for the same value, on values that fit in the display and on a value that does not.

## Script

1. For each `op ∈ { +, -, ×, ÷ }`: press `C`, `2`, `0`, `0`, `op`, `1`, `0`, `%`; read the display.
   Then press `C`, `2`, `0`, `0`, `×`, `1`, `0`, `÷`, `1`, `0`, `0`, `=`; read the display. Compare
   the two strings character by character.
2. Press `C`, `5`, `0`, `%`; read the display. Press `C`, `1`, `÷`, `2`, `=`; read the display.
   Compare the two strings character by character.
3. Press `C`, `2`, `0`, `0`, `×`, `1`, `0`, `%`; read the display. Press `C`, `2`, `0`, `0`, `÷`,
   `1`, `0`, `=`; read the display. Compare the two strings character by character.
4. Press `C`, then `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `×`, `1`, `2`, `3`, `4`, `5`, `6`,
   `7`, `8`, `9`, `%`; read the display after `%` as `X`. Press `=`. Read the display as `D1`.
   Then press `C`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `×`, then the digits of `X`, `=`;
   read the display as `D2`. Compare `D1` and `D2` character by character.
5. Check the display width of the step 4 result equals the display width the calculator uses for
   other results of the same magnitude (compare with `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`,
   `9`, `×`, `1`, `0`, `0`, `0`, `0`, `0`, `0`, `0`, `0`, `=`).

## Observable outcome

- Steps 1–3: the `%` route and the existing-key route produce identical character strings.
- Step 4: `D1` and `D2` are identical character strings — the operand used by `=` is exactly the
  displayed value of `X`, with no hidden digits.

## Traces

- `req:example-product:calculator:precision-parity#AC-1`
- `req:example-product:calculator:precision-parity#AC-2`
- `req:example-product:calculator:precision-parity#AC-3`
