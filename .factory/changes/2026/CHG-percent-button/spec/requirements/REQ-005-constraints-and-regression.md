---
schema: dark-factory.dev/requirement/v1
id: req:example-product:percent-button:constraints-and-regression
type: requirement
title: Constraints hold and existing operations are unregressed
product: example-product
status: draft
change: chg:example-product:2026:percent-button
---

# REQ-005 — Constraints hold and existing operations are unregressed

C-1 (keep the existing keyboard layout) and C-2 (no new runtime dependencies)
are verified here, together with the out-of-scope guard for scientific mode.
Baselines: `L0` (keypad label set), `B-3` (golden non-percent transcripts) and
`B-4` (dependency manifest, lockfile entries, and the set of modules imported
by the shipped bundle), all captured from the build that precedes this change.

## Acceptance criteria

### AC-1 — No scientific mode, and the keypad grows by exactly one label

Statement: The label set of the rendered keypad equals `L0` plus exactly one
`%` label. No scientific function label (`sin`, `cos`, `tan`, `log`, `ln`,
`x^y`, `√`, `π`, `e`, ...), no memory label, and no mode-switch control appears
anywhere in the calculator UI.

Verification: collect the post-change keypad label set and assert equality with
`L0 ∪ {%}`; assert no control matching the listed scientific/mode labels is
rendered.

Traceability: SCN `no-scientific-mode`.

### AC-2 — No new runtime dependency

Statement: Compared with baseline `B-4`, the change adds no runtime dependency
to the dependency manifest and no lockfile entry, and the shipped bundle
imports no module that was not imported by the pre-change bundle.

Verification: diff the dependency manifest and lockfile against `B-4`, and diff
the set of imported modules of the produced bundle against `B-4`; both diffs
must be empty.

Traceability: SCN `dependency-and-regression-parity`.

### AC-3 — No new runtime dependency in behaviour (network parity)

Statement: Executing the percent scenarios of REQ-001 to REQ-004 produces no
network request that the equivalent non-percent sequences do not also produce
(in particular, the calculator remains fully usable offline with `%`).

Verification: record the network requests of the percent scenarios and of the
equivalent non-percent scenarios in the same build; the request sets must be
equal.

Traceability: SCN `dependency-and-regression-parity`.

### AC-4 — Golden regression of existing behaviour

Statement: Replaying the baseline transcripts `B-3` — `2 + 2 =`,
`1 ÷ 3 =`, `5 × -3 =`, `1 + 2 + 3 =`, `10 ÷ 4 =`, `0.1 + 0.2 =`, `7 - 9 =`,
`C` after entry, and repeated `=` — on the changed build yields, per keystroke,
display strings byte-identical to `B-3`.

Verification: replay `B-3` keystroke by keystroke and compare each display
string with the recorded baseline.

Traceability: SCN `dependency-and-regression-parity`.
