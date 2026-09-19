---
schema: dark-factory.dev/requirement/v1
id: req:example-product:calculator:percent-without-pending-operation
type: requirement
title: Percent with no pending operation or no second operand
product: example-product
status: draft
change: chg:example-product:2026:0002
---

# REQ-004 — Percent with no pending operation or no second operand

When `%` is pressed and there is no pending binary operation, the current entry `b` is divided by 100
and the result is displayed (`p = b ÷ 100`). When `%` is pressed with a pending operation whose second
operand has not been entered, `%` is a no-op: the display and the pending operation are unchanged.
`%` never produces an error state and never leaves the display blank.

## Acceptance criteria

- **AC-1** With no pending operation, `50 %` displays `0.5`.
  Check: press `5`, `0`, `%`; display reads `0.5`.
- **AC-2** With a pending operation and no second operand, `%` changes nothing: after
  `200 +` `%` the display still reads `200`, and a following `=` displays the same characters as
  `200 + =` displays without the `%` press (checked against the same build).
- **AC-3** `%` on a displayed result behaves like AC-1: `200 + 10 =` displays `210`, a following `%`
  displays `2.1`.
- **AC-4** `%` in the initial state is not an error: directly after a reset (`C`), the display reads
  `0`, and pressing `%` leaves the display reading `0` with no error indicator and no blank display.
  Check: reset, press `%`, compare the display characters with the pre-change build's reset display.

## Decision

AC-1, AC-2 and AC-4 encode decision **DEC-3** (`intent.md`): the operator's answer A to
`q_959333f8266c7fa5` — with no pending operation, divide the entry by 100; with a pending operation
whose second operand has not been entered, do nothing. AC-3 records the point made in that answer,
that a result displayed after `=` is the same case as a plain entry. No open question remains for
this requirement.

## Traceability

| Criterion | Scenario |
| --- | --- |
| AC-1 | `scenario:example-product:calculator:percent-without-pending-operation` |
| AC-2 | `scenario:example-product:calculator:percent-without-pending-operation` |
| AC-3 | `scenario:example-product:calculator:percent-without-pending-operation` |
| AC-4 | `scenario:example-product:calculator:percent-without-pending-operation` |
