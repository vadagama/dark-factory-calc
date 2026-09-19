---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-005
type: ui_scenario
title: Repeated percent presses and zero operands
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: Press `C`, `2`, `0`, `0`, `+`, `1`, `0`, `%`; the display shows `20`.
    screen: SCR-001
  - id: S2
    text: Press `%` again; the display shows `40` — the rule is applied once more to the operand that is current, `200 × 20 ÷ 100` — and not `0.2` (REQ-002 AC-5).
    screen: SCR-001
  - id: S3
    text: Press `=`; the display shows `240` — the pending addition and the first operand survived both presses (REQ-002 AC-5).
    screen: SCR-001
  - id: S4
    text: Press `C`, `2`, `0`, `0`, `+`, `0`, `%`; the display shows `0`; press `=`; the display shows `200` (REQ-002 AC-6).
    screen: SCR-001
  - id: S5
    text: Press `C`, `0`, `+`, `1`, `0`, `%`; the display shows `0`; press `=`; the display shows `0` (REQ-002 AC-6).
    screen: SCR-001
  - id: S6
    text: Assert no error indicator appears and the display never blanks in any of the steps; the keypad stays fully operable.
    screen: SCR-001
---

# SCN-005 — Repeated percent presses and zero operands

Two behaviours meet on the same screen here: `%` is repeatable, and it degrades gracefully on zero. Repeated
presses are the observable proof that `%` does not commit the pending operation — if it did, the second press
would act on a committed `20` and show `0.2` instead of `40` (`REQ-002` AC-5). Zero operands are the observable
proof that `%` never reaches an error state: `200 + 0 % =` and `0 + 10 % =` both pass through the same rule and
show ordinary values (`REQ-002` AC-6).

Neither case changes the screen: the display shows the value the rule produces, the keypad stays exactly as it
was, and `EL-display` never shows an error indicator and never blanks — the percent command's contract in
`design/overview.md` is that it never errors and never blanks.

Grounded in: `intent.md` (rules 3 and 1, assumption A-1, DEC-1), `design/overview.md` (each press applies the
rule once to the operand current at that moment), `ADR-002`; `REQ-002` AC-5, AC-6.

Traces: `req:example-product:calculator:percent-pending-operation#AC-5`,
`req:example-product:calculator:percent-pending-operation#AC-6`.
