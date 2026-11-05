---
name: design-fidelity-gate
description: >-
  Verifies that an implemented screen (web, Android or iOS) matches its mockup
  before merge: screen inventory, reproducible screenshots, a region-based
  comparator with fixed thresholds, and a by-eye review of what pixels can't
  see. Use when writing a screen's inventory, implementing it, and reviewing
  its PR. For whether the screen is easy to use, see ux-complexity-gate.
---
# Fidelity gate: design vs. implementation

Proven in production on a web + Android app: one run took a home screen from 7.26 % to 0.03 % differing pixels. The core idea: an agent doesn't get closer to the design because you describe it better, but because it can measure how far off it is. **The number says where to look; the review decides.**

## Roles

- **Design:** makes the mockup and the inventory.
- **Implementation:** builds the screen against them.
- **Verifier:** cannot be whoever implemented it. Can be design.

## 1. Before implementing

1. **HTML mockup versioned in the repo**, using shared demo data (a fictional user the app can also load via seed or fixture). State and theme are selected by URL (`?state=`, `?theme=`).
2. **Screen inventory** (`docs/.../specs/screens/<screen>.md` or a ticket comment):
   - Elements top to bottom, including what is **not** there (logo, chrome, badges).
   - Every state: empty, loading, error, with data, dialogs and sheets.
   - How to reach each state in the app.
   - Exact copy, tokens and measurements.
   - Known exceptions.
   - On native, which parts are **brand** and which are **platform** (see below).
3. **Shared base first:** base components (bars, cards, rows, amounts, chips) copy the mockup's CSS measurements once, and screens inherit them.
4. **One ticket is one complete screen**, in a PR straight to `main`, no integration branch.
5. **Implement by reading the mockup's HTML and CSS** (the values), not by looking at screenshots.

## 2. Reproducible screenshots on both sides

- Same viewport, DPR 1, theme, locale and time zone, plus fixed date, animations off, fonts loaded and the same data.
- **Web:** Playwright against the real app. **Android:** Robolectric + Roborazzi (Compose on the JVM, no emulator). Since they are tests, any PR regenerates them. **iOS:** an equivalent snapshot test (e.g., swift-snapshot-testing).
- Whatever the OS draws (status bar, gestures, keyboard) is masked.
- The mockup is rendered to PNG with Playwright at the same size. Example sizes: 390×844 and 1440×900 on web, 412×915 dp on Android.

## 3. Region-based comparator

- **One matrix per screen** (JSON) that pairs state × viewport × theme with the mockup/screenshot pair and declares exceptions with a `label` and reason.
- The image is cut into **named regions** (top bar, content, bottom navigation, dialog or sheet), each with its own threshold. A pair takes the status of its worst region: **pass / review / fail**.
- The report (`report.html`) has side-by-side, 50 % overlay, diff map and average color per region. Average color exists because a small tone change doesn't show up in the %.
- **Starting-point thresholds (web):**

  | Region | Pass | Review |
  |---|---|---|
  | Shell (bars, nav) | ≤ 0.5 % | ≤ 2 % |
  | Content | ≤ 1 % | ≤ 5 % |

  Above "review" it fails.
- **PR gate:** a "review", a "fail" or a missing screenshot blocks, unless the verifier accepts it in writing with a reason (e.g., different test data).
- **Exceptions** (something the app doesn't have yet). In this order:
  1. A mockup flag that hides it.
  2. `hide` by selector.
  3. `mask`, only if it occupies the same place on both sides.

  An exception not listed in the inventory doesn't count.
- **Building it:** render the mockup and capture the app with Playwright; diff each named region with pixelmatch (anti-aliasing tolerance ~0.1); drive it from a per-screen JSON matrix plus region profiles (JSON, per device/screen type); emit `results.json` and an HTML report. A `--gate` flag exits non-zero if any pair isn't "pass".

## 4. By-eye review (this is what approves)

With the screenshots side by side and the report, walk the inventory and mark each item **Present / Missing / Differs / Extra**. An **Extra** (chrome or logo the mockup doesn't have) fails unless there's an explicit decision. Also check what the % can't see:

- Copy, word for word, against the copy doc.
- Exact brand colors: a similar gray is not the ink.
- The app's domain rules (e.g., a color reserved for a single meaning, values that must never be summed).
- Logic edges: undo, mixed results, boundary states.
- Accessibility: screen-reader labels, touch targets, focus.

Feedback goes as a **PR review, 2 rounds max**; after that the product owner decides. With design's OK written on the PR and checks green, the implementer merges per the project's merge rules.

## Rules

1. **Never raise a threshold to make something pass.** If it fails, fix the mockup or the app. Changing a threshold is a written decision, region by region, approved by the product owner.
2. **The mockup is the truth.** If you want to deviate, ask first. If accepted, the mockup is updated.
3. **A failure is not a verdict.** It may be an outdated mockup or different data. The verifier needs upkeep too: on a structural change, update the mockup first.
4. **Copy lives in docs** and the app copies it literally. New text is documented before it's verified.
5. **A "pass" doesn't approve on its own.** States, focus, hover and scroll don't fit in a screenshot.

## Native platforms (Android, iOS)

Rule: **platform patterns and standard components come before 100 % parity with the mockup.** On Android that means Material 3; on iOS, apply the same with the HIG and SwiftUI/UIKit components. The inventory marks each part as brand or platform.

**Brand (non-negotiable; blocks even if the % passes):** colors and tokens, typography, copy, amount and date formats, icons, which elements exist and their order.

**Platform (accepted drift):**
- Text line height and hinting (shifts of a few px).
- Shape, size and animation of system components: top bar, navigation bar, menus, snackbar, chips, sheets, dialogs, date and time pickers.
- Ripple and state layers, pull-to-refresh, transitions, predictive back or swipe-back.
- System insets and bars.
- Platform touch targets: 48 dp on Android, 44 pt on iOS.

**How it's applied:**
- Native thresholds are looser and set per screen in the matrix, with their reason. Example native thresholds (an Android list screen):

  | Region | Pass | Review |
  |---|---|---|
  | Screen | ≤ 2 % | ≤ 6 % |
  | Content | ≤ 1.5 % | ≤ 5 % |
  | Top bar | ≤ 1 % | ≤ 3 % |
  | Navigation | ≤ 0.75 % | ≤ 2 % |

- An accepted platform difference is carried into the mockup, not the threshold. E.g., the app's active-tab indicator was kept and the mockup was adjusted to match.
- **FAIL on native:**
  - An inventory element is missing or extra.
  - Different copy, color, typography, icon or amount.
  - Sections in a different order.
  - A platform deviation not listed in the inventory. Ask first; it is not assumed accepted.
- If a native pattern changes visible behavior (e.g., a search that expands to full screen), the product owner decides, and those pairs move to manual review.

## Chaining with ux-complexity-gate

1. `ux-complexity-gate` on the **mockup**. It goes first, because this gate would faithfully copy any usability problem in the design.
2. Implementation against the approved mockup.
3. **This gate** (fidelity).
4. `ux-complexity-gate` on the **implemented screen**.
5. Both green → merge.

If both fail, fix complexity first: it changes the mockup and forces fidelity to run again.
