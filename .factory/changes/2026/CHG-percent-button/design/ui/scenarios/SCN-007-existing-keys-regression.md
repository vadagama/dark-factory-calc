---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-007
type: ui_scenario
title: A user who never presses percent sees the same calculator
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: "On the pre-change build, run the baseline sequences `12 + 34 =`, `5 × 6 =`, `7 ÷ 2 =`, `9 - 4 =`, `C` then `12 + 3 =`, `1 ÷ 3 =`, `0 + 0 =` and record every display string."
    screen: SCR-001
  - id: S2
    text: Run the same sequences on the changed build and record every display string.
    screen: SCR-001
  - id: S3
    text: Compare the recorded strings pairwise, character by character; assert they are identical (REQ-001 AC-4).
    screen: SCR-001
  - id: S4
    text: Assert no sequence produces an error indicator or a blank display, and that the keypad layout the sequences were pressed on is the only visible difference (one more key, labelled `%`).
    screen: SCR-001
---

# SCN-007 — A user who never presses percent sees the same calculator

Regression safety on the screen: a user who never presses `%` must not notice that the keypad changed
(`REQ-001` AC-4). The check is deliberately a comparison of display *strings*, not of numeric values, because
that is the observable the user has — `12 + 34 =` must show the same characters it showed before the change, on
the same screen, reached through the same keys.

The sequences cover the four binary operations, the reset key and a repeating fraction, so a behaviour change in
evaluation, formatting or state handling on the pre-existing path would surface as a different string. The only
permitted difference on `SCR-001` is the presence of one additional key, labelled `%` (`SCN-002`), which this
scenario's sequences never press.

Grounded in: `REQ-001` AC-4, the constraint in `intent.md` (every pre-existing key keeps its label, its function
and its relative order), `design/overview.md` ("nothing in the requirements forces a change to the evaluation
core's public command semantics"), `ADR-004` point 4 (exhaustive freeze list) — `SCN-008` of the specification
scenarios is the source of the baseline suite.

Traces: `req:example-product:calculator:percent-key#AC-4`.
