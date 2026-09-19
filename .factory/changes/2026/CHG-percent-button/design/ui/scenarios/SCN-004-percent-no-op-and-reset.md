---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-004
type: ui_scenario
title: Percent with a missing second operand and directly after a reset
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: Press `C`, `2`, `0`, `0`, `+`; the display shows the first operand `200` with an addition pending and no second operand entered.
    screen: SCR-001
  - id: S2
    text: Press `%`; the display still shows `200` and the pending addition is unchanged — `%` is a no-op in this state (REQ-004 AC-2).
    screen: SCR-001
  - id: S3
    text: Press `=`; the display shows the same characters the pre-change build shows for `C`, `2`, `0`, `0`, `+`, `=` without the `%` press (REQ-004 AC-2).
    screen: SCR-001
  - id: S4
    text: Press `C`; the display shows `0` in the reset state (REQ-004 AC-4).
    screen: SCR-001
  - id: S5
    text: Press `%`; the display still shows `0`, with no error indicator and no blank display (REQ-004 AC-4).
    screen: SCR-001
---

# SCN-004 — Percent with a missing second operand and directly after a reset

The two states in which `%` has nothing legitimate to transform are handled on the screen as a non-event: the
display does not change and nothing appears next to it. With a pending operation whose second operand has not
been entered, `%` is a no-op (`REQ-004` AC-2, DEC-3), so the key stays an ordinary enabled `Button` rather than
becoming disabled or entering a mode; the user can press it at any time without the keypad's availability
depending on core state. Right after `C`, where there is nothing to take a percentage of, `%` leaves the reset
display `0` untouched and raises no error state (`REQ-004` AC-4) — the percent command has no error branch at all
(`design/overview.md`).

Step 3 is the regression guard for the no-op case: `200 +` followed by `%` and `=` must land on exactly the
display the pre-change build produces for `200 + =`, character for character, because the `%` press contributed
nothing.

Grounded in: `intent.md` (rule 5 and DEC-3), `design/overview.md` (`percent` never errors, never blanks; no-op
when the second operand is missing), `ADR-002`; `REQ-004` AC-2 and AC-4.

Traces: `req:example-product:calculator:percent-without-pending-operation#AC-2`,
`req:example-product:calculator:percent-without-pending-operation#AC-4`.
