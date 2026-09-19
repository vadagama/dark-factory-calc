---
schema: dark-factory.dev/change/v1
id: chg:example-product:2026:0002
type: change
title: Percent button for the calculator (M3)
product: example-product
status: draft
change: chg:example-product:2026:0002
slug: percent-button
stage: specification
---

# Percent button for the calculator (M3) — intent

## Problem

The calculator has no percent operation. To answer a question of the form "what is 10 % of 200" or
"200 plus 10 %" the user has to leave the calculator, work out `200 × 10 ÷ 100` by hand and type the
result back in as a number. This is error-prone and it defeats the purpose of the calculator, which
is to perform the arithmetic for the user.

Evidence that the problem exists is observable in the current build: no keypad key is labelled `%`
and no input sequence produces a percentage; the only way to obtain a percentage is the manual
three-operation detour above.

## Goal

A `%` key that applies a percentage to the current operand, with the same precision as the
calculator's existing operations (`+`, `-`, `×`, `÷`).

## Constraint

No existing key may be removed, renamed or reordered. `%` is an addition to the existing keypad:
every pre-existing key keeps its label, its function and its relative order on the keypad.

## Scope (in scope)

1. A `%` key rendered as part of the existing calculator keypad.
2. Percent applied to the current operand while a binary operation (`+`, `-`, `×`, `÷`) is pending.
3. Percent applied to an entry when no binary operation is pending.
4. Precision, rounding and number formatting of percent results identical to those of the existing
   binary operations.
5. Regression safety: input sequences that do not use `%` behave exactly as before the change.

## Out of scope

- Scientific functions (trigonometry, logarithms, powers, roots, constants, any operation taking
  more than two operands, and any expression-parsing mode).
- New number formatting, new rounding rules, or a different display width.
- Any change to the behaviour of the pre-existing keys `+`, `-`, `×`, `÷`, `=`, `C`, `←`/`⌫` and the
  digit keys.
- Keyboard/OS-level input mapping for the `%` character, and localization of the key label.
- Memory keys, history tape, unit conversion and any other calculator feature.

## Working definition of the percent rule (assumed in this specification)

With a pending binary operation `op ∈ { +, -, ×, ÷ }`, an accepted first operand `a` and a current
(not yet committed) operand `b`:

1. Pressing `%` replaces the current operand with `p = a × b ÷ 100` and displays `p`.
2. The pending operation and the first operand `a` are retained; the following `=` computes `a op p`.
3. Every press of `%` applies rule 1 once, to the operand that is current at that moment.

Consequences used as acceptance criteria (see `spec/requirements/REQ-002-percent-pending-operation.md`):

| Keystrokes | Display after `%` | Display after `=` |
| --- | --- | --- |
| `200 + 10 %` | `20` | `220` |
| `200 - 10 %` | `20` | `180` |
| `200 × 10 %` | `20` | `4000` |
| `200 ÷ 10 %` | `20` | `10` |
| `200 + 10 % %` | `20`, then `40` | `240` |

Rules 1–3 are the subject of the open questions Q1–Q3 listed below; the criteria that depend on
them are anchored so that a later round can amend them in place.

## Assumptions and open questions

- **A-1 (derived, not an assumption about intent).** Because rule 3 re-applies rule 1 to the current
  operand, a second `%` press is well defined: `200 + 10 %` → `20`, `%` again → `200 × 20 ÷ 100 = 40`.
  This is checked by `REQ-002#AC-5`.
- **Q1 (blocking).** Which percent semantics apply while a binary operation is pending — percentage
  of the pending operand (`200 + 10 % = 220`) or a plain divide-by-100 of the current entry
  (`200 + 10 % = 200.1`)? Anchor: `spec/requirements/REQ-002-percent-pending-operation.md#AC-1`.
- **Q2 (blocking).** Is the value handed to the pending operation the value as displayed after `%`
  (display-rounded), or the exact product `a × b ÷ 100` at full internal precision (which can differ
  once the result exceeds the display width)? Anchor:
  `spec/requirements/REQ-003-precision-parity.md#AC-3`.
- **Q3 (blocking).** What must `%` do when it has no pending operand to take a percentage of —
  i.e. with no pending operation (`50 %`) and with a pending operation whose second operand has not
  been entered (`200 + %`)? Anchor:
  `spec/requirements/REQ-004-percent-without-pending-operation.md#AC-1`.

## Artifacts of this ChangeSet

| Path | Content |
| --- | --- |
| `intent.md` | this file: problem, goal, constraint, scope, out of scope, assumptions |
| `spec/delta.yaml` | change operations, artifact index, criterion → scenario traceability, open questions |
| `spec/requirements/REQ-001-percent-key.md` | the key exists; existing keys unchanged |
| `spec/requirements/REQ-002-percent-pending-operation.md` | percent relative to the pending operation |
| `spec/requirements/REQ-003-precision-parity.md` | precision, rounding, formatting parity |
| `spec/requirements/REQ-004-percent-without-pending-operation.md` | `%` with no pending operation / no second operand |
| `.factory/scenarios/SCN-001..SCN-008-*.md` | scenarios referenced by the traceability matrix |
