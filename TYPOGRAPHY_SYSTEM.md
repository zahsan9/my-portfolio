# Typography System

## Three-Face Hierarchy

### 1. Instrument Serif (Primary Editorial)
**Role:** Heroes, main case study titles, authoritative moments  
**Character:** Refined, elegant, confident  
**Usage:**
- Hero titles (`.cs-hero-title`)
- Page headlines
- Primary case study headings

**Weights:** Regular, Italic

---

### 2. Fraunces (Expressive Editorial) ✨ NEW
**Role:** Questions, reflections, warm editorial moments  
**Character:** Soft, wonky, characterful - brings warmth and personality  
**Usage:**
- Case study questions (`.cs-question`) - opsz 72, weight 300
- About page principles (`.ab-principle`) - opsz 16, weight 300, italic
- Editorial section headings

**Weights:** Light 300 (primary), Semibold 600 (accent)  
**Variable axes:** Optical size (opsz) 9-144  
**Settings:** Lower optical sizes (9-72) for softer, editorial quality

**Why Fraunces:**
- Complements Instrument's refinement with warmth
- Variable font with optical sizing for typographic nuance
- Distinctive without being trendy (not in top-10 portfolio fonts)
- Soft serifs fit visual/Pinterest aesthetic (not blocky like Space Mono)
- Brings personality to reflective, question-driven moments

---

### 3. Inter (Structure & UI)
**Role:** Body text, navigation, metadata, UI elements  
**Character:** Clean, geometric, highly legible  
**Usage:**
- Body copy (`.cs-copy`, `.ab-bio`)
- Navigation labels
- Case study metadata
- Section labels (`.cs-label`) - uppercase, tracked
- Facts, stats, captions
- Buttons, controls

**Weights:** 400 (regular), 500 (medium), 600 (semibold), 700 (bold)

---

## Typographic Voice

**Instrument Serif** says: "Authoritative, polished, confident"  
**Fraunces** says: "Thoughtful, warm, human"  
**Inter** says: "Clear, structured, modern"

Together they create a **layered editorial system** rather than a flat utility stack.

---

## Application Rules

1. **Reserve Instrument for heroes only** - don't overuse it in every heading
2. **Use Fraunces for questions and reflections** - moments that invite pause
3. **Use Inter for everything structural** - it's your workhorse
4. **Optical sizing matters** - Fraunces at opsz 72 for large text, 16 for small
5. **Weight contrast** - Fraunces Light (300) makes it feel editorial, not heavy

---

## What Changed (2026-09-10)

**Before:**
- Instrument Serif (editorial)
- Inter (everything else)
- `--mono` token pointed to Inter (wasted opportunity)

**After:**
- Instrument Serif (heroes)
- **Fraunces (expressive moments)** ← NEW
- Inter (structure)
- `--mono` stays Inter for now (metadata, small caps work fine with Inter)

**Why not Space Mono:** Too blocky/technical for a visual portfolio with Pinterest feel. Fraunces brings the unexpected character without sacrificing refinement.

---

## Future Considerations

- If case study data tables need true monospace, consider **IBM Plex Mono** or **Recursive**
- Could use Fraunces Semibold (600) for accent pull-quotes or impact callouts
- Explore Fraunces' goofy/wonk axis if more personality is needed

