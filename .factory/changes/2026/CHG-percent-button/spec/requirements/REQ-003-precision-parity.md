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
  no digits held beyond the display: `a op b % =` and `a op <the digits displayed after %> =` display
  the same characters, including for results whose exact value exceeds the display width.
  Check: with `a = 123456789` and `b = 123456789` and `op = ×`, run `123456789 × 123456789 % =`; then
  run `123456789 × X =`, where `X` is the digit string displayed after `%`; the two displays must be
  character-identical. This criterion is the direct encoding of open question Q2 and it fails on an
  implementation that carries hidden internal precision into `=`.

## Depends on

- Open question Q2 (`intent.md`): AC-3 encodes "committed operand = displayed value". If the operator
  chooses full internal precision, AC-3 must be rewritten in place (id `AC-3` is kept) to require
  that `123456789 × 123456789 % =` differ from `123456789 × X =` whenever the exact product
  `a × b ÷ 100` is not exactly representable in the display.

## Traceability

| Criterion | Scenario |
| --- | --- |
| AC-1 | `scenario:example-product:calculator:percent-of-pending-multiplication`, `scenario:example-product:calculator:precision-parity` |
| AC-2 | `scenario:example-product:calculator:precision-parity` |
| AC-3 | `scenario:example-product:calculator:precision-parity` |
