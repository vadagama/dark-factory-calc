---
schema: dark-factory.dev/change/v1
id: chg:example-product:2026:percent-button
type: change
title: Percent button for the calculator
product: example-product
status: draft
change: chg:example-product:2026:percent-button
---

# Intent — Percent button for the calculator

## Problem

The calculator has no percent operation. A user who needs "10 % of 200" or
"200 + 10 %" has to compute the percentage by hand (or in a second tool) and
type the result back in as a plain operand. This is slow, error-prone, and the
hand-computed value is typed at whatever precision the user chose, which is
not necessarily the precision the calculator itself would have produced.

## Goal

The calculator gains a `%` key that applies a percentage to the current
operand and completes with the existing arithmetic, at the same precision and
with the same display formatting as the other operations, without changing the
existing keyboard layout and without adding a runtime dependency.

"Same precision as the other operations" is read as: the percent transform
introduces no rounding of its own — the value that reaches the pending
operation is the value the calculator's own arithmetic produces for the
equivalent expression, and the formatted output obeys the existing formatting
rules (see REQ-003).

## Scope

### In scope

- A `%` key on the existing calculator keypad (REQ-001).
- The percent semantics for the four binary operations and for a bare entry
  (REQ-002).
- Display/precision parity with the other operations (REQ-003).
- Zero, negative, empty-operand, post-`=` and error parity (REQ-004).
- Verification that the constraints hold: existing layout preserved, no new
  runtime dependency, no regression of existing operations (REQ-005).

### Out of scope

- Scientific mode: no scientific functions, no scientific keypad, no mode
  switch, no memory keys. The keypad label set may grow by exactly one label,
  `%` (REQ-005#AC-1).
- Any change to the layout of pre-existing keys, their labels, or their
  ordering (REQ-001#AC-2, REQ-001#AC-3).
- Any new runtime dependency, new asset, or new network access (REQ-005#AC-2,
  REQ-005#AC-3).
- New formatting or rounding rules for numbers; the existing rules are used
  as-is (REQ-003).

## Constraints

| Id | Constraint | Enforced by |
| --- | --- | --- |
| C-1 | Keep the existing keyboard layout. | REQ-001#AC-2, REQ-001#AC-3, REQ-005#AC-1 |
| C-2 | No new runtime dependencies. | REQ-005#AC-2, REQ-005#AC-3 |

## Success signal

A test that drives the keypad and compares per-keystroke display strings and
the dependency/bundle diff against the pre-change build passes: the only
differences are the new `%` behaviour and the one added `%` key.

## Artifacts

- `spec/delta.yaml` — change delta, requirement index, scenarios and the
  criterion-to-scenario traceability matrix.
- `spec/requirements/REQ-001-percent-key.md`
- `spec/requirements/REQ-002-percent-semantics.md`
- `spec/requirements/REQ-003-precision-and-display.md`
- `spec/requirements/REQ-004-edge-cases-and-errors.md`
- `spec/requirements/REQ-005-constraints-and-regression.md`
