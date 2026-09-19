---
schema: dark-factory.dev/adr/v1
id: adr:example-product:0003
type: adr
title: Display precision and formatting parity for percent results
product: example-product
status: proposed
change: chg:example-product:2026:0002
impact: [public_api]
---

## Контекст

`REQ-003` requires `%` to be indistinguishable, character for character, from the calculator's
existing operations: the display after `%` must equal the display produced by the equivalent
existing-key sequence `a × b ÷ 100 =` (`AC-1`), `%` must use the existing number formatter and add no
formatting rule of its own — no extra decimals, thousands separators or exponent notation (`AC-2`),
and the operand handed to the pending operation must be exactly the value displayed after `%` (`AC-3`).

`AC-3` encodes the operator's answer DEC-2 (`intent.md`): the displayed (rounded) value is committed
and no digits are held beyond the display. Its check uses
`123456789 × 123456789 ÷ 100 = 152415787501905.21`, a percentage whose exact value has more
significant digits than a plain-digit display shows; the last displayed digit is exactly where an
implementation that carries unrounded internal precision into `=` diverges from an implementation that
commits the displayed value.

Two places decide how a number is shown and how much of it survives: the arithmetic core and the
number formatter. A percent-specific copy of either would be a second source of truth and would drift
from the `+`, `-`, `×`, `÷` behaviour that `AC-1` and `AC-2` compare against.

This decision lives entirely on the value/display path of the core. The rework order
`rw_a63e37d99e9b4589` asks for a design that reaches the required behaviour without changing the
keypad's public API; this ADR contributes to that answer by keeping the parity mechanism internal —
the format-then-parse boundary of point 3 below is inside the core, the keypad neither formats nor
rounds, and no keypad field, entry point or signature is involved (`ADR-004`). The alternative that
would avoid touching the core's value path at all — computing the percentage in the keypad and
committing it there — is analysed in the alternatives table and rejected, precisely because it would
move both arithmetic and formatting into the layer whose contract must stay unchanged.

## Решение

1. **Reuse, do not reimplement.** The `percent` command (`ADR-002`) computes `p = a × b ÷ 100` by
   invoking the same internal multiply and divide operations, in the same evaluation order, that the
   `×` and `÷` keys use. No percent-specific arithmetic, precision or rounding routine is introduced.
2. **Single formatter.** `p` is rendered through the existing number formatter — the same instance and
   settings used for every other result. `%` contributes no formatting rule: no fixed decimal count,
   no thousands separator, no exponent notation, no display-width change.
3. **Commit the displayed value.** The operand that `%` stores as the current operand `b`, and that a
   following `=` consumes, is the numeric value of the string the formatter produced. Internally this
   is a formatter round-trip: format `p`, then take the value of that formatted string. Hidden digits
   of `a × b ÷ 100` beyond the display do not survive into `=` (DEC-2, `AC-3`).
4. **No new display states.** `%` cannot introduce a notation that does not already appear for the
   same value produced by an existing operation, and it cannot widen or narrow the display.
5. **No keypad-facing contract change.** The parity mechanism adds nothing the keypad can see: no
   formatting mode, no precision field, no callback and no render variant is requested from the
   keypad, and the value the display shows is produced by the same pipeline in the same way as for
   `+`, `-`, `×`, `÷`. The one new step on the value path — the round-trip of point 3 — happens
   between the core and the formatter, before any display binding.

Equivalently: for any `a`, `b`, `op`, `%` produces the value of the existing-key route `a × b ÷ 100 =`
and displays it with the existing formatter; the commitment boundary is the displayed string.

## Обоснование

- Parity becomes a property of construction, not of duplicated rules: there is exactly one arithmetic
  path and exactly one formatter, so `AC-1` and `AC-2` cannot diverge from the existing keys by
  construction. A percent-only rounding or formatting rule would have to be kept in sync forever.
- `AC-3` demands the formatter round-trip: it asserts that `R1` (`a op b % =`) and `R2` (`a op <` the
  displayed `X` re-entered `> =`) are character-identical, including for a percentage whose exact
  value exceeds the display width. This only holds if the stored operand is the displayed value.
- The operator's DEC-2 answer selects the displayed-value commitment over full internal precision;
  this ADR is its implementation, and `REQ-003` states explicitly that a later reversal is expressed
  by rewriting `AC-3` in place rather than by renumbering anything.
- Reusing the existing evaluation order preserves the left-to-right semantics assumed by the
  reference route in `REQ-003` ("`p = a × b ÷ 100` is exactly the sequence `a × b ÷ 100 =` evaluated
  left to right"), so the reference route is literally the same code path.
- Keeping the whole mechanism behind the display binding is also what makes the keypad answer cheap:
  the layout only has to name a command (`ADR-004`), and there is no precision or notation knob that
  a keypad descriptor or entry point would have to carry.

## Альтернативы

| Вариант | Плюсы | Минусы | Почему не выбран |
| --- | --- | --- | --- |
| Keep full internal precision of `p`, format only for display | Apparently "more accurate"; avoids an explicit round-trip step | The value handed to `=` differs from the value shown; `SCN-006` step 4 shows `R1 ≠ R2`; contradicts the operator's answer DEC-2 | Fails `REQ-003` AC-3 by construction; DEC-2 commits the displayed value |
| Compute the percentage and commit the rounded value in the keypad/input layer, leaving the core's value path untouched | No change to the core's operand/display contract; the formatting parity is decided in one place next to the key that triggers it | The keypad would need its own multiply/divide and its own format-then-parse step — a second arithmetic and formatting path that drifts; and it cannot see `a`, so it cannot compute `a × b ÷ 100` at all | Fails `REQ-003` AC-1/AC-2 (single arithmetic path and single formatter) and fails `REQ-002` AC-1…AC-6 structurally; it also changes the keypad contract the rework order asks to leave alone |
| Percent-specific rounding, e.g. round all percent results to two decimals | Predictable percentages; easy to explain | Introduces a new rounding rule, which is out of scope; breaks character-identity with the existing-operation route | Fails `AC-1`/`AC-2` parity; scope violation ("new rounding rules" is out of scope) |
| Format percent results by string manipulation (trim/append characters) | Trivial to implement; no formatter work | A second formatting implementation that will drift from the formatter in width, notation and rounding | Breaks the single-formatter guarantee `AC-2` relies on |
| A second formatter instance configured for percent | Isolation; no risk to existing formatting | Two sources of truth for width/precision; drift is invisible until a regression | Unnecessary duplication; `REQ-003` asks for identity with the existing formatter |
| Truncate instead of round the committed operand | Simple integer arithmetic | Differs from the formatter's own rounding, so the shown and committed values disagree | Fails `AC-3` (the committed operand must be the displayed value) |

## Последствия

- The core needs an explicit format-then-parse step when a `%` result becomes the operand of a
  pending operation; this is a small, well-defined boundary and the only new operation on the value
  path. It is invisible to the keypad: no descriptor, binding or keypad entry point is involved.
- No data schema or persisted state is touched; the impact is on the core's internal operand/display
  contract (`public_api` of the core), while the keypad's contract keeps its impact `ui`-only status
  as recorded in `ADR-004`.
- Verification: `SCN-006` (including the large-operand step 4) and `SCN-002`; criteria
  `REQ-003` AC-1…AC-3. Regression: `SCN-008` confirms the formatter's behaviour for existing-key
  sequences is unchanged.
- If a later round reverses DEC-2 toward full internal precision, `REQ-003` AC-3 is rewritten in
  place and this ADR is revised in place; the id `adr:example-product:0003` is kept.
