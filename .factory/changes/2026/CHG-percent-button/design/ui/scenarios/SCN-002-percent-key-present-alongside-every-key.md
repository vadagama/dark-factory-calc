---
schema: dark-factory.dev/ui-scenario/v1
id: SCN-002
type: ui_scenario
title: The keypad shows the percent key alongside every existing key
product: example-product
status: proposed
change: chg:example-product:2026:0002
steps:
  - id: S1
    text: On the pre-change build, read every keypad label in row-major order (left to right, top to bottom) → `before_labels`, `before_order`.
    screen: SCR-001
  - id: S2
    text: Open the calculator on the changed build and read every keypad label in row-major order → `after_labels`, `after_order`.
    screen: SCR-001
  - id: S3
    text: Assert the number of rendered keys labelled `%` is exactly 1 (REQ-001 AC-1).
    screen: SCR-001
  - id: S4
    text: Assert `after_labels − before_labels == { "%" }` and `before_labels − after_labels == ∅` — one new label, no pre-change label missing or renamed (REQ-001 AC-2).
    screen: SCR-001
  - id: S5
    text: Assert `[k for k in after_order if k in before_labels] == before_order` — filtering the rendered order down to the pre-change labels reproduces the pre-change order element by element, with no transposition (REQ-001 AC-3).
    screen: SCR-001
  - id: S6
    text: Assert the `%` key is rendered, enabled and visually indistinguishable in weight from the other keypad keys, and that no key carries a second label or a long-press affordance.
    screen: SCR-001
---

# SCN-002 — The keypad shows the percent key alongside every existing key

The user's first sight of the change is the keypad itself: the same calculator as before, with one more key,
labelled `%`. This is the screen-level scenario behind `REQ-001` AC-1…AC-3 — the one thing the change is allowed
to alter visibly is the label multiset, by exactly one `%`.

The read is row-major because that is the order in which the keypad renders its descriptors, and it is the order
`SCN-002` step 5 filters down to the pre-change labels; the position of the new key is deliberately not
constrained by the requirement (`REQ-001`, "Out of scope for this requirement"). On this screen the `%` key is
appended after every pre-existing key in the row-major read (`SCR-001`, ordering rule; `ADR-004` branch 3), which
makes step 5 hold by construction.

Step 6 records the presentation side of the same constraint: `%` is an ordinary keypad `Button`, not a variant, a
badge, a toggle or a gesture on another key — `ADR-004` rejects overloading a pre-existing key precisely because
it would put a pre-existing key's label count and behaviour in question.

Grounded in: `REQ-001` AC-1…AC-3, the constraint in `intent.md` ("no existing key may be removed, renamed or
reordered"), `design/overview.md` (rendered-keypad layer: one more key, labelled `%`), `ADR-004`, `ADR-005`.

Traces: `req:example-product:calculator:percent-key#AC-1`,
`req:example-product:calculator:percent-key#AC-2`,
`req:example-product:calculator:percent-key#AC-3`.
