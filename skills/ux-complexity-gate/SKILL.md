---
name: ux-complexity-gate
description: >-
  Verifies that a screen (mockup or implemented) and the app's navigation are
  not too complex: actions and decisions per screen, touch targets,
  affordances, depth and taps per task. Use before accepting a mockup, before
  merging a screen (after design-fidelity-gate), when navigation changes, and
  when closing a redesign milestone. Does not judge aesthetics or brand.
---
# Complexity gate

Comes from the research done for Profe (a redesign for non-technical teachers). It measures whether a screen and the app are easy to use; fidelity to the mockup is measured by `design-fidelity-gate`. **It is run by a verifier other than whoever designed or implemented.** It hasn't been used in production yet, so thresholds marked *calibrate* are proposals.

## Inputs

- **Screens to review:** URL or screenshot of each, with its type (home, list, form or detail) and its state inventory (the same one used for fidelity).
- **Frequent tasks:** `tasks.json`, with frequency (daily, weekly or occasional) and a tap budget. Built with the product owner and the real user.
- **Thresholds:** versioned in the repo and changed only in a separate PR.

## Layers

- **A (automated):** counts over the first viewport, no scroll.
- **V (vision agent):** same screenshot with numbered marks; run 3 times, keep what appears in ≥ 2.
- **U (real user):** creates tickets, never blocks a merge.

Viewports: one Android (412×915) and one iPhone (390×844).

## Per-screen thresholds

"First viewport" excludes the top bar and the navigation bar. S = Nielsen severity: **S3 and S4 block**.

| # | Check | Threshold | Layer | S |
|---|---|---|---|---|
| P1 | Visible primary actions | ≤ 1 | A+V | 3 |
| P2 | Distinct decisions (a homogeneous group counts as 1) | ≤ 5; fail if > 8 (*calibrate*) | A | 2/3 |
| P3 | Interactive elements (homogeneous lists don't add up) | ≤ 20; fail if > 30 (*calibrate*) | A | 2/3 |
| P4 | Options per decision point | ≤ 5, more only if sorted or grouped | A+V | 2 |
| P5 | Touch target of frequent or destructive actions | ≥ 44 px/pt (Android 48 dp) | A | 3 |
| P6 | Touch target of any interactive element | ≥ 24×24 (WCAG 2.5.8) | A | 4 |
| P7 | Primary action and the 3 most frequent | in the bottom 60 % (thumb zone) | A | 2 |
| P8 | Text | nothing < 12; body and buttons ≥ 14 | A | 3/2 |
| P9 | Contrast | text ≥ 4.5:1; borders and states ≥ 3:1 | A+V | 3 |
| P10 | Everything tappable looks tappable | 100 % | A+V | 3 |
| P11 | False affordances (looks like a button but isn't) | 0 | V | 3 |
| P12 | Icon-only controls | universal only (back, close, search, more, add), with accessible label | A+V | 3/2 |
| P13 | Levels of hidden content (tab → accordion → sheet = 3) | ≤ 2 | A+V | 3 |
| P14 | Forms | 1 decision per step by default; warn if > 7 fields | A | 2 |
| P15 | Dialogs | ≤ 2 actions; confirm only the irreversible; reversible gets "Undo" | A+V | 2 |
| P16 | Empty list | state + hint + action | V | 2 |
| P17 | Cognitive walkthrough of the step: knows what to do, sees the action, links it to the goal, sees progress? | 4 yes | V | 3 |

## App-level thresholds (navigation)

| # | Check | Threshold | S |
|---|---|---|---|
| A1 | Bottom bar | 3 to 5 labeled destinations; no hamburger for main sections | 3 |
| A2 | Depth from the bar | ≤ 3 levels | 3 |
| A3 | Taps from Home per task | ≤ budget (daily 3, weekly 5, occasional 7) | 3/2 |
| A4 | Dead-end screens | 0 | 4 |
| A5 | Reachable from Home | 100 % of the inventory | 3 |
| A6 | Back closes the overlay before changing screens; back from a tab goes to Home | always | 3 |
| A7 | Common actions with the same label, position and style | 1 variant per action | 2 |
| A8 | Opening the app always lands on Home | always | 3 |

## Real user (optional, per milestone)

- SEQ (1 to 7) after each daily task; if < 5, ask why.
- UMUX-Lite at the end: target ≥ 68 on the SUS scale.
- With a single user each session finds roughly a third of the problems: don't generalize.

## Result

- **FAIL:** any S4, or an automated S3, or a visual S3 with majority on P1, P10, P11 or P12. At mockup stage it goes back to Design; at implemented stage it blocks the merge.
- **REVIEW:** only S2, or another visual S3. Doesn't block if the reviewer accepts it in writing on the PR; each accepted S2 opens a ticket.
- **PASS:** nothing, or cosmetic only.
- 2 rounds max; after that the product owner (Simon) decides. **Never loosen a threshold to make it pass:** fix the mockup or the app.
- The report goes as a PR comment, with blockers, warnings, metrics and taps per task.

## Platforms

The core applies equally to web, Android and iOS. What changes:

- **Touch targets:** the platform's (48 dp on Android, 44 pt on iOS).
- **Back (A6):** on Android, system or predictive back; on iOS, swipe-back and the bar button.
- **Bar (A1):** Material NavigationBar or iOS tab bar, always 3 to 5 destinations.
- **Measurement:** on web, DOM with Playwright + axe. On native, from the semantics/accessibility tree (Compose semantics, XCUITest), or by the vision agent on the fidelity gate's screenshots. Not built yet.

Measurement details and the vision agent prompt are in [`reference.md`](reference.md).

## Chaining with design-fidelity-gate

1. **This gate on the mockup.** It goes first, because the fidelity gate would copy any usability problem.
2. Implementation.
3. `design-fidelity-gate`.
4. **This gate on the implemented screen**, with real touch targets, back stack and real taps.
5. Merge.

At app level, it runs when navigation changes and when each milestone closes. If both gates fail, fix complexity first.
