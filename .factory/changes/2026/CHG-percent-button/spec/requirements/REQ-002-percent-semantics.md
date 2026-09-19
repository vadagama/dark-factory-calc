---
schema: dark-factory.dev/requirement/v1
id: req:example-product:percent-button:percent-semantics
type: requirement
title: Percent semantics applied to the current operand
product: example-product
status: draft
change: chg:example-product:2026:percent-button
---

# REQ-002 — Percent semantics applied to the current operand

`%` converts the operand currently displayed (the *current operand*) into a
percentage and leaves the pending operation to be completed by `=`. The first
operand `a` and the pending operator are not consumed by `%`.

Rule applied on a `%` press, where `b` is the value currently displayed:

| Pending operator | New current operand | Rationale |
| --- | --- | --- |
| `+` | `a × b ÷ 100` | "b percent of a" is added to `a` |
| `-` | `a × b ÷ 100` | "b percent of a" is subtracted from `a` |
| `×` | `b ÷ 100` | the second factor is a plain ratio |
| `÷` | `b ÷ 100` | the divisor is a plain ratio |
| none | `b ÷ 100` | a bare percentage is a ratio |

All examples below are keystroke sequences; "shows X" means the display string.

## Acceptance criteria

### AC-1 — Percent of the first operand with a pending addition

Statement: With a pending `+`, first operand `a` and current operand `b`,
pressing `%` replaces the displayed current operand with `a × b ÷ 100` and
keeps the pending `+`. Sequence `2 0 0 + 1 0 %` shows `20`; continuing with `=`
shows `220`.

Verification: run the sequence and assert both display strings.

Traceability: SCN `add-percent-of-operand`.

### AC-2 — Percent of the first operand with a pending subtraction

Statement: With a pending `-`, first operand `a` and current operand `b`,
pressing `%` replaces the displayed current operand with `a × b ÷ 100` and
keeps the pending `-`. Sequence `2 0 0 - 1 0 %` shows `20`; continuing with `=`
shows `180`.

Verification: run the sequence and assert both display strings.

Traceability: SCN `subtract-percent-of-operand`.

### AC-3 — Ratio with a pending multiplication

Statement: With a pending `×`, first operand `a` and current operand `b`,
pressing `%` replaces the displayed current operand with `b ÷ 100` and keeps
the pending `×`. Sequence `2 0 0 × 1 0 %` shows `0.1`; continuing with `=`
shows `20`.

Verification: run the sequence and assert both display strings.

Traceability: SCN `multiply-by-percent`.

### AC-4 — Ratio with a pending division

Statement: With a pending `÷`, first operand `a` and current operand `b`,
pressing `%` replaces the displayed current operand with `b ÷ 100` and keeps
the pending `÷`. Sequence `2 0 0 ÷ 1 0 %` shows `0.1`; continuing with `=`
shows `2000`.

Verification: run the sequence and assert both display strings.

Traceability: SCN `divide-by-percent`.

### AC-5 — Bare percentage with no pending operation

Statement: With no pending binary operation and value `b` displayed, pressing
`%` replaces the display with `b ÷ 100`. Sequence `5 0 %` shows `0.5`.

Verification: run the sequence and assert the display string.

Traceability: SCN `bare-percent`.

### AC-6 — Pending operator and first operand survive a `%` press

Statement: For each of AC-1 to AC-4, the pending operator and the first operand
`a` are unchanged by the `%` press, so that a single following `=` produces
exactly the result stated in AC-1 to AC-4. No state reset, no error, and no
second `=` press is required.

Verification: after the `%` press, inspect the pending operator state and the
stored first operand, then press `=` once and assert the display equals the
value stated in AC-1 to AC-4.

Traceability: SCN `add-percent-of-operand`, `subtract-percent-of-operand`,
`multiply-by-percent`, `divide-by-percent`.

### AC-7 — A second `%` press applies the rule to the displayed value

Statement: Pressing `%` again applies the rule of AC-1 to AC-5 to the value
currently displayed (the result of the first press), not to the originally
entered digits. Sequence `2 0 0 + 1 0 % %` shows `20` after the first press and
`40` after the second; continuing with `=` shows `240`.

Verification: run the sequence and assert all three display strings.

Traceability: SCN `repeated-percent`.
