---
schema: dark-factory.dev/requirement/v1
id: req:example-product:percent-button:edge-cases-and-errors
type: requirement
title: Percent edge cases and error parity
product: example-product
status: draft
change: chg:example-product:2026:percent-button
---

# REQ-004 — Percent edge cases and error parity

`%` must not invent error states or lose state in the cases below. "Same as the
equivalent sequence" always means: run the stated non-percent sequence on the
same build and compare the resulting display string and calculator state.

## Acceptance criteria

### AC-1 — Zero as the current operand

Statement: With `b = 0`, the rule of REQ-002 applies unchanged: sequence
`2 0 0 × 0 %` shows `0` after `%` and `0` after `=`; sequence `2 0 0 + 0 %`
shows `0` after `%` and `200` after `=`.

Verification: run both sequences and assert the four display strings.

Traceability: SCN `zero-and-error-parity`.

### AC-2 — Division by zero after `%`

Statement: Sequence `2 0 0 ÷ 0 % =` produces the same error state and the same
error message string as `2 0 0 ÷ 0 =`, and the same recovery behaviour from
that state (clearing input, entering a digit, and pressing an operator behave
identically in both cases).

Verification: run both sequences, assert identical error message strings, and
assert identical display/state after each recovery keystroke.

Traceability: SCN `zero-and-error-parity`.

### AC-3 — `%` pressed with no current operand entered

Statement: With a pending `+` and no current operand entered (sequence
`2 0 0 +`), pressing `%` treats the current operand as `0`: the display is
unchanged (`200`), the pending `+` is preserved, and the following `=` shows
`200`.

Verification: run `2 0 0 + % =` and assert the display is `200` after `%` and
after `=`; assert no error is raised.

Traceability: SCN `no-second-operand`.

### AC-4 — `%` pressed immediately after `=`

Statement: With a completed calculation whose result `b` is displayed and no
pending operation (sequence `5 0 =`), pressing `%` shows `0.5`, creates no
pending operation, and a following digit starts a new operand entry.

Verification: run `5 0 = %` and assert the display is `0.5`, that the pending
operator state is empty, and that the next digit starts a fresh entry.

Traceability: SCN `bare-percent`.

### AC-5 — Negative current operand

Statement: A negative current operand is handled by the same rule: sequence
`2 0 0 + (- 1 0) %` shows `-20` after `%` and `180` after `=`; sequence
`2 0 0 - (- 1 0) %` shows `-20` after `%` and `220` after `=`. The sign is
formatted with the same rule as for other operations.

Verification: run both sequences and assert the four display strings.

Traceability: SCN `negative-operand`.

### AC-6 — No error except where the equivalent sequence errors

Statement: None of AC-1, AC-3, AC-4, AC-5 raises an error, and AC-2 raises an
error only because the equivalent non-percent sequence `2 0 0 ÷ 0 =` raises the
same error.

Verification: run the sequences of AC-1 and AC-3 to AC-5 and assert no error
message/state occurs; run AC-2 and assert the error matches the equivalent
non-percent sequence.

Traceability: SCN `zero-and-error-parity`, `no-second-operand`,
`negative-operand`.
