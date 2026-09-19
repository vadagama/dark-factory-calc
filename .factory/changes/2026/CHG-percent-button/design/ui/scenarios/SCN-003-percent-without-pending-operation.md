---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-003
type: ui_scenario
title: Percent of a plain entry and of a displayed result
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: Press `C`, `5`, `0`, `%`; the display shows `0.5` (REQ-004 AC-1).
    screen: SCR-001
  - id: S2
    text: Compare the characters shown with those the same build shows for the existing-key route `C`, `1`, `÷`, `2`, `=` — `0.5` (REQ-003 AC-2).
    screen: SCR-001
  - id: S3
    text: Press `C`, `2`, `0`, `0`, `+`, `1`, `0`, `=`; the display shows `210`.
    screen: SCR-001
  - id: S4
    text: Press `%`; the display shows `2.1` — the displayed result is treated as a plain entry, so it is divided by 100 (REQ-004 AC-3).
    screen: SCR-001
  - id: S5
    text: Assert the display has not blanked, shows no error indicator and that the keypad, including the `%` key, remains enabled.
    screen: SCR-001
---

# SCN-003 — Percent of a plain entry and of a displayed result

The other half of the percent rule: when there is no pending operation, `%` divides what is on the display by 100
and shows the result (`REQ-004` AC-1). This is the "what is 10 % of 200" shape of the user's question — a plain
entry, no operator involved — and it is also the case a value shown after `=` falls into, because the operator's
answer DEC-3 puts a displayed result on the same footing as a typed entry (`REQ-004` AC-3).

Nothing about this path is percent-specific in the presentation: the value produced by `%` is formatted by the
same formatter that produced `1 ÷ 2 =`, so `0.5` after `50 %` and `0.5` after `1 ÷ 2 =` are character-identical
(`REQ-003` AC-2). No exponent notation, no extra decimal place and no thousands separator appears that the same
value does not already produce through `+`, `-`, `×`, `÷`.

The screen consequence of step 4 is deliberately unremarkable: the display simply shows the new value, the
pending-operation state is empty before and after, and nothing else on `SCR-001` changes.

Grounded in: `intent.md` (rule 4, DEC-3), `design/overview.md` (`b ÷ 100` otherwise; the single formatter),
`ADR-002`, `ADR-003`; `REQ-004` AC-1 and AC-3, `REQ-003` AC-2.

Traces: `req:example-product:calculator:percent-without-pending-operation#AC-1`,
`req:example-product:calculator:percent-without-pending-operation#AC-3`,
`req:example-product:calculator:precision-parity#AC-2`.
