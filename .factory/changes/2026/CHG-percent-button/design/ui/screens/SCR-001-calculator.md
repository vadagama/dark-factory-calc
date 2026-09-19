---
schema: dark-factory.dev/ui-screen/v1
id: SCR-001
type: ui_screen
title: Calculator display and keypad
product: example-product
status: proposed
change: chg:example-product:2026:0002
route: /
states:
  loading: >-
    The app shell that hosts the calculator is starting: the calculator panel is present but the display slot
    shows a `Spinner` and the keypad grid is not rendered, so no key — and therefore no `%` key — is pressable
    yet. This state is the app shell's and is untouched by the change; percent introduces no loading path, no
    extra request and no placeholder of its own.
  empty: >-
    Reset state, entered with `C`: the display reads `0`, the core holds neither a first operand nor a pending
    operation, and the whole keypad is rendered and operable, including the one `%` key. This screen has no
    collection (history and memory are out of scope), so the `EmptyState` pattern has no place here; the "empty"
    state is the reset display of `REQ-004` AC-4, and pressing `%` in it leaves the display reading `0` with no
    error indicator and no blank display.
  error: >-
    Two cases, both outside percent's own behaviour. (a) Core-level errors on the pre-existing path (for example
    a division by zero): presented exactly as before the change, with the same component and wording; percent
    has no error branch — it never errors, never blanks the display and treats every degenerate operand (`0`, a
    missing second operand, the reset entry) as an ordinary value (`REQ-002` AC-6, `REQ-004` AC-4). (b) A failure
    to start the app shell: the panel presents the `Alert` component and the keypad is not rendered. No new
    message, `Dialog` or recovery control is introduced by this change.
  success: >-
    Normal operating state, covered by `SCN-001` … `SCN-007`: the display shows the current operand or result, the
    keypad grid is fully interactive, and exactly one key is labelled `%`. Pressing `%` with a binary operation
    pending and a second operand entered replaces the shown operand with `p = a × b ÷ 100` and retains the pending
    operation and the first operand (`REQ-002`); with no pending operation it replaces the entry with `b ÷ 100`
    (`REQ-004` AC-1, AC-3); with a pending operation and no second operand it changes nothing (`REQ-004` AC-2). The
    following `=` evaluates `a op p` through the unchanged evaluation path, and the characters shown come from the
    existing number formatter (`REQ-003`).
  access: >-
    The product declares no per-user permission or entitlement surface for the calculator (`product.md`), so the
    screen is available to every user who can open the product, and the keypad — the `%` key included — is always
    rendered and always enabled. No feature flag, rollout toggle or configuration key gates the key: `REQ-001`
    AC-1 expects exactly one `%` key unconditionally, and `ADR-004` rejects environment-dependent label sets for
    that reason. If the host product ever gates the calculator itself, that gate sits outside this screen and
    outside this change; a denial, where one exists, is presented with the `Alert` component in place of the
    panel, with no keypad and no `%` key.
elements:
  - id: EL-calculator-card
    kind: container
    label: Calculator panel
    component: Card
  - id: EL-display
    kind: input
    label: Calculator display (read-only; shows the current operand or result)
    component: Input
  - id: EL-keypad
    kind: container
    label: Keypad key grid (read row-major, left to right, top to bottom)
    component: Toolbar
  - id: EL-key-7
    kind: button
    label: '7'
    component: Button
  - id: EL-key-8
    kind: button
    label: '8'
    component: Button
  - id: EL-key-9
    kind: button
    label: '9'
    component: Button
  - id: EL-key-divide
    kind: button
    label: '÷'
    component: Button
  - id: EL-key-4
    kind: button
    label: '4'
    component: Button
  - id: EL-key-5
    kind: button
    label: '5'
    component: Button
  - id: EL-key-6
    kind: button
    label: '6'
    component: Button
  - id: EL-key-multiply
    kind: button
    label: '×'
    component: Button
  - id: EL-key-1
    kind: button
    label: '1'
    component: Button
  - id: EL-key-2
    kind: button
    label: '2'
    component: Button
  - id: EL-key-3
    kind: button
    label: '3'
    component: Button
  - id: EL-key-subtract
    kind: button
    label: '-'
    component: Button
  - id: EL-key-0
    kind: button
    label: '0'
    component: Button
  - id: EL-key-decimal
    kind: button
    label: '.'
    component: Button
  - id: EL-key-equals
    kind: button
    label: '='
    component: Button
  - id: EL-key-add
    kind: button
    label: '+'
    component: Button
  - id: EL-key-clear
    kind: button
    label: 'C'
    component: Button
  - id: EL-key-backspace
    kind: button
    label: '⌫'
    component: Button
  - id: EL-key-percent
    kind: button
    label: '%'
    component: Button
  - id: EL-loading-spinner
    kind: indicator
    label: Calculator shell loading indicator (loading state only)
    component: Spinner
  - id: EL-error-alert
    kind: alert
    label: Calculator unavailable (shell start failure or access denial)
    component: Alert
transitions:
  - to: SCR-001
    trigger: Open the calculator
    condition: Loading; the panel shows the Spinner until the app shell has started, then the success state
  - to: SCR-001
    trigger: Press '%' while a binary operation is pending and a second operand has been entered
    condition: Display becomes `a × b ÷ 100`; the pending operation and the first operand are retained (REQ-002 AC-1…AC-4)
  - to: SCR-001
    trigger: Press '%' with no pending operation, on a plain entry or on a displayed result
    condition: Display becomes `b ÷ 100` (REQ-004 AC-1, AC-3)
  - to: SCR-001
    trigger: Press '%' with a pending operation and no second operand
    condition: No change at all; display and pending operation stay as they are (REQ-004 AC-2)
  - to: SCR-001
    trigger: Press '=' after '%'
    condition: Display becomes `a op p` through the existing evaluation path (REQ-002)
  - to: SCR-001
    trigger: Press any pre-existing key (digit, '.', '+', '-', '×', '÷', '=', 'C', '⌫')
    condition: Pre-change behaviour, character-identical display (REQ-001 AC-4)
  - to: SCR-001
    trigger: A core-level error occurs on the pre-existing path
    condition: Unchanged error presentation; '%' never triggers this state (REQ-004)
---

# SCR-001 — Calculator display and keypad

## Purpose

The calculator is a single screen: a panel that holds one read-only display and one keypad grid of keys. Every
action of this change happens here — there is no secondary screen, no dialog and no confirmation step, because
`REQ-002` and `REQ-004` make `%` a plain keypad key whose only output is the display. The screen exists to let the
user enter operands, choose a binary operation and complete it, and, after this change, to apply a percentage to
the operand that is current at that moment instead of working the percentage out by hand.

The change's applied impact is exactly this screen's keypad: one additional key labelled `%`, with every
pre-existing key keeping its label, its function and its relative order (`REQ-001` AC-1…AC-3, the constraint in
`intent.md`). `ADR-004` composes the key as two data values (one descriptor, one binding) and `ADR-005` freezes
the keypad's public API, so from the screen's point of view the delta is one rendered key.

## Elements

| Element | Kind | Component | Role / delta |
| --- | --- | --- | --- |
| `EL-calculator-card` | container | `Card` | The screen's only panel, holding the display and the keypad. Unchanged. |
| `EL-display` | input | `Input` | Read-only display of the current operand or result, i.e. the output of the existing formatter (`REQ-003`). Unchanged. |
| `EL-keypad` | container | `Toolbar` | Grouping of the keypad keys; the row-major read order that `SCN-002` filters lives on this element. Unchanged grouping, one more child. |
| `EL-key-0` … `EL-key-9`, `EL-key-decimal` | button | `Button` | Digit and decimal keys. Unchanged in label, function and relative order. |
| `EL-key-add`, `EL-key-subtract`, `EL-key-multiply`, `EL-key-divide` | button | `Button` | The binary operations `+`, `-`, `×`, `÷`. Unchanged; they are what `%` retains while it transforms the current operand (`REQ-002`). |
| `EL-key-equals` | button | `Button` | Completes the pending operation through the unchanged evaluation path (`REQ-002`, `REQ-003`). Unchanged. |
| `EL-key-clear`, `EL-key-backspace` | button | `Button` | Reset and delete. Unchanged; `C` produces the reset display checked by `REQ-004` AC-4. |
| `EL-key-percent` | button | `Button` | **The single element this change adds.** One key labelled `%`, bound to the core's additive `percent` command id (`ADR-002`, `ADR-004`). It is always enabled and never disabled by operand state: with a pending operation and no second operand it is a silent no-op (`REQ-004` AC-2), not a disabled key. |
| `EL-loading-spinner` | indicator | `Spinner` | Loading state only. Not rendered in the success state. Unchanged. |
| `EL-error-alert` | alert | `Alert` | Shell start failure or access denial. Not rendered in the success state. Unchanged in component and wording; this change adds no message. |

Every element maps onto a component or pattern that already exists in the UI kit: `Card`, `Input`, `Toolbar`
(pattern), `Button`, `Spinner`, `Alert`. The kit has no dedicated "display" or "keypad" component, so the display
uses the read-only `Input` and the key grid's grouping uses the `Toolbar` pattern; no new component, pattern or
variant is introduced, and the keypad's renderer contract is consumed as-is (`ADR-005`).

### Order of the pre-existing keys

The list above is written in the conventional keypad skeleton purely so that every pre-existing key has a stable
`EL-*` anchor; the **authority on the pre-existing row-major order is the baseline read** of the pre-change build
(`before_labels`, `before_order` in `SCN-002` step 1 / `REQ-001` AC-3). Ordering rule for this change: `%` is a
new key placed *after* every pre-existing key in the row-major read, so that filtering the rendered order down to
the pre-change labels reproduces the pre-change order element by element with no transposition (`REQ-001` AC-3;
`ADR-004` branch 3, which appends at a free cell at the end of the row-major order and leaves the position
unconstrained — `REQ-001` deliberately does not constrain the cell of `%`).

## Percent behaviour on this screen

- **With a pending binary operation and an entered second operand** the displayed operand is replaced by
  `p = a × b ÷ 100`; the pending operation and the first operand survive, so the following `=` shows `a op p`
  (`REQ-002` AC-1…AC-4). `%` therefore does **not** look like `=` followed by `÷ 100`, and it does **not** commit
  the operation.
- **Each press applies the rule once** to the operand current at that moment: `200 + 10 %` shows `20`, a second
  `%` shows `40`, and `=` then shows `240` (`REQ-002` AC-5).
- **With no pending operation** — a plain entry or a value shown after `=` — the display is replaced by
  `b ÷ 100` (`REQ-004` AC-1, AC-3).
- **With a pending operation and no second operand** the key is a no-op: the display keeps reading the first
  operand and the pending operation stays (`REQ-004` AC-2).
- **Never an error, never blank:** the value shown after `%` goes through the same arithmetic path and the same
  formatter as `+`, `-`, `×` and `÷`, and the committed operand is exactly the value displayed (`REQ-003`
  AC-1…AC-3, `ADR-003`). The characters shown after `%` are, character for character, the characters the
  existing-key route `a × b ÷ 100 =` shows.

## What does not change on this screen

No pre-existing key is removed, renamed, reordered or given a second behaviour; no `%` character is mapped from
the OS keyboard (out of scope in `intent.md`); no new indicator of the pending operation is introduced (the core
holds `a` and `op` and the display keeps showing only the current operand or result); no layout reflow rule, no
history, no memory key and no scientific key is added. A user who never presses `%` must not notice the change
(`REQ-001` AC-4, `SCN-007`).

## States

The screen describes all five states because the same panel serves loading, the reset entry, the pre-existing
error path, the whole working session and access to the product: `loading` and `error`/`access` deliberately
leave the panel without a keypad, `empty` is the reset display, and `success` is the only state in which the
keypad — and with it the one `%` key — is rendered. Percent owns no state of its own: it adds no loading path, no
error branch and no gated variant, which is what keeps the label set read by `SCN-002` deterministic.

## Transitions

The product is single-screen, so every transition of this screen leads back to `SCR-001`; the interesting part of
each entry is the condition under which the display changes, and those conditions are the percent rules
themselves (`%` with a pending second operand, `%` without a pending operation, `%` as a no-op) plus the
pre-existing key paths that must stay untouched (`REQ-001` AC-4).

## Ids

`SCR-001`, the `EL-*` ids and the step ids `S<n>` of the scenarios of this ChangeSet are stable comment anchors.
They are never renamed or renumbered; removals are not expressed by editing an id, and new elements are appended
at the end of the list.
