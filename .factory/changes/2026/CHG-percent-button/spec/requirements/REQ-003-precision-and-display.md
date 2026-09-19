---
schema: dark-factory.dev/requirement/v1
id: req:example-product:percent-button:precision-and-display
type: requirement
title: Percent results keep the precision and formatting of other operations
product: example-product
status: draft
change: chg:example-product:2026:percent-button
---

# REQ-003 — Percent results keep the precision and formatting of other operations

The goal states "the same precision as the other operations". This requirement
makes that checkable by comparison rather than by naming a digit count: the
percent transform must introduce no rounding of its own, and the strings it
produces must come from the calculator's existing formatting rules.

## Acceptance criteria

### AC-1 — No intermediate rounding of the percentage value

Statement: The percentage computed by a `%` press is not rounded to the
display's precision before the pending operation is applied. Sequence
`1 0 0 ÷ 3 % =` displays exactly the same string as the sequence
`1 0 0 ÷ 0 . 0 3 =`, i.e. a value with the same number of significant digits as
the calculator produces for the equivalent expression.

Verification: run both sequences on the same build and assert string equality
of the final display.

Traceability: SCN `precision-parity`.

### AC-2 — Formatting parity with the other operations

Statement: The strings displayed after a `%` press and after the following `=`
obey exactly the formatting rules the calculator applies to other operations —
same decimal separator, same thousands grouping, same minus sign, same
overflow/rounding behaviour. Sequence `2 0 0 × 1 0 %` shows `0.1` (not `0.10`
and not `0,1` unless the existing rule for other operations is `0,1`), and the
following `=` shows `20`.

Verification: run the sequence, assert the two display strings, and compare
their formatting with the formatting the same build produces for the
equivalent non-percent operands.

Traceability: SCN `precision-parity`.

### AC-3 — No global precision change

Statement: Pressing `%` does not change the calculator's number formatting or
precision settings for later operations. The display of the control sequence
`2 ÷ 3 =` is byte-identical when run before a percent sequence and when run
after it in the same session.

Verification: run `2 ÷ 3 =`, record the string; run a percent sequence
(`2 0 0 + 1 0 % =`); run `2 ÷ 3 =` again; assert string equality.

Traceability: SCN `precision-parity`.

### AC-4 — Percentages below the display precision are not collapsed

Statement: A percentage whose ratio is smaller than the display's decimal
precision is not rounded to zero or to the display precision before use.
Sequence `2 0 0 × 0 . 5 % =` displays `1` (a premature rounding to two decimals
would yield `2`; a rounding to zero would yield `0`).

Verification: run the sequence and assert the display string is `1`.

Traceability: SCN `precision-parity`.
