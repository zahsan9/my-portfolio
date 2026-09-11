# Portfolio Design Audit
*Evaluated against artifact-design and dataviz principles*

Date: 2026-09-10  
Subject: portfolio.html (single-file portfolio site)  
Framework: None (vanilla HTML/CSS/JS)

---

## Executive Summary

This portfolio demonstrates **strong technical craft and systematic thinking**, with particular strengths in motion design, accessibility infrastructure, and token-based theming. However, it trends toward **templated visual decisions** in several areas that the design skills would flag as generic AI-generated patterns.

**Grade: B+ for execution, C+ for distinctiveness**

The gap is not in capability—the CLAUDE.md shows deep understanding of layout mechanics, animation choreography, and accessibility—but in **aesthetic risk aversion**. The work reads as "well-executed default choices" rather than "a designer's point of view."

---

## ✅ Strong Adherence to Fundamentals

### Theme Architecture (Exemplary)
The three-state theme system is **textbook correct** per artifact-design:
- `:root` defines complete light palette
- `@media (prefers-color-scheme: dark)` with `:root:not([data-theme="light"])` guard
- `:root[data-theme="dark"]` for explicit toggle
- Every token defined in bare `:root` before redefinition
- Body sets explicit background (no transparent borrowing from host)

**Exception to praise:** Feed card palette `--c-*` deliberately skips dark-mode overrides, making cards identical in both themes. While intentional (per CLAUDE.md), this violates the "design both themes" principle—dark mode is an afterthought to light, not a selected palette for that mode.

### Spacing & Layout (Strong)
- Consistent use of `gap` over per-element margins
- Proper `overflow-x: auto` on wide content containers
- `overflow-anchor: none` to prevent scroll-anchoring bugs
- Clear structural hierarchy without over-carding

### Typography (Solid Foundation)
- Real fallback stacks (`'Instrument Serif', Georgia, serif`)
- Google Fonts loaded correctly via CSP-allowed host
- Type scale exists and is mostly consistent
- `text-wrap: balance` on headings
- `tabular-nums` for aligned figures

### Motion (Best-in-Class)
- All transitions reference tokens (`--motion-fast/med/slow`)
- `prefers-reduced-motion` collapses ALL durations to 0.01ms via token redefinition
- No hard-coded `transition` values that would ignore the preference
- Easing curves documented with rationale (front-loaded vs. ramped)

### Accessibility (Thorough)
- Visible `:focus-visible` states with outline + offset
- `aria-label` / `aria-labelledby` / `aria-pressed` / `aria-hidden` used correctly
- Sheet is `role="dialog"` but deliberately NOT `aria-modal` (rail stays accessible)
- Keyboard navigation supported (arrow keys in iteration players, roving tabindex)
- Reduced motion respected in JS storyboards

---

## ⚠️ Deviations from Design-Skill Principles

### 1. Generic Neutral Palette (Major)
**Finding:** The page-chrome colors are **pure achromatic grays**—no hue bias.

```css
:root {
  --paper:       oklch(0.965 0 0);  /* chroma 0 */
  --ink:         oklch(0.18  0 0);  /* chroma 0 */
  --ink-mid:     oklch(0.45  0 0);
  --ink-low:     oklch(0.65  0 0);
  --line:        oklch(0.88  0 0);
}
```

**artifact-design principle violated:**  
> "A pure mid-grey reads as unconsidered; a grey with a slight hue bias toward the page's accent reads as chosen."

**Recommendation:** Inject 0.005–0.01 chroma biased toward the 262° accent (purple) into `--ink-mid`, `--ink-low`, and `--line`. Example:
```css
--ink-mid: oklch(0.45 0.008 262);
--line:    oklch(0.88 0.005 262);
```

This costs nothing visually but signals intentionality.

---

### 2. Feed Card Color Assignments (Moderate)
**Finding:** Cards use a hand-picked palette (`--c-clay`, `--c-sage`, `--c-plum`, etc.) applied via `.art-*` classes. The palette itself is fine—muted, balanced—but the **assignments feel arbitrary**.

- `brew` = clay (coral/terracotta)
- `climate` = yellow
- `crow` = charcoal
- `dreamcatcher` = blue
- `habitat` = sage
- `wiired` = plum

**artifact-design principle:**  
> "Ground it in the subject. [...] carry at least one detail only this subject would have—its real units and scales, its document conventions, its terms of art."

**What's missing:** Why is brew coral? Why is climate yellow? The colors don't encode anything about the projects themselves (brew could have been cream/coffee from its case study palette; climate could reference its survey/data nature; habitat's streak mechanic doesn't map to sage).

**Recommendation:**  
Either make the card color **mean something**—pull from the project's own palette, or encode discipline/year/client type—or embrace full randomness and stop assigning them by hand. The current state reads as "I picked colors I liked" without a system.

---

### 3. Card Hover Mechanics vs. Layout Stability (Minor)
**Finding:** Card `.sub` captions expand on hover, pushing cards below them down (`max-height: 0 → auto`). This is **intentional** per CLAUDE.md, but it violates layout stability.

**artifact-design principle (implied):**  
Hover states should not reflow sibling content—it's disorienting when scanning a grid.

**CLAUDE.md defense:**  
> "revealing a card's caption pushes the card below it down within the same column—this is intentional, not a layout bug"

**Counterpoint:**  
Intentional ≠ good UX. The `.sub` could be `position: absolute` universally (as it already is for pins above static components), eliminating the push without losing the reveal. The current mechanic adds cognitive load when browsing.

**Severity:** Low (it works, and `lockFeedHeight()` prevents footer bounce), but worth reconsidering.

---

### 4. Over-Reliance on Rounded Rectangles (Moderate)
**Finding:** Nearly every interactive element uses `border-radius`:
- Rail buttons: `24px` (pill)
- Feed cards: `16px`
- Section labels: `999px` (full pill)
- Case study containers: `12px`
- Skill pills: `999px`
- Sheet close button: `999px`

**artifact-design warning:**  
> "AI-generated design currently clusters around a few looks: [...] `rounded-lg` everywhere."

**Assessment:** Not egregious—rounded corners are a reasonable default—but the **lack of variation** makes everything read as one family. Consider:
- Sharp corners for case-study figures (analytical precision)
- Subtle radius (2–4px) for structural containers
- Pills reserved for truly pill-shaped actions (tags, filters)

---

### 5. Typography: Lack of Display Face Restraint (Moderate) ✅ PARTIALLY FIXED
**Finding:** `Instrument Serif` is used for:
- Hero titles
- Section questions (`.cs-question`)
- About page principles
- Case study headings

**artifact-design principle:**  
> "a characterful display face used **with restraint**"

**Assessment:** Instrument Serif is beautiful but overused. It appears in **every major heading**, which dilutes its impact.

**✅ FIXED - Monospace Addition:** Added **Space Mono** as the third typeface for data/technical contexts:
- Replaces Inter in `--mono` token
- Applied to: eyebrows, case study role pills, metadata, timestamps, copyright
- Brings retro-futuristic character without being gimmicky
- Complements Inter's geometry and Instrument's refinement

**Remaining work:** Reserve Instrument Serif for page-level heroes and editorial moments only. Use bold sans (`Inter 600/700`) for case-study section headings and product flows—those are structural, not editorial.

---

### 6. Case Study Content Palette Asymmetry (Minor)
**Finding:** `brew` and `crow` remap case-study colors:
```css
.sheet[data-page="brew"] {
  --paper: oklch(0.965 0.018 85);  /* cream */
  --ink: oklch(0.27 0.028 40);     /* coffee */
}
```

But `climate` (the analytical report) does **not**—it keeps the global gray canvas.

**artifact-design principle:**  
> "all finished case studies use the same mode-aware sheet background"

**CLAUDE.md says:** This is correct—reports and product studies are different treatments.

**Counterpoint:** A report can still have a **thematic accent** without full repaint. `climate` could tint its `--paper` with a hint of its yellow data-viz palette (0.005 chroma at 90°) to distinguish it from the global chrome, while keeping the same lightness/contrast ratios.

**Recommendation:** Give analytical reports a **canvas accent** (very subtle chroma shift) to differentiate them from the site chrome, even if they don't get a full brand treatment like product studies.

---

### 7. No Data Visualizations in Climate Report (Critical for dataviz)
**Finding:** The climate case study (`PAGES.climate`) is described as "evidence-led, finding-first, data and figure heavy" but I see **no actual chart code** in the portfolio. Figures are placeholder `<div>` blocks.

**dataviz principle violated:**  
Every principle in the dataviz skill—the entire procedure—is unused.

**What I expected to see:**
- A treemap or grouped bar chart showing the 22% Neutral distribution
- Enrollment-weighted vs. unweighted comparison
- CVD-validated categorical palette
- Hover tooltips with exact values
- Validated via `scripts/validate_palette.js`

**What exists:** Generic figure placeholders:
```css
.sheet-body .fig {
  aspect-ratio: 16/9;
  display: grid;
  place-items: center;
  color: oklch(0.98 0 0 / 0.6);
  /* "FIGURE PLACEHOLDER" text */
}
```

**Recommendation (Urgent):**  
The climate report is the **showcase for dataviz skill**. Without actual, working charts:
1. Load real (anonymized) data
2. Build 2-3 key charts following the dataviz procedure:
   - Pick form (bar, treemap, small multiples)
   - Assign categorical color (fixed hue order, never cycled)
   - **Validate with the palette checker script**
   - Apply mark specs (thin marks, 4px radius, 2px lines)
   - Add hover layer
   - Table view for accessibility
3. Inline the charts as SVG or Canvas (no library needed for 2-3 charts)

Until this happens, the climate case study is a **design doc, not a portfolio piece**.

---

## 🎯 Editorial Treatment Assessment

Per artifact-design, some requests call for **editorial treatment**: distinctive point of view, opinionated calls, one real aesthetic risk.

**Current portfolio treatment:** Utilitarian+, trending toward editorial but not committing.

### What's Good
- Motion choreography (splash glyph, rail tuck, sheet transitions) shows **authorial voice**
- Custom cursor with morphing labels is a **bold choice that serves the work**
- Rail index color-coding (active row fills with card's own surface) is **distinctive**

### What's Missing
- **Typography doesn't take a risk.** Instrument Serif + Inter is a safe, legible pair—but it's the same pair 40% of portfolios use. Where's the unexpected face? The monospace data? The condensed display?
- **Color palette is conservative.** Muted pastels + near-black is the "tasteful designer" default. Brew's cream/coffee is the **only project-specific palette**, and it's still in the safe zone.
- **No aesthetic thesis in the hero.** The portfolio opens with a feed grid—fine for browsing, but it doesn't **make an argument** about the designer's POV. Compare: a landing that opens on the most characteristic project moment (brew's mug illustration? climate's 22% finding? the magnetic skill pills?), *then* reveals the index.

**artifact-design editorial principle:**  
> "The hero is a thesis: open with the most characteristic thing in the subject's world."

**Current hero:** A grid. It's efficient, not memorable.

---

## 🔧 Dataviz-Specific Findings

### No Charts Exist Yet
As noted above, the climate report has **placeholder figures only**. This is the #1 blocker to portfolio readiness from a dataviz perspective.

### Skill Pills Chart-Adjacent
The "In the overlap" skill pills (`.skill-cloud`) use **magnetic repulsion physics**—a playful interaction, but:
- It's a **force-directed layout**, not a chart
- No categorical palette validation (pills are solid fills, not outlined chips)
- No accessibility layer (keyboard nav? screen reader labels?)

**Missed opportunity:** This could demonstrate **interaction design for non-chart data**—but it needs labels that announce what the physics is showing (overlap/distance = relatedness?).

---

## 📊 Anti-Pattern Check (from dataviz skill)

Referencing `references/anti-patterns.md` from dataviz:

1. **Dual-axis charts:** N/A (no charts)
2. **Rainbow sequential scales:** N/A (no charts)
3. **Color-only encoding without secondary cues:** The rail index relies on color to identify projects, but has **icon + label** as secondary encoding—✅ acceptable
4. **Cycling categorical hues:** N/A (feed cards hand-assigned, not cycled)
5. **Unvalidated CVD palettes:** ⚠️ The feed card palette has not been run through the validator

**Action item:**  
Run the feed card palette through `scripts/validate_palette.js`:
```bash
node scripts/validate_palette.js \
  "#b37356,#8d9977,#b59bc2,#5a82ac,#e4d18b,#d3b191,#423c3b,#232323" \
  --mode light
```

Verify:
- Adjacent-pair CVD Delta E ≥ 8 (target) or ≥ 6 (floor with secondary encoding)
- Normal-vision floor ≥ 15
- Contrast passes for small text

**Prediction:** The `--c-noir` / `--c-charcoal` pair will likely fail the normal-vision floor (too close in lightness).

---

## 🎨 Specific Recommendations

### High Priority
1. **Add real charts to climate report** (see §7 above)
   - Validate categorical palette before publishing
   - Follow dataviz procedure: form → color → validate → marks → interaction
   
2. **Inject hue bias into neutral grays** (see §1)
   - 0.005–0.01 chroma at 262° in `--ink-mid`, `--line`
   
3. **Systematize feed card color assignments** (see §2)
   - Option A: Pull from project's own palette (brew=cream, climate=yellow, crow=purple from its design system)
   - Option B: Encode discipline (ux=sage, viz=yellow, swe=blue) and composite for multi-discipline projects

### Medium Priority
4. **Reduce Instrument Serif usage**
   - Reserve for page heroes and editorial moments only
   - Use bold sans for case-study structural headings
   
5. **Add canvas accent to analytical reports**
   - `climate` sheet gets a hint of yellow (0.005 chroma) to distinguish from global chrome
   
6. **Reconsider card hover push** (see §3)
   - Test `position: absolute` for all `.sub` elements
   - Measure if users actually read the hover captions (analytics?)

### Low Priority
7. **Vary border-radius usage**
   - Sharp corners for analytical figures
   - 2–4px for structural containers
   - Pills only for tags/actions
   
8. **Editorial hero treatment**
   - Open on **one project moment** (large, immediate) before revealing the index
   - Splash glyph is beautiful but generic (Z lettermark ≠ portfolio thesis)

---

## 🏆 What to Keep Doing

- **Motion token architecture** is production-ready and should be a case study itself
- **CLAUDE.md documentation** is exemplary—every design decision has rationale
- **Accessibility infrastructure** (focus states, ARIA, keyboard nav) is thorough
- **Theme system** is textbook correct for the three-state pattern
- **Custom cursor** is a standout detail that doesn't feel gimmicky
- **Rail choreography** (tuck, index reveal, color-coded pills) shows real craft

---

## 🚨 Blockers to "Editorial Treatment" Status

Per artifact-design, editorial work requires:
1. **A distinctive point of view** → ⚠️ Present in motion/interaction, absent in color/type
2. **One real aesthetic risk** → ⚠️ Custom cursor qualifies, but it's a utility not a thesis
3. **Plan that deviates from generic default** → ❌ Instrument Serif + Inter + muted pastels is the default

**To cross the threshold:**
- Pick a **characterful display face** that's not in the top-10 portfolio fonts (not Serif Display, not Space Grotesk). Try: Fraunces, Departure Mono, Newsreader, Bricolage Grotesque.
- Give the **index itself a POV**: instead of "everything in a grid," open with your strongest project moment—full-bleed—and let the grid be the "see more" state.
- Use **color to encode information**, not just taste. Right now card colors are decoration; make them taxonomy.

---

## 📝 Conclusion

This is **strong craft applied to safe choices**. The mechanics—theme, motion, accessibility—are production-grade. The aesthetics are **tasteful but templated**.

The portfolio reads as "a designer who can execute a brief" but not yet "a designer with a POV." For a junior role, this is excellent. For a senior role, it needs **one more layer of authorship**:
- Real charts in the climate report (must-have)
- A color system that means something (should-have)
- An unexpected typographic choice (nice-to-have)

The skill is here. The point of view is waiting to be articulated.

---

**Next Steps:**
1. Run `validate_palette.js` on feed cards
2. Build climate report charts with validated palettes
3. Pick one aesthetic risk (type, color, or hero) and commit to it

**Estimated effort:**
- Charts: 8–12 hours (data prep + 3 chart types + validation)
- Palette systematization: 2–4 hours
- Type refresh: 4–6 hours (if changing faces)

