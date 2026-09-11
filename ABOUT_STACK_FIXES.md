# About Card Stack - Edge Case Fixes

## Issues Fixed (2026-09-10)

### 1. ✅ Resize During Animation
**Problem:** Window resize while cards are animating would skip remeasurement because `aboutStackBusy` blocked refresh  
**Fix:** 
- Added debounced resize handler (150ms)
- Force-finishes animation if in progress
- Remeasures stack with new dimensions
- Prevents stuck cards or misaligned fan on window resize

```javascript
let resizeTimer;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => {
    if (aboutStackBusy) {
      finishAboutMotion();
      void aboutStack.offsetWidth;
    }
    refreshAboutStack();
  }, 150);
});
```

---

### 2. ✅ Touch Device Support
**Problem:** 
- No explicit touch handling
- Hover states triggering on touch devices
- Possible double-tap issues

**Fix:**
- Added proper touchstart/touchmove/touchend handlers
- Prevents activation if user was dragging
- Only applies hover listeners on `(hover: hover)` devices
- Touch gestures now work smoothly on mobile/tablets

```javascript
card.addEventListener('touchstart', () => {
  card._touchMoved = false;
}, { passive: true });

card.addEventListener('touchmove', () => {
  card._touchMoved = true;
}, { passive: true });

// Only activate if not dragging
if (e.type === 'touchend' && card._touchMoved) return;
```

---

### 3. ✅ Rapid Click Protection
**Problem:** User rapidly clicking/tapping multiple cards could pile up animations and cause janky behavior  
**Fix:**
- Added 100ms throttle between card selections
- Prevents animation queue pile-up
- Blocks spam clicks/taps without feeling sluggish

```javascript
const MIN_SELECT_INTERVAL = 100;
let lastSelectTime = 0;

const now = Date.now();
if (now - lastSelectTime < MIN_SELECT_INTERVAL) return;
lastSelectTime = now;
```

---

### 4. ✅ Viewport Overflow on Small Screens
**Problem:** On narrow viewports (<600px), fanned cards could escape their container  
**Fix:**
- Responsive `--stack-tilt` and `--stack-peek` values
- Reduced fan spread at 600px: 4deg → 2.5deg
- Further reduced at 400px: 2.5deg → 2deg
- Adjusted margin compensation for smaller peek values

```css
@media (max-width: 600px) {
  .ab-stack {
    --stack-tilt: 2.5deg;
    --stack-peek: clamp(60px, 8vw, 94px);
    margin-right: calc(var(--stack-peek) * 1.5);
  }
}
@media (max-width: 400px) {
  .ab-stack {
    --stack-tilt: 2deg;
    --stack-peek: 50px;
    margin-right: var(--stack-peek);
  }
}
```

---

## Existing Safeguards (Already Working)

### ✅ Reduced Motion Support
- Detects `prefers-reduced-motion: reduce`
- Skips all animations, instant state changes
- If motion preference changes mid-animation, finishes immediately

### ✅ Mobile Flattening
- At 800px and below, deck becomes vertical stack
- Removes `role="button"` and `tabindex`
- Disables all fan/hover mechanics
- Pure read mode on small screens

### ✅ Focus/Hover Race Condition
- Already handled via `is-hovered` JS-owned state
- Only one card can be hovered at a time
- Keyboard focus gets separate visual indicator
- Prevents two cards lifting simultaneously

### ✅ Front Card Protection
- Front card cannot be re-selected (no-op)
- Cursor changes to `default` on front card
- Only back cards are interactive

---

## Testing Checklist

- [ ] Desktop: Click through all cards in sequence
- [ ] Desktop: Rapid-fire click multiple cards - should throttle gracefully
- [ ] Desktop: Hover cards without clicking - should lift and return
- [ ] Desktop: Resize window while card is animating - should reset cleanly
- [ ] Mobile/Tablet: Tap cards - should select without hover effects
- [ ] Mobile/Tablet: Drag/swipe - should not accidentally trigger selection
- [ ] Narrow viewport (<600px): Cards should stay within bounds
- [ ] Very narrow (<400px): Fan should be minimal but still functional
- [ ] Keyboard: Tab through cards, Enter/Space to select
- [ ] Reduced motion: All transitions instant, no animation delays

---

## Known Limitations

1. **Intro animation on very fast resize:** If user aggressively resizes during the initial intro spread, cards will finish then snap to new metrics. Not harmful, just not perfectly smooth.

2. **Hover on touch + mouse hybrid devices:** Some Windows laptops with touchscreens will show hover states. This is expected - `(hover: hover)` detects capability, not current input method.

3. **800px breakpoint is hard:** Mobile flattening happens at exactly 800px. Future: could make this more granular with intermediate states.

---

## Future Enhancements (Optional)

- [ ] Add `pointer-events: none` during animation for extra interaction safety
- [ ] Detect orientation change on mobile and remeasure
- [ ] Add haptic feedback on card selection (mobile)
- [ ] ARIA live region announcing which card is now front
- [ ] Swipe gesture to cycle through cards on mobile

