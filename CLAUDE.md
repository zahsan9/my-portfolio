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

`setRoute(name)` shows/hides full-page `.page` sections by their `data-route` attribute. All state is in-memory — there is no URL hash. Nav buttons carry `data-route`; the logo always routes to `feed`.

### Feed cards

Cards are `.pin` elements with `data-cat` (for filter tabs) and `data-page` (sheet key). Clicking calls `openSheet(key, srcEl)`. A JS snippet at page load moves any `.sub` inside `.art` out to be a direct child of `.pin` so it overlays below the card on hover.

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

**Feed card palette (theme-independent — defined once in `:root`, never overridden):** `--c-clay`, `--c-blue`, `--c-cream`, `--c-yellow`, `--c-sage`, `--c-plum`, `--c-sand`, `--c-charcoal`, `--c-noir`, `--c-noir-2`, `--accent`

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

## Case study writing guidelines

Applies to any `PAGES[key].custom` write-up (the `cs-` component system). `PAGES.climate` (ESD Family Survey) is the reference implementation — copy its structure for new case studies. These rules come from how design-hiring reviewers actually read portfolios: they spend 3-5 minutes per case study (some reject in 30 seconds), and they're grading decision quality and measurable outcomes, not process narration.

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
