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

## Percent rule (fixed in this round)

With a pending binary operation `op ∈ { +, -, ×, ÷ }`, an accepted first operand `a` and a current
(not yet committed) operand `b`:

1. Pressing `%` replaces the current operand with `p = a × b ÷ 100` and displays `p`.
2. The pending operation and the first operand `a` are retained; the following `=` computes `a op p`.
3. Every press of `%` applies rule 1 once, to the operand that is current at that moment.
4. With no pending operation, `%` replaces the current entry `b` with `b ÷ 100` and displays it.
5. With a pending operation whose second operand has not been entered, `%` is a no-op.

Rules 1–3 and 4–5 are the operator's answers DEC-1 and DEC-3 below; they are decisions, not
assumptions. Consequences used as acceptance criteria:

| Keystrokes | Display after `%` | Display after `=` |
| --- | --- | --- |
| `200 + 10 %` | `20` | `220` |
| `200 - 10 %` | `20` | `180` |
| `200 × 10 %` | `20` | `4000` |
| `200 ÷ 10 %` | `20` | `10` |
| `200 + 10 % %` | `20`, then `40` | `240` |
| `50 %` | `0.5` | — |
| `200 + %` | `200` (unchanged) | — |

## Decisions taken by the operator in this round

All three questions raised in the previous round are answered. No question of this ChangeSet is left
open.

| Decision | Operator answer | Rule now fixed | Criteria that encode it |
| --- | --- | --- | --- |
| **DEC-1** `q_c105c77c6efdb5e2` | A | While a binary operation is pending, `%` applies the percentage to the pending operand and the operation is retained: `200 + 10 % =` gives `220`, `200 × 10 % =` gives `4000`. `%` is not a plain divide-by-100 of the current entry and it does not commit the operation. | `REQ-002` AC-1 … AC-6 |
| **DEC-2** `q_c88d93fd2bb49df4` | A | The operand handed to the pending operation is exactly the value displayed after `%` (the display-rounded value); no digits are held beyond the display. | `REQ-003` AC-3 |
| **DEC-3** `q_959333f8266c7fa5` | A | With no pending operation, `%` divides the current entry by 100 (`50 %` → `0.5`; a result displayed after `=` is the same case). With a pending operation and no second operand, `%` is a no-op. | `REQ-004` AC-1, AC-2, AC-4 |

## Assumptions

- **A-1 (derived, not an assumption about intent).** Because rule 3 re-applies rule 1 to the operand
  that is current, a second `%` press is well defined: `200 + 10 %` → `20`, `%` again →
  `200 × 20 ÷ 100 = 40`, then `=` → `240`. This is checked by `REQ-002#AC-5`.
- **A-2 (display contingency).** The checks that compare a committed operand with the digits shown
  after `%` assume those digits are re-enterable on the keypad. If the build's formatter renders such
  a value in a notation the keypad cannot re-enter (e.g. exponent notation), the check falls back to
  the largest operand pair whose `%` display is a plain digit string. Encoded in `REQ-003` AC-3.
- No other behaviour is assumed. Every rule used by an acceptance criterion is fixed by DEC-1 … DEC-3
  or by the parity clauses of `REQ-003`. No open question remains in this ChangeSet.

## Artifacts of this ChangeSet

| Path | Content |
| --- | --- |
| `intent.md` | this file: problem, goal, constraint, scope, out of scope, decisions |
| `spec/delta.yaml` | change operations, artifact index, criterion → scenario traceability |
| `spec/requirements/REQ-001-percent-key.md` | the key exists; existing keys unchanged |
| `spec/requirements/REQ-002-percent-pending-operation.md` | percent relative to the pending operation |
| `spec/requirements/REQ-003-precision-parity.md` | precision, rounding, formatting parity |
| `spec/requirements/REQ-004-percent-without-pending-operation.md` | `%` with no pending operation / no second operand |
| `.factory/scenarios/SCN-001..SCN-008-*.md` | scenarios referenced by the traceability matrix |
