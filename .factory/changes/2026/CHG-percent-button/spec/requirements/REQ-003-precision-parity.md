---
schema: dark-factory.dev/requirement/v1
id: req:example-product:calculator:precision-parity
type: requirement
title: Percent results use the calculator's existing precision and formatting
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# REQ-003 — Percent results use the calculator's existing precision and formatting

The value that `%` computes and the way it is displayed are the same as the value and the display the
calculator already produces for the equivalent sequence of existing keys. `%` introduces no new
rounding, no truncation and no formatter behaviour of its own.

Reference implementation of the rule in existing keys: `p = a × b ÷ 100` is exactly the sequence
`a × b ÷ 100 =` evaluated left to right.

## Acceptance criteria

- **AC-1** For a pending `op` and operands `a`, `b`, the display immediately after `%` is
  character-identical to the display produced by the existing-key sequence `a × b ÷ 100 =` for the
  same `a` and `b`.
  Check: for each of `op ∈ { +, -, ×, ÷ }` with `a = 200`, `b = 10`, both routes must display the
  same characters (`20`). Any difference is a defect in `%`.
- **AC-2** `%` uses the calculator's existing number formatter and adds no formatting rule: whenever
  the value of `p` equals the value of an existing-operation result, both are displayed with the same
  characters.
  Check: `50 %` (`0.5`) displays the same characters as `1 ÷ 2 =`; `200 × 10 %` (`20`) displays the
  same characters as `200 ÷ 10 =`; no additional decimal places, thousands separators or exponent
  notation appear at `%` that do not appear for the same value produced by `+`, `-`, `×`, `÷`.
- **AC-3** The operand handed to the pending operation is exactly the value displayed after `%`, with
  no digits held beyond the display: `a op b % =` and `a op <the value displayed after `%`,
  re-entered as shown> =` display the same characters, including for percentages whose exact value
  exceeds the display width.
  Check: press `C`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `×`, `1`, `2`, `3`, `4`, `5`, `6`,
  `7`, `8`, `9`, `%` and read the display as `X` — the exact percentage
  `123456789 × 123456789 ÷ 100 = 152415787501905.21` has more significant digits than a plain-digit
  display shows, so the tail digit of `X` is where an implementation that commits the unrounded
  product diverges. Press `=` and read `R1`. Then press `C`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`,
  `9`, `×`, the keys that enter `X` exactly as displayed, `=`, and read `R2`. Assert `R1` and `R2` are
  character-identical. If the build renders `X` in a form the keypad cannot re-enter (for example
  exponent notation), repeat the check with the largest operand pair whose `%` display is a plain
  digit string; `R1` and `R2` must still be character-identical.
  This criterion is the direct encoding of decision DEC-2 and it fails on an implementation that
  carries hidden internal precision into `=`.

## Decision

AC-3 encodes decision **DEC-2** (`intent.md`): the operator's answer A to `q_c88d93fd2bb49df4` — the
displayed (rounded) value is committed. If a later round chooses full internal precision instead,
AC-3 is rewritten in place — the id `AC-3` is kept — so that it instead requires `R1` and `R2` to
differ whenever `a × b ÷ 100` is not exactly representable on the display. No open question remains
for this requirement.

## Traceability

| Criterion | Scenario |
| --- | --- |
| AC-1 | `scenario:example-product:calculator:percent-of-pending-multiplication`, `scenario:example-product:calculator:precision-parity` |
| AC-2 | `scenario:example-product:calculator:precision-parity` |
| AC-3 | `scenario:example-product:calculator:precision-parity` |
