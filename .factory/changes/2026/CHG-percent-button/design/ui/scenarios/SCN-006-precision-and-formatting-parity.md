---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-006
type: ui_scenario
title: Percent display matches the existing operations character for character
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: "For each of `+`, `-`, `×`, `÷`: press `C`, `2`, `0`, `0`, op, `1`, `0`, `%` and read the display; then press `C`, `2`, `0`, `0`, `×`, `1`, `0`, `÷`, `1`, `0`, `0`, `=` and read the display; compare the two strings character by character (REQ-003 AC-1)."
    screen: SCR-001
  - id: S2
    text: Press `C`, `5`, `0`, `%` and read the display; press `C`, `1`, `÷`, `2`, `=` and read the display; compare the two strings character by character (REQ-003 AC-2).
    screen: SCR-001
  - id: S3
    text: Press `C`, `2`, `0`, `0`, `×`, `1`, `0`, `%` and read the display; press `C`, `2`, `0`, `0`, `÷`, `1`, `0`, `=` and read the display; compare the two strings character by character (REQ-003 AC-2).
    screen: SCR-001
  - id: S4
    text: "Press `C`, `1` … `9`, `×`, `1` … `9`, `%` and read the display as `X`; press `=` and read `R1`; then press `C`, `1` … `9`, `×`, the keys that enter `X` exactly as displayed, `=`, and read `R2`; compare `R1` and `R2` character by character (REQ-003 AC-3)."
    screen: SCR-001
  - id: S5
    text: "If the build renders `X` in a form the keypad cannot re-enter (for example exponent notation), repeat step 4 with the largest operand pair whose `%` display is a plain digit string; `R1` and `R2` must still be character-identical (contingency, intent.md A-2)."
    screen: SCR-001
  - id: S6
    text: Assert no `%` display carries a decimal place, a separator or a notation that the same value does not carry when produced by `+`, `-`, `×`, `÷`.
    screen: SCR-001
---

# SCN-006 — Percent display matches the existing operations character for character

`%` is invisible on the display: the user cannot tell from the characters on `SCR-001` whether a value came from
the new key or from the existing keys. That is the whole of `REQ-003` — one arithmetic path, one formatter, no
percent-specific rounding, truncation or notation (`ADR-003`) — and it is checked by comparing strings, not
numbers, because the requirement is about exactly the characters `EL-display` shows.

Steps 1–3 cover values that fit the display (`20`, `0.5`); step 4 covers a percentage whose exact value exceeds
the display width and is therefore the case where an implementation that carried hidden precision into `=` would
diverge: the operand handed to the pending operation is exactly the value displayed as `X`, so re-entering `X`
and pressing `=` must reproduce `R1` character for character (`REQ-003` AC-3, DEC-2, `ADR-003`). Step 5 is the
contingency the intent already states (assumption A-2) for a formatter that renders `X` in a notation the keypad
cannot re-enter.

Grounded in: `REQ-003` AC-1…AC-3, `intent.md` (goal: same precision as the existing operations; DEC-2; A-2),
`design/overview.md` (reuse of the arithmetic path and the formatter; the committed operand is the displayed
value), `ADR-003`.

Traces: `req:example-product:calculator:precision-parity#AC-1`,
`req:example-product:calculator:precision-parity#AC-2`,
`req:example-product:calculator:precision-parity#AC-3`.
