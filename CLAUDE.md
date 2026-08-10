# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project files

| File | Purpose |
|---|---|
| `portfolio.html` | Everything: all CSS (inline `<style>`), all JS (inline `<script>`), all HTML |
| `assets/images/` | Feed card art, case-study screenshots, icons |

**Do not** introduce npm, a bundler, or a framework. This project is intentionally dependency-free and runs with `python3 -m http.server 8080` or directly as `file://`.

---

## Architecture

### Routing

`setRoute(name)` shows/hides full-page `.page` sections. Routes are reflected in the URL hash (`#experience`, `#about`); an empty or unrecognized hash resolves to `feed`. Rail buttons carry `data-route`; the ZA logo always routes to `feed`. Header links may use these hashes directly, as with the Experience shortcut.

### Feed cards

Cards are `.pin` elements with `data-cat` (for filter tabs) and `data-page` (sheet key). Clicking calls `openSheet(key, srcEl)`. A JS snippet at page load moves any `.sub` inside `.art` out to be a direct child of `.pin`. By default `.sub` is in-flow (`max-height: 0` → expands on hover/focus), so revealing a card's caption pushes the card below it down within the same column — this is intentional, not a layout bug; `overflow-anchor: none` on `.main` (see the comment near the top of the stylesheet) exists specifically to stop Chrome's scroll anchoring from fighting this on hover near the bottom of the feed. The one exception: a pin sitting directly above a static, non-case-study component (`.pin[data-page]:has(+ .pin:not([data-page])) .sub`, e.g. before "Find me elsewhere") renders its `.sub` as an absolute overlay instead, so that specific component card never gets pushed. Do not flatten this back to a universally-absolute `.sub` — that silently kills the push interaction for every other card and has regressed before.

Because that push growth is real layout growth, it would also grow `.feed`'s own auto height and bounce the in-flow copyright footer beneath it — which the "card hover states cannot move it" rule (see Navigation and shared chrome below) forbids. `lockFeedHeight()` (defined next to `applyFilter` in the Filter + FLIP script section) pins `.feed` to a fixed measured height at rest, re-measuring only on real layout changes (`window load`/`resize`, after a filter's FLIP settles, on `setRoute('feed')`, and inside `flipPins` for sheet open/close) — never on hover. `.feed`'s `120px` bottom padding is the headroom that growth pushes into instead of the footer.

Filter tabs match `data-filter` against `data-cat`. Visibility changes use a FLIP animation (snapshot positions before/after, then transition with `translate`).

### Sheet panel

`openSheet(key, srcEl)` reads from the `PAGES` object and writes HTML into `#sheetBody`. If `PAGES[key].custom` exists, that string is used verbatim (full custom layout); otherwise the default template renders `eyebrow`, `title`, `lead`, `facts`, a figure placeholder, and `body`.

The app shell gains `.sheet-open` on open, triggering: rail hides, topbar hides, tabs reflow into a fixed header bar, feed shifts left by `--panel-w` (68vw desktop, 100vw on small screens). A FLIP animation repositions the active pin.

### Experience timeline

The experience section (`#expView`) is a scroll-driven horizontal track. A sticky sentinel + `IntersectionObserver` drives `updateExperienceTrackNow()` which reads scroll position and sets `--exp-track-x` on `.exp-scroll` to translate `.exp-track`. 

End-of-scroll: `endHold = viewport.clientHeight * 0.55` — a buffer where the card deck fades out (smoothstep opacity + translateY) before the sticky viewport releases. Cards individually fade in via `--scroll-reveal-opacity` as they enter the track.

### Theme

`data-theme="dark"` on `<html>` drives all color tokens via CSS custom property overrides in `[data-theme="dark"]`. Toggling adds `.theme-transitioning` for a 520ms crossfade, then removes it.

`:root` holds the values dark mode actually ships with (`--paper`, `--ink`, `--c-cream`, etc. — dark is treated as the source of truth, not light). The `[data-theme="dark"]` block only overrides page-chrome tokens (paper/ink/line/control); it deliberately does **not** redefine the feed-card palette (`--c-*`) — those must render identically in both themes. `--sub-color` (used by `.pin .sub`, the feed-card hover caption) is the one token that's genuinely different per theme: black in light, white in dark, since it sits on the page background rather than a card surface.

### Site cursor

A custom cursor (`#site-cursor`, injected by JS, `pointer: fine` only) replaces the system cursor and morphs into a labeled pill via `.is-pill` when hovering `[data-cursor-link]` elements (external links, mailto) or grows via `.is-hover` over generic interactive elements. Position updates are rAF-batched off `mousemove`.

### Skill pill magnetic repulsion

The "In the overlap" feed card's pills (`.skill-cloud .skill-pill`) rest at their normal grid layout and spring away from the cursor when it's within `PHYSICS.repelRadius`, with pairwise AABB collision resolution so pushed pills never overlap each other — search "SKILL PILL STORYBOARD" for the full physics IIFE. All tunable values live in one `PHYSICS` config object at the top of that block.

## Key globals (in `portfolio.html` inline `<script>`)

| Symbol | What it is |
|---|---|
| `PAGES` | Object keyed by card slug. Each entry has `meta`, `eyebrow`, and either `custom` (raw HTML string) or `title/lead/facts/body`. |
| `FIG_BG` | Map of `data-cat` → background color for the placeholder figure in default sheet layouts. |
| `setRoute(name)` | Switches active page view. |
| `openSheet(key, srcEl)` | Populates and opens the sheet panel. |
| `closeSheet()` | Closes the panel and restores layout. |
| `updateExperienceTrackNow()` | Recalculates horizontal scroll position and card reveal state. |

---

## Design tokens (CSS custom properties)

**Page chrome (theme-dependent — flips between `:root` and `[data-theme="dark"]`):** `--paper`, `--paper-2`, `--ink`, `--ink-mid`, `--ink-low`, `--line`, `--control-soft`, `--control-muted`, `--accent-soft`, `--sub-color`

**Feed card palette (theme-independent — defined once in `:root`, never overridden):** `--c-clay`, `--c-blue`, `--c-cream`, `--c-yellow`, `--c-sage`, `--c-plum`, `--c-sand`, `--c-charcoal`, `--c-noir`, `--c-noir-2`, `--accent`. `--c-plum` is a muted lavender (`oklch(0.71 0.06 305)`), lighter and softer than the old saturated plum but darkened back toward `--c-sage`/`--c-clay`'s lightness so it doesn't read as washed out next to them — don't push it lighter than ~0.72 or more saturated than ~0.07 chroma. `--c-clay` doubles as the "coral" tone (used by the Daily type & motion card and the About page's Design skill pill) — there is no separate `--c-coral` token, don't add one.

**Card content colors (also theme-independent):** `--card-ink`, `--card-muted`, `--card-paper` — used for text/labels drawn on top of the `--c-*` card backgrounds, regardless of page theme.

**Motion:** `--motion-fast` (180ms), `--motion-med` (420ms), `--motion-slow` (480ms), `--ease-out` (cubic-bezier 0.22,1,0.36,1), `--ease-soft` (cubic-bezier 0.19,1,0.22,1)

**Fonts:** `--serif` (Instrument Serif), `--sans` / `--mono` (Inter)

**Layout:** `--chrome-size: 80px` (rail width *and* topbar height — kept as one token on purpose, don't split it), `--panel-w: 68vw` (sheet panel width, `100vw` on small screens), `--open-column-gutter: 24px`

**Route transitions (tunable via DialKit if added):** `--rt-fade-out`, `--rt-slide-y`, `--rt-child-dur`, `--rt-stagger-base`, `--rt-stagger-step`

---

## CSS class conventions

| Prefix | Used for |
|---|---|
| `cs-` | Case study sheet content components |
| `ab-` | About page sheet content |
| `exp-` | Experience timeline components |
| `art-*` | Feed card color variants (`art-clay`, `art-sage`, `art-charcoal`, etc.) |
| `pin` | Feed card (`.pin`, `.pin-active`, `.pin-meta`, `.sub`) |

---

## Navigation and shared chrome

- **Compact rail**: the closed desktop rail is exactly `--chrome-size` (`80px`), matching the topbar's explicit height. Do not split those dimensions or widen the compact state independently.
- **Rail reveal**: outside the one-time entry storyboard below, the rail stays compact and expands only when the navigation is hovered or keyboard-focused (`:focus-within`).
- **Work rail entry storyboard**: `playWorkRailEntry()` (defined next to `updateFeedScrollbar`, called from the hash-routing block only when `initialRoute === 'feed'`) briefly auto-expands the rail (`.is-intro-expanded`, same CSS as hover/focus-within) 180ms after landing on Work, then auto-collapses it at 2380ms — a one-time "there's navigation here" reveal, not a persistent state. It fires on true entry into Work (first load or a refresh while on Work), desktop widths only (`min-width: 801px`), and respects `prefers-reduced-motion`. It must never fire from in-app navigation back to Work (rail click, logo, Back to Work) — only `setRoute()` handles those, and `setRoute()` never calls it. This exact feature has been silently deleted before while "refining navigation" (the CSS class survived, the JS didn't) — don't let that happen again.
- **Expanded width**: the rail expands only to `168px`, which is the smallest width that gives the Experience pill adequate right padding. Do not restore the older `192px` width without a demonstrated need.
- **Rail labels**: the primary route is named **Work** in the rail even though its internal route key is `feed`. The other labels are Experience and About Me.
- **Header**: the topbar no longer carries a contact/utility nav (the Experience/Email/LinkedIn text links were removed); it exists only for the scroll-shadow chrome (`.topbar::before` / `.scrolled`) and to match the rail's `--chrome-size` height. Email, LinkedIn, GitHub, and Instagram are reachable via the social tiles in the feed banner and the Elsewhere card.
- **Return controls**: About and Experience each show a quiet outlined **Back to Work** button beneath the introductory title. Both call `setRoute('feed')`.
- **Footer**: the copyright is a centered, in-flow footer at the bottom of the Feed view, never fixed to the viewport. Current copy is `© 2026 Zainab Ahsan, All Rights Reserved`. Keep it low contrast and ensure card hover states cannot move it.
- **Mobile**: the desktop rail remains hidden at `800px` and below. Do not introduce rail-only information that becomes unavailable on mobile.

---

## Case study and report design system

`PAGES.climate`, `PAGES.brew`, and `PAGES.crow` are the reference implementations. They share one reading system but represent different kinds of work:

- **Analytical report (`climate`)**: evidence-led, finding-first, data and figure heavy.
- **Product-design case study (`brew`)**: problem-led, ownership-first, interaction and iteration heavy.
- **Client case study (`crow`)**: brew's product-design grammar (measured section rhythm, `cs-product-principles`, framed `cs-fig` figures with real screenshots instead of an iteration player) applied to a real client redesign — WIIRED Studio × UWB CROW Journal, a WIIRED Studio partner project (partner unnamed by request). Its palette is lifted directly from `the-crow/README.md`'s documented design tokens (`--dark:#191331`, `--ink:#2d2357`, `--purple:#43396d`, `--cream:#f5f0e8`), scoped under `.sheet[data-page="crow"]` the same way brew scopes its cream/coffee palette — do not invent a different portfolio-only palette for it. Every fact in its copy (repo, Figma, demo link, contributors, timeline, the still-unmigrated `uwbcrow.com` production domain) is verified from `the-crow/README.md` and its git history, not invented — keep it that way if this entry is edited.

Do not make every sheet visually identical. Consistency comes from the canvas, spacing, hierarchy, figure treatment, and interaction patterns. The accent palette and content structure should still communicate the kind of work being presented.

### Shared visual foundation

- **Case-study canvas**: all finished case studies use the same mode-aware sheet background. Dark mode uses the lighter neutral gray `oklch(0.29 0 0)`; light mode uses the darker neutral gray `oklch(0.91 0 0)`. Keep the shared selector centralized so case-study backgrounds cannot drift apart.
- **Section rhythm**: use `66px` of top spacing between major `.cs-section` blocks. The space belongs before the next text section, not as extra space after a figure. Every major section should use the same rhythm, including Impact/Current Outcome.
- **Section labels**: title case, bold sans, compact rounded rectangle, and visually smaller than the content heading. Labels are not all-caps mono pills. Current size is `15px` for Brew and `16px` for Neutral.
- **Structural boxes**: use a subtle fill with no outline and no shadow. Neutral and Brew section containers should not look like bordered cards. Borders are reserved for true content boundaries, such as figures, tables, version controls, or input-like elements.
- **Shadows**: case-study content does not use decorative shadows in either theme.
- **Typography**: use bold sans for direct product/report hierarchy and serif selectively for editorial questions or decision headings. Do not use pill typography for data rows or major titles.
- **Copy**: do not use em dashes. Prefer a period, comma, colon, or parentheses.
- **Figures**: every image sits in a stable frame and has a caption that explains what the evidence demonstrates. Captions use a quiet light-gray family, not the accent color. The graph/screenshot itself keeps its intended background; only the caption surface changes.
- **Zoom control**: magnifying-glass controls are black with a white icon. Only hovering or focusing the control expands it to “Zoom”; hovering the whole figure must not trigger it.
- **Top-right CTAs**: external project actions belong in the hero’s top-right, as with Neutral’s Tableau CTA and Brew’s Repo/Figma pair. Use real destinations only. Never invent or point a CTA to a generic homepage.
- **Light and dark mode**: test both explicitly. Brew’s light-mode accent is pale warm ivory (`--brew-cream: oklch(0.965 0.018 85)`), not saturated beige. Dark mode may use a warmer cream, but both modes retain the shared neutral canvas.

### Analytical report variant (`PAGES.climate`)

- Lead with the finding, not the artifact name.
- Hero facts may use a three-column stat row when those figures help a reviewer understand scale immediately.
- Report sections typically follow: Context, The Ask, Data + Method, What I Found, Analytical + Design Decisions, Impact, and reflection.
- Data tables and figure frames may use subtle boundaries because they are evidence containers, not decorative section cards.
- “What I Found” and Impact may use a solid yellow break in the gray rhythm. Keep text black and use clean bullets for impact details.
- Numbered analytical decisions align to one vertical text guide regardless of whether an alternating row has a fill.
- Figure references are real links. Clicking a reference smoothly scrolls to the target figure with sticky-header offset.
- Tableau and similar report CTAs stay visible in the hero.

### Product-design case study variant (`PAGES.brew`)

- Start with product premise, current status, role/team framing, and build context.
- Keep Stack + Ownership near the top when implementation is materially part of the work, but do not let the stack dominate the rest of the story.
- Preferred narrative order: hero, Stack + Ownership, Product Context, Product Principles, product flows, Current Outcome, Next Iteration.
- Product flow section names should be direct and title case. Use “Comment Section,” “Onboarding,” “Splash + Loading,” and “Directions,” not vague headings such as “Design Work.”
- State collaboration precisely. Brew is a partner project and broader product direction is shared. The case study separately identifies the flows the author independently owns. Never claim sole ownership of shared work or completed ownership of work that is still WIP.
- WIP flows use honest placeholders and statuses such as “Not yet designed” or “Work in progress.” Preserve final layout space without presenting unfinished work as complete.
- Product accents can differ from the report palette. Brew uses cream/coffee warmth over the shared gray canvas; it should not inherit Neutral’s yellow report blocks.
- Current Outcome states what presently works. Unresolved bugs and missing actions belong under Next Iteration, not as negative annotations on the final screenshot.

### Iteration players

Use an annotated click-through player when several screenshots show the same surface evolving. Do not line up many near-identical screenshots.

- Keep one stable screenshot frame and morph between versions with a restrained opacity/position transition.
- The user selects versions with circular `V1`, `V2`, etc. buttons. Inactive versions are outlined; the active version is filled. Buttons must look clickable without an additional “Select a version” label.
- Support click, keyboard focus, and Left/Right Arrow navigation. Update `aria-pressed`, roving `tabIndex`, active image, `aria-hidden`, and active annotation together.
- Respect `prefers-reduced-motion`; version changes become effectively immediate.
- Put the player category in the top-right of the container using literal labels such as “Reviews + ratings,” “Reply threads,” and “Comment card.”
- Use an explicit, on-the-nose main title that names the UX change, for example “Preventing accidental 0/5 ratings and duplicate reviews.” Avoid generic titles such as “Refining the experience.”
- Annotation text uses primary text color for contrast, not gray-on-gray. Use a soft green `+` for improvements and a soft red `−` for unresolved problems. Do not place annotation lines inside pills or outlined mini-cards.
- The final version contains only positive annotations. Move remaining problems to Next Iteration.
- Order Brew’s interaction stories as: Reviews + ratings, Reply threads, then Comment card. DOM order, visual order, and keyboard order must match.
- Use the author’s actual critique notes. Do not infer or fabricate a rationale when the real iteration notes are available.

### Motion and accessibility

- All case-study motion must honor `prefers-reduced-motion`, including JavaScript-driven transitions and scroll behavior.
- Animated state changes must be interruptible. A new version click should replace the current transition immediately rather than queueing.
- Controls need visible hover and keyboard-focus states. Do not rely on hover alone to reveal essential meaning.
- Case-study images need specific alt text describing the state or change shown. Decorative placeholders use `aria-hidden="true"`.
- Keep inactive iteration screenshots out of the accessibility tree with `aria-hidden="true"`.

---

## Case study writing guidelines

Applies to any `PAGES[key].custom` write-up (the `cs-` component system). Choose the analytical-report or product-design structure above rather than copying one project mechanically. These rules come from how design-hiring reviewers actually read portfolios: they spend 3-5 minutes per case study (some reject in 30 seconds), and they're grading decision quality and measurable outcomes, not process narration.

### Structure, in order

1. **`cs-hero` title** — states the finding or twist, not just the project name ("What 22% Neutral Hid.", not "Neutral Response Analysis"). `cs-hero-sub` tells the whole story in 1-2 sentences: context → tension → what you built.
2. **`cs-role-line`** — Solo/Team, client, year. This sets the "I" vs "we" frame for everything below it (see Rules).
3. **Scale numbers** — fold 1-2 real scale figures into the `cs-hero-sub` sentence itself ("a climate survey of ~6,000 families across 35+ schools") rather than adding a separate `cs-stat-row` block. `cs-hero-sub` + `cs-role-line` is already two stacked pieces under the title; a third block (stat row) reads as cluttered. Reserve `cs-stat-row` for heroes that have no `cs-role-line` (see the WIIRED hero) — never stack both.
4. **"The Question"** (`cs-section` + `cs-question` + `cs-copy`) — the problem, framed as a real ambiguity, plus the constraint that made it hard (data scale, conflicting audiences, a deadline).
5. **"Data + Approach"** (`cs-two-col` / `cs-feature`) — source and transforms, 2-4 sentences each. This is the *only* section where methods belong — don't repeat tool/technique names elsewhere.
6. **"Design Decisions"** (`cs-decision-block`, numbered, 3-5 of them) — each one names the option considered, why it was rejected, and the insight that drove the final call. Insight over technique: never state a choice ("used a treemap") without the reason ("bar chart implies false equivalence between a 30- and 900-respondent school").
7. **"Impact"** (`cs-impact-callout`) — lead with the sharpest *true* outcome. Use `cs-impact-n` when a real metric exists. When it doesn't, don't invent one — `cs-impact-line` should still name a concrete, verifiable event (sent, shipped, adopted), never something that didn't actually happen. A missing or vague impact line is the single most common rejection reason in the research this is based on, but a fabricated one is worse — get the facts from the user rather than inventing specifics (meetings, presentations, collaboration) that sound plausible.
8. **"What I'd Change"** (`cs-learnings`) — keep this in every case study. Specific self-critique (name the exact chart, the exact tradeoff) reads more senior than a project with no acknowledged flaws.

### Rules

- **Never invent facts** — collaboration stories, meetings, stakeholder feedback, prototypes, or outcomes that didn't happen. If the real story isn't known, ask before writing it in. This applies even when the invented version would score better against the checklist above — an honest, modest case study beats a fabricated impressive one.
- **"I" vs "we"** — `cs-role-line` sets the frame. If it says Solo, the whole write-up uses "I." Collaborators get named with their role; "we" only for decisions genuinely made together.
- **No method-dumping** — tool/technique names belong only in "Data + Approach." Every other section states an insight or a decision.
- **No screenshot without context** — every `cs-preview-frame` / `cs-decision-fig` needs a `cs-caption` that states the finding the image proves, not just a figure sitting alone.
- **Scale over safety** — lead with what made the problem hard; don't bury a real constraint (respondent count, edge cases, conflicting stakeholders) in a middle paragraph.
- **2-3 finished case studies, not more** — everything else stays `wip: true` (renders the "Coming soon" state). A placeholder reads better than a rushed write-up.
- **Reads as a talk** — the section order above is the talk order: hook, question, decisions with reasoning, impact, reflection. Amazon-style loops expect this to work presented out loud, not just skimmed.
