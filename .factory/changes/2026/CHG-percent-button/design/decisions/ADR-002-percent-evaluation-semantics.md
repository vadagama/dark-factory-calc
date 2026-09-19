---
schema: dark-factory.dev/adr/v1
id: adr:example-product:0002
type: adr
title: Percent as a command in the evaluation core
product: example-product
status: proposed
change: chg:example-product:2026:0002
impact: [public_api, ui]
---

## Контекст

The calculator evaluates incrementally, driven by key commands: digit keys build the current operand
`b`; a binary operator (`+`, `-`, `×`, `÷`) commits the first operand `a` and records the pending
operation `op`; `=` evaluates `a op b`; `C` resets; backspace edits the entry. There is no expression
stack and no expression-parsing mode (out of scope).

`REQ-002` and `REQ-004` fix the behaviour of `%`, on the operator's answers DEC-1 and DEC-3
(`intent.md`): with a pending operation and an entered second operand, `%` replaces the operand with
`p = a × b ÷ 100`, keeps `a` and `op`, and the following `=` evaluates `a op p`; each press applies
the rule once to the operand current at that moment; with no pending operation, `%` replaces the
entry with `b ÷ 100`; with a pending operation whose second operand has not been entered, `%` is a
no-op. `%` never produces an error state and never leaves the display blank.

Two competing readings are explicitly excluded by the acceptance criteria themselves: a plain
divide-by-100 of the current entry (fails `REQ-002` AC-1) and an implementation in which `%` commits
the pending operation (fails AC-5, via the `200 + 10 % %` sequence). The constraint that no existing
key changes behaviour means `%` may not be smuggled into the semantics of `=` or of an operator key.

The rework order `rw_a63e37d99e9b4589` requires a design that reaches this behaviour without changing
the keypad's public API. `ADR-004` answers that for the layout side: the `%` key is data — one
descriptor plus one binding — registered through the layout's own extension point when one exists and
otherwise appended to the built-in descriptor list, with no schema, signature or semantic change to
the keypad. This ADR answers the complementary half of the same question: where the capability must
live so that the keypad needs no contract change at all. The core's command surface and the keypad's
API are two separate surfaces, and only the first one grows — additively, as one new command id. The
state that percent reads (`a`, `op`, and whether a second operand has been entered) is core state; the
keypad may name a command, but it must never own that state.

## Решение

Model the core state as `(a, op, b, entry_state)` with `entry_state ∈ {empty, entered, result}`, and
add exactly one command, `percent`, to the core's command set:

1. **No pending operation** (`op` unset — including a value just produced by `=`, which is the same
   case): `b ← b ÷ 100`, `entry_state ← result`, display `b`. `0` stays `0`.
2. **Pending operation, second operand entered**: `b ← a × b ÷ 100`; keep `a` and `op` unchanged;
   display `b`; `entry_state ← result` so `b` is now the current operand of `op`.
3. **Pending operation, second operand not entered**: no-op — the display, `a`, `op` and
   `entry_state` are unchanged.
4. **Register the command additively, as data.** `percent` is one new command id in the command
   dispatch table against which the keypad already binds; if the core assembles its commands through a
   registration mechanism, the command is registered through it, otherwise one entry is added to the
   built-in table. No existing command is modified, renamed or re-signatured, and the shape of the
   keypad-side binding stays exactly what it is today (`key_id → command id`), so no keypad field,
   callback or arithmetic is needed — the layout only carries one more descriptor pointing at an
   already-existing kind of value (`ADR-004`).

Repeated presses re-apply rule (1) or (2) to the operand current at that moment (`200 + 10 % %` →
`20`, then `200 × 20 ÷ 100 = 40`; a following `=` → `240`).

The command decomposes into the core's existing multiply and divide operations; it introduces no new
arithmetic, no new operation type and no new precedence. `=` is untouched: it still evaluates
`a op b`, and `%` merely rewrites `b` before `=` runs. Degenerate operands need no special case:
`200 + 0 %` → `b = 0`; `0 + 10 %` → `b = 0`; neither path can raise an error or blank the display.

## Обоснование

- Only code that can see both `a` and `op` can compute `a × b ÷ 100` and retain the pending
  operation; that is the core, not the view or the input layer (DEC-1, AC-5). Placing the capability
  anywhere else would force the keypad to mirror or receive core state, i.e. to change the very
  contract the rework order asks to leave alone.
- Keeping the transform inside the core preserves a single source of truth for evaluation state; no
  shadow state is mirrored in the view, and the `entry_state` flag that rule (3) needs is already a
  property of the state machine.
- Because `%` reuses the existing multiply and divide operations in evaluation order, precision
  parity (`REQ-003` AC-1) holds by construction rather than by a second rounding implementation; the
  formatting side of that parity is fixed separately in `ADR-003`.
- The no-op case (`REQ-004` AC-2) is a guard on the same code path, not a separate mode, which keeps
  the state machine small and the behaviour testable.
- Reporting the error-free guarantees of `REQ-004` AC-4 is possible without a new error state: the
  command's total function `(a, op, b, entry_state) → (a, op, b', entry_state')` has no error branch.
- The contract delta of this ADR is exactly one additive command id and nothing else: the state
  machine's shape, the existing commands' arguments and semantics, and the binding shape between
  keypad and core all stay as they are, which is what `REQ-001` AC-4's regression suite observes.

## Альтернативы

| Вариант | Плюсы | Минусы | Почему не выбран |
| --- | --- | --- | --- |
| `%` handled in the view/input layer as "displayed value ÷ 100", handed to the core as a new entry | No core change; tiny diff | Cannot see the pending operation's first operand `a`, so it cannot compute `a × b ÷ 100`; it either loses or overwrites the pending operation | Fails `REQ-002` AC-1…AC-6 — `200 + 10 %` would display `0.1` and `=` would display `200.1` |
| `%` computed in the keypad/input layer and injected into the core as a typed entry, so that the core command table stays as it is | The core's command surface does not grow | To compute the percentage the keypad must read `a` and `op` from the core and write `b` back, so the keypad gains an inspection/mutation surface — a keypad API change, which is exactly what the rework asks to avoid | Strictly worse than the proposal on both counts: it fails `REQ-002` AC-1…AC-6 and it changes the keypad contract instead of leaving the layout a data-only consumer |
| `%` exposed to the keypad as a new descriptor field with a keypad-side handler (callback), leaving the core command table untouched | The core's command set is unchanged | Adds a field to the descriptor schema for a single key — an intensional keypad API change — and makes the keypad the owner of percent state that lives in the core | A contract change with no requirement behind it; percent needs `a` and `op` (`REQ-002` AC-5), which the keypad must not own (`ADR-004`) |
| Reuse an existing command id for `%` (e.g. give `÷` an extra argument or a mode flag) | No new command id | Changes the contract and the semantics of a command that pre-existing keys already invoke; every existing caller must be re-verified | Violates the additive-only constraint and the regression guarantee `REQ-001` AC-4 relies on |
| `%` implemented by replaying a scripted key sequence (`×`, `b`, `÷`, `1`, `0`, `0`, `=`) through the existing commands | Reuses existing commands only; no new command | The replay commits the pending operation; the sequence is undefined when no second operand exists; the injected key presses are part of the state and become observable | Fails AC-5 and `REQ-004` AC-2; it leaks implementation detail into user-visible behaviour |
| `%` as a binary operator of its own, with precedence/expression handling | Familiar modelling of `a % b` | Introduces an operation type and parsing semantics that are explicitly out of scope; `%` does not combine two operands — it rewrites one | Scope violation: "scientific functions … any expression-parsing mode" is out of scope; no criterion requires it |
| `%` commits the pending operation (behaves as `=` followed by `÷ 100`) | Matches some desk-calculator behaviours | The pending operation and `a` are lost, so a second `%` cannot re-apply the rule | Explicitly excluded by `REQ-002` AC-5 (`200 + 10 % %` would show `0.2` instead of `40`) |
| `%` as a display-only suffix transformation (show `b %`) | Trivial to render | Nothing is computed; the problem in `intent.md` (hand arithmetic) remains | Does not satisfy the goal or any acceptance criterion |

## Последствия

- The core's command surface grows by exactly one additive command id; every existing command keeps
  its contract, which is what `REQ-001` AC-4 relies on. The `public_api` impact recorded in this ADR
  is the core's command/state surface — **not** the keypad's, whose contract is untouched and whose
  impact is recorded as `ui` only in `ADR-004`.
- The state machine must expose whether a second operand has been entered (rule 3 guard) and whether
  a value came from a result rather than a live entry (rule 1); both are properties of the existing
  state, not new modes.
- `%` on a shown result (`REQ-004` AC-3: `200 + 10 =` → `210`, `%` → `2.1`) needs no special case
  once a result is treated as the current operand.
- Verification: `SCN-003`, `SCN-004`, `SCN-005`, `SCN-007`; criteria `REQ-002` AC-1…AC-6,
  `REQ-004` AC-1…AC-4.
- If a later round chooses different percent semantics, this ADR is revised in place and the id
  `adr:example-product:0002` is kept.
