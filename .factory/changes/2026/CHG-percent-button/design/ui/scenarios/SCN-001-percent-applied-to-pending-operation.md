---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-001
type: ui_scenario
title: Percent applied to a pending operation and completed with =
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: Open the calculator; the panel shows the keypad grid with exactly one key labelled `%` and the display in its reset state.
    screen: SCR-001
  - id: S2
    text: Press `2`, `0`, `0`, `+`, `1`, `0`; the display shows the current operand `10` while the addition and the first operand `200` are pending.
    screen: SCR-001
  - id: S3
    text: Press `%`; the display shows `20` — the percentage `200 × 10 ÷ 100` — and the pending `+` with the first operand `200` are retained (REQ-002 AC-1, REQ-003 AC-1).
    screen: SCR-001
  - id: S4
    text: Press `=`; the display shows `220` (REQ-002 AC-1).
    screen: SCR-001
  - id: S5
    text: Press `C`, `2`, `0`, `0`, `-`, `1`, `0`, `%`; the display shows `20`.
    screen: SCR-001
  - id: S6
    text: Press `=`; the display shows `180` (REQ-002 AC-2).
    screen: SCR-001
  - id: S7
    text: Press `C`, `2`, `0`, `0`, `×`, `1`, `0`, `%`; the display shows `20` (REQ-002 AC-3).
    screen: SCR-001
  - id: S8
    text: Press `=`; the display shows `4000` (REQ-002 AC-3).
    screen: SCR-001
  - id: S9
    text: Press `C`, `2`, `0`, `0`, `÷`, `1`, `0`, `%`; the display shows `20`.
    screen: SCR-001
  - id: S10
    text: Press `=`; the display shows `10` (REQ-002 AC-4).
    screen: SCR-001
---

# SCN-001 — Percent applied to a pending operation and completed with =

The primary flow of the change: the user asks the calculator for a percentage of a pending operand — "200 plus
10 %", "200 minus 10 %", "200 times 10 %", "200 divided by 10 %" — and never leaves the screen to work the
percentage out by hand. Everything happens on `SCR-001`: digits and the operator go into the keypad, `%` rewrites
the operand that is current, and `=` finishes the pending operation through the unchanged evaluation path.

The scenario is one flow because all four binary operations share it: `%` is an operand transform, not a fifth
operation, so the *same* key press behaves identically under `+`, `-`, `×` and `÷` and differs only in the value
`=` finally shows (`REQ-002` AC-1…AC-4). The user never sees an operator-specific percent affordance, a dialog or
a mode.

Observable outcome: after `%` the display is character-identical to the display the existing-key route
`a × b ÷ 100 =` produces (`REQ-003` AC-1); after `=` it shows `220`, `180`, `4000` and `10` respectively, with
the pending operation and the first operand having survived the `%` press.

Grounded in: `intent.md` (goal, the percent rule, DEC-1), `design/overview.md` (percent as an operand transform
that retains `a` and `op`), `ADR-002`, `ADR-003`; `REQ-002` AC-1…AC-4 and `REQ-003` AC-1.

Traces: `req:example-product:calculator:percent-pending-operation#AC-1`,
`req:example-product:calculator:percent-pending-operation#AC-2`,
`req:example-product:calculator:percent-pending-operation#AC-3`,
`req:example-product:calculator:percent-pending-operation#AC-4`,
`req:example-product:calculator:precision-parity#AC-1`.
