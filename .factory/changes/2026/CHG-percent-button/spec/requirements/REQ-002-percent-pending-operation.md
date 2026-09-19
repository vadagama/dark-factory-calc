---
schema: dark-factory.dev/requirement/v1
id: req:example-product:calculator:percent-pending-operation
type: requirement
title: Percent applied to the current operand of a pending operation
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# REQ-002 — Percent applied to the current operand of a pending operation

With a binary operation `op ∈ { +, -, ×, ÷ }` pending, an accepted first operand `a` and a current
(not yet committed) operand `b`, pressing `%` replaces the current operand with `p = a × b ÷ 100` and
displays `p`. The pending operation and the first operand `a` are retained, so the following `=`
computes `a op p`. Each press of `%` applies this rule once to the operand current at that moment.

## Acceptance criteria

- **AC-1** `200 + 10 %` displays `20`; a following `=` displays `220`.
  Check: press the keys in order, compare the display character by character after `%` and after `=`.
- **AC-2** `200 - 10 %` displays `20`; a following `=` displays `180`.
- **AC-3** `200 × 10 %` displays `20`; a following `=` displays `4000`.
- **AC-4** `200 ÷ 10 %` displays `20`; a following `=` displays `10`.
- **AC-5** The pending operation and the first operand survive `%`: `200 + 10 % %` displays `20` after
  the first `%` and `40` after the second, and a following `=` displays `240`.
  Check: the second display `40` equals `200 × 20 ÷ 100`; if `%` had committed the operation, the
  second press would have displayed `0.2` (20 ÷ 100) instead of `40`. This criterion fails on any
  implementation where `%` commits the pending operation.
- **AC-6** Degenerate operands follow the same rule and raise no error state: `200 + 0 %` displays
  `0` and a following `=` displays `200`; `0 + 10 %` displays `0` and a following `=` displays `0`.

## Depends on

- Open question Q1 (`intent.md`), which fixes the semantics encoded in AC-1 … AC-6: percentage of the
  pending operand rather than a plain divide-by-100 of the current entry. AC-5 distinguishes the two
  readings independently of Q1.

## Traceability

| Criterion | Scenario |
| --- | --- |
| AC-1 | `scenario:example-product:calculator:percent-of-pending-addition` |
| AC-2 | `scenario:example-product:calculator:percent-of-pending-addition` |
| AC-3 | `scenario:example-product:calculator:percent-of-pending-multiplication` |
| AC-4 | `scenario:example-product:calculator:percent-of-pending-division` |
| AC-5 | `scenario:example-product:calculator:repeated-percent-and-zero-operands` |
| AC-6 | `scenario:example-product:calculator:repeated-percent-and-zero-operands` |
