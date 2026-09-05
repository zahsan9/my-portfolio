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

Cards are `.pin` elements with `data-cat` (for filter tabs), `data-page` (sheet key) and `data-title` (the short name the rail index lists it under — the card art is a logo or a line-broken `h3`, so a readable title can't be derived from the DOM). Clicking calls `openSheet(key, srcEl)`. A JS snippet at page load moves any `.sub` inside `.art` out to be a direct child of `.pin`. By default `.sub` is in-flow (`max-height: 0` → expands on hover/focus), so revealing a card's caption pushes the card below it down within the same column — this is intentional, not a layout bug; `overflow-anchor: none` on `.main` (see the comment near the top of the stylesheet) exists specifically to stop Chrome's scroll anchoring from fighting this on hover near the bottom of the feed. The one exception: a pin sitting directly above a static, non-case-study component (`.pin[data-page]:has(+ .pin:not([data-page])) .sub`, e.g. before "Find me elsewhere") renders its `.sub` as an absolute overlay instead, so that specific component card never gets pushed. Do not flatten this back to a universally-absolute `.sub` — that silently kills the push interaction for every other card and has regressed before.

Because that push growth is real layout growth, it would also grow `.feed`'s own auto height and bounce the in-flow copyright footer beneath it — which the "card hover states cannot move it" rule (see Navigation and shared chrome below) forbids. `lockFeedHeight()` (defined next to `applyFilter` in the Filter + FLIP script section) pins `.feed` to a fixed measured height at rest, re-measuring only on real layout changes (`window load`/`resize`, after a filter's FLIP settles, on `setRoute('feed')`, and inside `flipPins` for sheet open/close) — never on hover. `.feed`'s `120px` bottom padding is the headroom that growth pushes into instead of the footer.

Filter tabs match `data-filter` against `data-cat`. Visibility changes use a FLIP animation (snapshot positions before/after, then transition with `translate`).

### Sheet panel

`openSheet(key, srcEl)` reads from the `PAGES` object and writes HTML into `#sheetBody`. If `PAGES[key].custom` exists, that string is used verbatim (full custom layout); otherwise the default template renders `eyebrow`, `title`, `lead`, `facts`, a figure placeholder, and `body`.

The app shell gains `.sheet-open` on open, triggering: **the rail turns into the case-study index** (see Navigation below), topbar hides, and the feed, tabs and copyright fade out behind the panel while keeping their exact layout. `--panel-w` becomes `calc(100vw - var(--chrome-size))` — the panel starts at the *compact* rail, and the rail's expanded index overlays its left edge rather than pushing it, so hovering the rail never reflows the case study. On small screens the rail is gone and the panel is `100vw`.

**The rail owns the seam.** While a sheet is open the panel's own `border-left` is transparent: it sits exactly under the compact rail's right border, so as the rail expanded its border travelled out with it while the panel's stayed behind, leaving two lines and making the divider look dropped and redrawn. One line moving with the rail through every state is what makes the expansion read as the same navigation getting wider rather than a different element appearing.

Two insets keep that overlay safe, and both must hold together: `.app.sheet-open .sheet` reserves `--sheet-inset-left` so the reading column never sits under the expanded rail, and `.sheet-head`'s padding aligns its controls to the reading column's own edges instead of the panel's. The study's meta sits on the reading column's **left** edge — level with the study's own eyebrow and hero — and the return control on its right edge, both on the same guides as the case study underneath rather than the panel's. That also puts the way out where the rail's expanded index can never cover it. (A `·` divider between them existed while the two were adjacent; it has no job now that they're at opposite ends.) The pill's hover/focus fill is the one thing it can't inherit from `.page-back`: `--control-soft` is a page-theme token, but a case study repaints `--paper` and `--ink` for its own palette, so that fill lands too close to the study's canvas — `.sheet-close` mixes the study's own ink into its own paper instead, which is always a visible step away from whatever it sits on. Header pills also carry an ink `:focus-visible` outline rather than the UA's blue, since the sheet focuses this one the moment a study opens.

That control is the **same `.page-back` "Back to Work" pill** About and Experience use (`#sheetClose` carries both classes), so every route out of a subview is one control. `.sheet-close` adds **nothing** visual — it carried a hairline for a while to separate it from a case study's repainted canvas, and that read as a different button, which is worse than the separation was worth. Keep them identical. The head's 16px vertical padding against that 48px pill is what keeps it exactly `--chrome-size` tall.

`--sheet-inset-left` is a `clamp()` written **in terms of the tokens**, never a pixel breakpoint: it reserves only the shortfall, so on a wide window where the centered column already clears the rail it falls to `0` and the column centers properly. It solves `chrome + inset + (panel − inset − toc − column) / 2 ≥ rail-open` for the inset, capped at the rail's own overhang. A hard-coded breakpoint there goes stale the moment the rail width, the column cap or the index width moves — which already happened once.

Nothing reflows on open, and that includes the banner: it fades with the rest of the board but must **not** collapse its height. It used to (`max-height: 0` plus its padding), which removed ~700px of layout above the feed the instant a study opened — everything below jumped up, and `.main` going `overflow: hidden` clamped the scroll against the newly shorter content. That was a visible flash of a scrolled-looking feed before the panel arrived — and the same jump in reverse on close, since one class drives both. Nothing behind the panel is visible anyway, so collapsing it buys nothing. Verified in both directions: `scrollTop` and `scrollHeight` hold constant through an open and through a close sampled every 100ms.

Nothing reflows on open, which is the point: there is no mini feed column, no FLIP of the pins, and no tabs-into-header animation any more (all three were removed with the mini column, along with `--open-column-gutter` and the sheet-open feed scrollbar accent). Closing is a straight cross-fade back to the board the visitor left. `.main` goes `overflow: hidden` while open so a wheel over the rail can't scroll the invisible feed; `closeSheet()` restores `main.scrollTop` from `feedScrollBeforeOpen`.

`.sheet-body` is capped at `1240px` and centered — a ~1030px measure. The figures are sized against this column, so widen it further only with a real look at them.

**Section index** (`.cs-toc`, `buildSheetToc()`): every case study gets a sticky right-hand table of contents, built from its own `.cs-section > .cs-label` text — a study gets one by being written normally, with nothing to maintain per page. Fewer than two labelled sections (the WIP placeholders) means no index at all. `buildSheetToc()` parks it level with that study's `.cs-hero` (via `--toc-top`) rather than at a fixed offset — heroes carry different eyebrows and toplines, so one hard-coded top lands differently in every study. It lives **inside** `.sheet` so it inherits each study's remapped palette (climate's yellow, brew's cream), floats into a reserved right gutter (`--toc-w`/`--toc-gap`/`--toc-edge`) with a negative margin so it costs `.sheet-body` no layout, and appears only at `1400px` and up — below that the study keeps the full width instead of being squeezed by an index. Section elements get ids and `scroll-margin-top`, so links are real anchors. Inactive links are `--ink-mid`, not `--ink-low`: on a study's own canvas the low ink lands around 2.5:1, under the floor for text.

The current section is read straight off scroll position in a rAF-throttled `scroll` handler on `.sheet`, **not** through an IntersectionObserver: exactly one link has to be current at every offset, including mid-section where an observer sits silent between threshold crossings. `.sheet` sets `scroll-behavior: smooth` so index and figure-reference anchors glide — because of that, content swaps must use `sheet.scrollTo({ top: 0, behavior: 'auto' })`, never `scrollTop = 0`, which would animate a long scroll during the cross-fade.

The `.sheet-head` rule runs the full width of the panel — from the rail's edge to the edge of the screen — by cancelling both insets with negative margins and paying them back as padding, so its controls stay on the column while its divider does not.

The dialog is named by `setSheetLabel()` — the **study**, then its meta ("brew., Product / UX · Design Eng"). It used to be `aria-labelledby="sheetMeta"`, which announced only the discipline, so opening any product study said "Product / UX · Design Eng" and never which one. Don't put `aria-labelledby` back: it would win over the label.

The sheet is `role="dialog"` but deliberately **not** `aria-modal` — the rail beside it is live navigation while a case study is open, so the rest of the page has to stay in the accessibility tree.

### Experience timeline

The experience section (`#expView`) is a scroll-driven horizontal track. A sticky sentinel + `IntersectionObserver` drives `updateExperienceTrackNow()` which reads scroll position and sets `--exp-track-x` on `.exp-scroll` to translate `.exp-track`. 

Experience cards carry no tech-logo stack. There was one (`.exp-tech-zone`, an absolute 76×210 column at the card's top-right); it overlaid the card's own content box and collided with the title and bullets as soon as the card narrowed, and it was removed rather than patched. The logo files under `assets/logos/` are now unreferenced.

End-of-scroll: `endHold = viewport.clientHeight * 0.55` — a buffer where the card deck fades out (smoothstep opacity + translateY) before the sticky viewport releases. Cards individually fade in via `--scroll-reveal-opacity` as they enter the track.

### Motion

**Splash glyph**: the outline's ease (`cubic-bezier(0.4, 0.1, 0.25, 1)`) is heavily front-loaded — the pen is 90% round by 600ms even though its transition does not end until 900ms — so the *perceived* landing is ~600ms, and overlap scheduled into that last 300ms is invisible. That is why the ink at 640ms still read as starting after the outline finished. It starts at 480ms now, and the wipe's curve was flattened at the head (`cubic-bezier(0.22, 0.35, 0.3, 1)`) because the old ease-in loitered near zero, so the sweep ran while nothing was visibly filling. **When retiming this, compare against the perceived landing, not `draw + drawDur`.**

`--motion-fast` / `--motion-med` / `--motion-slow` / `--rail-motion` are collapsed to `0.01ms` under `prefers-reduced-motion: reduce`. The JS storyboards opt out separately (`motionDelay()`, `playSheetRailEntry()`, the splash), but every CSS transition on the site runs off those four tokens, so that block is what actually honours the preference — put new durations on a token rather than hard-coding them, or they'll ignore it.

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
| `buildRailIndex()` | Builds the sheet-open rail index from the feed's case-study cards. |
| `syncRailIndex(key, indexId)` | Marks exactly one index row active (`null` key clears it). |
| `cardPalette(pin)` | A card's `{color, on, icon, dark}` — surface, text-on-surface, and its icon color per theme. |
| `playSheetRailEntry()` | Expands the rail on open, then collapses it to swatches. |
| `updateExperienceTrackNow()` | Recalculates horizontal scroll position and card reveal state. |

---

## Design tokens (CSS custom properties)

**Page chrome (theme-dependent — flips between `:root` and `[data-theme="dark"]`):** `--paper`, `--paper-2`, `--ink`, `--ink-mid`, `--ink-low`, `--line`, `--control-soft`, `--control-muted`, `--accent-soft`, `--sub-color`

**Feed card palette (theme-independent — defined once in `:root`, never overridden):** `--c-clay`, `--c-blue`, `--c-cream`, `--c-yellow`, `--c-sage`, `--c-plum`, `--c-sand`, `--c-charcoal`, `--c-noir`, `--c-noir-2`, `--accent`. `--c-plum` is a muted lavender (`oklch(0.71 0.06 305)`), lighter and softer than the old saturated plum but darkened back toward `--c-sage`/`--c-clay`'s lightness so it doesn't read as washed out next to them — don't push it lighter than ~0.72 or more saturated than ~0.07 chroma. `--c-clay` doubles as the "coral" tone (used by the Daily type & motion card and the About page's Design skill pill) — there is no separate `--c-coral` token, don't add one.

**Card content colors (also theme-independent):** `--card-ink`, `--card-muted`, `--card-paper` — used for text/labels drawn on top of the `--c-*` card backgrounds, regardless of page theme.

**Motion:** `--motion-fast` (180ms), `--motion-med` (420ms), `--motion-slow` (480ms), `--ease-out` (cubic-bezier 0.22,1,0.36,1), `--ease-soft` (cubic-bezier 0.19,1,0.22,1)

**Fonts:** `--serif` (Instrument Serif), `--sans` / `--mono` (Inter)

**Layout:** `--chrome-size: 80px` (rail width *and* topbar height — kept as one token on purpose, don't split it), `--rail-open-w: 240px` (the rail's *expanded* index width while a sheet is open, `212px` under 1100px — 240 is the narrowest that keeps every study label on one line, "Daily type & motion" beside its WIP tag being the tightest), `--toc-w` / `--toc-gap` / `--toc-edge` (the case-study section index and the gutter it reserves), `--panel-w` (sheet panel width: `68vw` at rest, `calc(100vw - var(--chrome-size))` under `.app.sheet-open`, `100vw` at 800px and below — the mobile override has to be re-stated on `.app.sheet-open` because it outranks `:root`)

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
- **Rail reveal**: outside the one-time entry storyboard below and the sheet-open index state, the rail stays compact and expands only when the navigation is hovered or keyboard-focused (`:focus-within`).
- **Rail index (`.rail-index`, sheet-open only)**: while a case study is open the rail becomes the index of case studies, which is what lets the feed column disappear and the sheet take the rest of the screen. `buildRailIndex()` builds it from the feed's own `.pin[data-page]` elements at load, so adding a card to the feed adds it here for free.
  - **Grouping**: sections are `RAIL_INDEX_CATS` = `ux, swe, viz, misc`, and a study is listed under **every** discipline in its `data-cat` — brew and Habitat appear under both Product / UX and SWE on purpose, because those entries will eventually open a product view and a software view of the same project. Anything whose `data-cat` falls outside those four (the studio-only WIIRED card) falls back to Misc, so nothing can drop out of the index. **Studio is deliberately not a section**: it's a client relationship, not a discipline, and the studio work already appears under the craft it represents. The feed's filter tabs still have their own WIIRED Studio tab — that's a different surface, don't sync them.
  - **Order** comes from `RAIL_INDEX_ORDER`, not from the feed's hand-balanced column order, so a study listed in two sections sits in the same relative place in both — brew then Habitat under Product / UX exactly as under SWE. A study missing from the list keeps feed order, after the listed ones.
  - **Choosing a study always starts it at the beginning.** All three `openSheet` paths reset the panel's scroll — a fresh open, a cross-fade to another study, and re-selecting the study that's already open (which loads nothing, so it glides rather than jumps).
  - **One active entry**: `syncRailIndex(key, indexId)` activates a single row, keyed by `data-index-id` (`cat:page`), not by project. Opening from the index activates the row that was clicked; opening from a feed card or an About deep link has no discipline context, so the project's first listed row stands in. Two rows lighting up for one study reads as a bug — don't reintroduce it.
  - **The compact stack shows each project once.** Collapsed, the rail has no category labels to explain why a project would appear twice, so the labels, the group spacing and every repeat row collapse to zero height and it reads as one evenly spaced column of unique projects; expanding restores the sections and the rows slide down into them. `syncRailIndex` picks the representative: the **open** entry always represents its own project (so the active fill can never land on a collapsed row) and every other project falls back to its first entry. That is why the index spaces itself with margins rather than a flex `gap` — a gap would keep reserving space for the collapsed rows. A hidden row is still focusable, which is correct here: `:focus-within` expands the rail, so tabbing to one reveals it.
  - **Rows**: `.rail-index-item` is literally a `.rail-btn`'s box — same 48px height, radius, 13px left padding, 10px icon gap, 10px rhythm between rows, same 16px container insets. Compact, that makes a row exactly the circle a route button is, starting where Work starts and on the same pitch, so the two rail states line up row for row. Nothing about the row's own box changes between states — the rail's width is what turns the pill into a circle — so no icon shifts sideways as it opens. The active row fills with the card's **own** surface color and uses that card's text color (`cardPalette()`, mirroring the `.art-*` rules) — `--card-color` / `--card-on` per row. **The near-black cards invert**: charcoal and noir read as a hole punched in the rail at chip scale rather than a sibling of the cream and yellow pills, so those fill with the light tone the card draws its own text in and carry the card's color as the text plus a hairline `--card-ring` — without that ring an off-white chip vanishes into the light rail. Both colors still come from the same card; they swap roles. Section labels use `--ink-mid`, not `--ink-low`: at 10px uppercase, low ink is unreadable on the dark rail.
  - **Written studies only**: `buildRailIndex()` skips cards still marked `Under construction`. An index is navigation and a WIP card has nothing to open; it keeps its WIP badge in the feed, where that means something, and appears here on its own once it's written.
  - **Icons** (`RAIL_INDEX_ICONS`, keyed by sheet slug): one per study, **solid** on a 24×24 box — the route buttons above stay stroked, the studies read as a set of their own. They say what the project *is* (a cloud for a dream app, a target for habit goals), because the compact rail shows nothing else. A study with no entry falls back to `default` rather than rendering an empty slot.
    - **brew** is the exception: it wears its own card mark from `assets/images/brew-mug-icon.svg`, **inlined** with the icon's own `viewBox` rather than referenced. It was a CSS `mask` first, and that left an empty slot whenever the file didn't load — don't go back to an external reference. The viewBox is the mark's own, recentred on its measured ink centroid (42.4% / 48.6% of the artwork, not 50/50 — the cup carries the weight, the handle doesn't) and widened so the mark fills ~86% of the box, since the drawn icons carry ~2px of padding inside theirs and this one runs edge to edge. Re-measure both numbers if the mark is ever redrawn.
    - **Icon color is the rail's ink** — black on the light rail, white on the dark one. The icons read as chrome rather than seven accents competing down a column; card color is still what identifies a study, it's just held for the **active pill's fill**, where it means "this is the one you're reading" (and the icon flips to that card's own text color there). `CARD_PALETTE` therefore only carries `color` and `on`. An earlier version tinted every icon with its card's color and needed per-theme corrections for the pale and dark cards — if that comes back, it brings that whole problem with it.
  - **The routes step aside**: `.rail-nav` (Work / Experience / About) **tucks left** out through the rail's own `overflow: hidden` while a sheet is open. The logo and the sheet's close control are the way back out. It goes `position: absolute`, pinned at `--rail-nav-top` (where it rests, below the logo), the moment a study opens — that is what makes this read as a tuck rather than a sag. Left in the flex flow it was shoved *downward* by the index growing above it, roughly 290px of push against 80px of sideways travel, so the icons looked like they were sliding out of the bottom of the rail; out of flow they cannot be pushed at all and only ever move left. Going absolute also hands the index its space immediately, which the index still eases into on its own `flex-grow`. Keep `--rail-nav-top` in step with the logo block; it is only load-bearing for the ~360ms of the exit, so a small drift is a nudge in the animation rather than a broken layout. The transform and opacity run slightly *shorter* than `--rail-motion` on purpose — the one place a shorter duration is correct, because it buys direction.
  - **Mechanics**: the index sits between the logo and `.rail-nav` in the DOM (so the sheet-open reading and tab order match the visual one) and is zero-height at rest via `flex: 0 1 0` — the closed rail is exactly what it always was. Opening animates `flex-grow` 0 → 1. It's also `visibility: hidden` at rest so it stays out of the tab order and the accessibility tree.
- **Closing is one gesture.** The panel slides out on `--motion-slow` / `--ease-soft`; `.app.sheet-closing` retimes the rail's whole choreography to those same values (`--rail-motion` / `--rail-ease`), so the collapse and the slide are one move rather than two events at different speeds and curves. Two things go with it: `closeSheet()` removes `.is-sheet-expanded` **inside** the same rAF that adds `.sheet-closing` (outside it, the rail started collapsing a frame early on its own clock), and the index's fade and `visibility` flip run the collapse's full duration — on a shorter one the index blanked while its box was still shrinking, which was most of the chop.
- **Labels fade as a group; they don't wipe.** `.rail-index-title` animates opacity only, at one duration and one delay for every row — the row's own overflow (and the rail's) does the clipping. A `max-width` wipe, which is what `.rail-label` still uses, finishes when a label runs out of letters, so short names landed a third of the way through while long ones were still unrolling; that is what read as rows popping up one at a time. For the same reason a repeat row unfolding on hover clears the per-item entry stagger (`transition-delay: 0s` on `.is-compact-hidden`), which otherwise made the second copy of a study arrive after everything else.
- **One duration for the open/close**: everything the rail moves when it expands or collapses — its width, row heights and padding, label heights, group spacing, the routes stepping aside — runs on `--rail-motion` with `--ease-out`. Mismatched timings (a 320ms width against 420ms heights) are what made the expand read as several animations racing each other; if you add something that moves in that beat, put it on the same token. Text is the exception, and deliberately so: labels fade **out** immediately on collapse and fade **in** 90ms behind the geometry on expand, so nothing is caught mid-clip.
- **Sheet rail storyboard** (`playSheetRailEntry()`): on a device that can't hover (`hover: none`, 801px and up — tablets, touch laptops) the rail simply **stays expanded** while a study is open. There is no way to reopen it there: hover doesn't exist and tapping a row opens that study instead, so a collapsed index would strand the visitor with unlabelled icons. It reuses `.is-sheet-expanded`, and the panel already reserves the expanded width, so nothing else changes. Everywhere else, opening a study expands the rail for 2600ms so the rest of the work is visible without hunting, then collapses it to a column of card-colored swatches and hands the width back to the case study. Hover and keyboard focus reopen it from there — CSS `:hover` holds it open on its own, so the timer never has to know where the pointer is. Reduced motion skips the auto-expand and leaves the swatch column, which hover still opens. `closeSheet()` calls `endSheetRailEntry()`.
- **Leaving from an open sheet**: the rail is clickable while a sheet is open now, so the logo calls `closeSheet()` before `setRoute()`. Anything else added to the rail must do the same.
- **Work rail entry storyboard**: `playWorkRailEntry()` (defined next to `updateFeedScrollbar`, called from the hash-routing block only when `initialRoute === 'feed'`) briefly auto-expands the rail (`.is-intro-expanded`, same CSS as hover/focus-within) 180ms after landing on Work, then auto-collapses it at 2380ms — a one-time "there's navigation here" reveal, not a persistent state. It fires on true entry into Work (first load or a refresh while on Work), desktop widths only (`min-width: 801px`), and respects `prefers-reduced-motion`. It must never fire from in-app navigation back to Work (rail click, logo, Back to Work) — only `setRoute()` handles those, and `setRoute()` never calls it. This exact feature has been silently deleted before while "refining navigation" (the CSS class survived, the JS didn't) — don't let that happen again.
- **Expanded width**: the rail expands only to `168px`, which is the smallest width that gives the Experience pill adequate right padding. Do not restore the older `192px` width without a demonstrated need.
- **Rail labels**: the primary route is named **Work** in the rail even though its internal route key is `feed`. The other labels are Experience and About Me.
- **Updated tag**: the topbar's social row ends with a hairline divider and an `Updated <Mon YY>` tag. The month in the markup is the fallback that ships in the file — it renders offline and on `file://` — and `refreshUpdatedTag()` replaces it with the date of the newest **public push** from GitHub's events feed for `GITHUB_USER`. No key and no backend: the endpoint is CORS-open and a portfolio's traffic stays far under the unauthenticated limit, and the answer is cached in `localStorage` for 12 hours so clicking around costs one request. That feed only reaches back ~90 days, so a quiet stretch leaves the shipped month standing rather than showing a gap; every failure path is silent. Bump the fallback month in the markup when it drifts.
- **Header**: the topbar no longer carries a contact/utility nav (the Experience/Email/LinkedIn text links were removed); it exists only for the scroll-shadow chrome (`.topbar::before` / `.scrolled`) and to match the rail's `--chrome-size` height. Email, LinkedIn, GitHub, and Instagram are reachable via the social tiles in the feed banner and the Elsewhere card.
- **Return controls**: About and Experience each show a quiet outlined **Back to Work** button beneath the introductory title (both call `setRoute('feed')`), and an open case study shows the same pill in its sheet head (`closeSheet()`). One shape for every way back to Work — don't reintroduce a bare × for the sheet.
- **Footer**: the copyright is a centered, in-flow footer at the bottom of the Feed view, never fixed to the viewport. Current copy is `© 2026 Zainab Ahsan, All Rights Reserved`. Keep it low contrast and ensure card hover states cannot move it.
- **Mobile drops the WIP cards.** At 800px and below, `.pin[data-cursor-label="Under construction"]` is hidden, and the Misc filter tab with it (daily is its only entry). A WIP card can't be opened and the only thing that says so is the custom cursor's "Under construction" label, which is `pointer: fine` only — on a phone it looks like a card that works and does nothing when tapped. Touch tablets at 801px and up still show them; same problem, not yet solved there.
- **Mobile**: the desktop rail remains hidden at `800px` and below. Do not introduce rail-only information that becomes unavailable on mobile. The rail index is the one exception, and only because the surface it replaces (the mini feed column) was already pushed off-screen at that width by the full-screen panel: on mobile the sheet is `100vw` and closing it returns to the full feed.

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
