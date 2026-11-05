# ux-complexity-gate · reference (optional)

## Automated measurement (web)

Stack: Playwright Test + `@axe-core/playwright` (+ `sharp` to annotate screenshots).

- **Interactive elements:** `a[href], button, input, select, textarea, [role=button|link|checkbox|radio|tab|switch|menuitem|option], [tabindex]:not([tabindex="-1"])`.
  - Count those that are visible, have no interactive ancestor, have their center inside the viewport and are not covered (`elementFromPoint`).
  - Before counting, temporarily set `pointer-events: none` on fixed/sticky elements.
- **Mockup contract** so the counts work:
  - Semantic elements.
  - `data-primary` on the primary action.
  - `data-group` on homogeneous rows or chips.
  - `data-bar` on the top bar and the navigation bar.
- **Per-check calculation:**
  - P1: elements with `data-primary`.
  - P2: distinct kinds (`data-group` or tag + class).
  - P3: interactive elements outside the bars.
  - P5 and P6: box size; axe `target-size` confirms.
  - P7: primary's center Y ≥ 40 % of the height.
  - P8: text nodes with `fontSize` < 12.
  - P9: axe `color-contrast`.
  - P10: has background, border, underline or link color.
  - P12: icon only and no `aria-label`.
- **Tasks (A3):** helpers wrapping `tap` and `fill`, always starting from Home, that count taps.
- **Graph (A2, A4, A5, A6):** BFS from `/` tapping every interactive element.
  - Records edges route → route or route#overlay.
  - Output: `graph.json` with depth, deadEnds and unreachable.

## Vision agent prompt

Input: screenshot with numbered marks on every interactive element, the list of marks (id, text, size) and the task that goes through the screen. Structured output. Run 3 times; keep findings that appear in ≥ 2 with the same rule and at least one shared mark.

```
You are a usability inspector. You receive ONE mobile screenshot with numbered marks on every
interactive element plus a list of marks. Evaluate ONLY what is visible. Cite mark ids in every
finding. Do not invent elements. If unsure, do not report. Score 0 fail / 1 partial / 2 pass:
R1 primary_action   Exactly one element reads as THE primary action.
R2 clickable_look   Every mark looks tappable.
R3 false_affordance Nothing non-interactive looks like a button, chip, link or input.
R4 icon_labels      Icon-only marks limited to back/close/search/more/add.
R5 hierarchy        Title → content → actions readable in under 3 s; one focal point.
R6 grouping         Related controls grouped; unrelated ones not adjacent.
R7 density          ≤ 5 distinct decision groups above the fold.
R8 disclosure       ≤ 2 levels of hiding.
R9 states           Loading, empty, error, success visible or implied.
R10 consistency     Same actions use the same label, style and position as reference screens.
R11 walkthrough     know_what / notice_action / associate / see_progress: yes or no each.
Severity per finding: 1 cosmetic, 2 minor, 3 major, 4 catastrophe.
```

- **Mapping:** R1 → P1, R2 → P10, R3 → P11, R4 → P12 (block with majority); R5 to R10 → REVIEW; R11 → P17.
- **Expected accuracy:** ~0.6 precision and ~0.4 recall per the literature. The agent flags; hard thresholds live in the automated measurement.
