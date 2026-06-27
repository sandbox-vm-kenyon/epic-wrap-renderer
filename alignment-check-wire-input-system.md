# Alignment Check — Wire the Input System

**Date:** 2026-06-27  
**Epic:** Epic-wrap + long-description live test  
**Task:** Wire the input system  
**Reviewer branch:** unharness-node-b474cf4b

---

## Scope

This check assesses whether the input system in `renderer.html` is correctly wired — i.e., whether all interactive controls (text fields, sliders, buttons) are connected to the live preview and update it accurately.

---

## Input System Inventory

The renderer exposes four categories of user input:

| Input element | ID | Purpose |
|---|---|---|
| Text input | `liveTitle` | Epic card title (live preview) |
| Textarea | `liveDesc` | Long description (live preview) |
| Range slider | `widthSlider` | Card viewport width (200–860 px) |
| Range slider | `clampSlider` | Description clamp depth (1–12 lines) |
| Expand button | `expandBtn` | Toggle truncated/full on live preview |
| Per-card expand buttons | (inline) | Toggle truncated/full on each preset card |

---

## Wiring Assessment

### 1. Title input → live title display
**Status: ✅ Correctly wired**

`liveTitle.addEventListener('input', updateLive)` (line 292) triggers `updateLive()`, which writes `liveTitle.value` into `liveTitleEl.textContent` (line 271). Initial value rendered on load via `updateLive()` call at line 294.

### 2. Description textarea → live description display
**Status: ✅ Correctly wired**

`liveDesc.addEventListener('input', updateLive)` (line 293) triggers `updateLive()`, which writes `liveDesc.value` into `liveDescEl.textContent` (line 272). Metrics (char count, word count, estimated lines) also update on every keystroke (lines 273–278).

### 3. Width slider → card viewport width
**Status: ✅ Functionally wired (minor style inconsistency)**

Wired via inline `oninput` attribute on the slider element (line 219–220), directly setting `cardViewport.style.width` and updating the `#widthVal` label. Works correctly.

Minor issue: uses inline `oninput` rather than `addEventListener`, inconsistent with the title/desc pattern. Functional impact: none.

### 4. Clamp slider → CSS custom property `--clamp`
**Status: ✅ Correctly wired**

Wired via inline `oninput` calling `updateClamp(this.value)` (line 223–224). `updateClamp()` sets the CSS custom property on `liveDescEl` via `style.setProperty('--clamp', v)` (line 289) and updates the `#clampVal` label. The `.truncated` class on `liveDescEl` consumes `--clamp` in CSS (line 147).

### 5. Expand/collapse (live preview)
**Status: ✅ Correctly wired**

`toggleExpand()` (lines 281–285) toggles the `.truncated` class on `liveDescEl` and flips button text. State tracked via `expanded` boolean.

### 6. Expand/collapse (preset cards)
**Status: ✅ Correctly wired**

`toggleCard(btn)` (lines 360–364) uses `classList.toggle('truncated')` on the sibling description block. Return value interpretation is correct: `true` (class added = truncated, show "Show more") and `false` (class removed = expanded, show "Show less").

### 7. Initial render on load
**Status: ✅ Present**

`updateLive()` is called unconditionally at line 294, so the live preview is populated from the hardcoded default values immediately on page load without requiring user interaction.

---

## Issues Found

### Issue 1 — Inline `oninput` vs. `addEventListener` inconsistency (style, low severity)
Width and clamp sliders use inline `oninput` attributes while title/desc use `addEventListener`. No functional difference in this static-HTML context, but inconsistent readability.

**Recommendation:** Migrate slider wiring into the script block alongside the other event listeners for consistency.

### Issue 2 — Width slider bypasses `updateLive()` (low severity)
The width slider updates card width directly without calling `updateLive()`. This is correct behavior (width doesn't affect metric values), but means any future metric that depends on rendered width (e.g., actual line count at current width) would need explicit wiring.

**Recommendation:** Document this separation if metrics are extended.

### Issue 3 — No `aria-live` region on live preview (accessibility gap)
The live preview updates on every keystroke but has no `aria-live` announcement. Screen reader users will not perceive changes. Not a wiring correctness issue, but an accessibility gap in the input-to-output pipeline.

**Recommendation:** Add `aria-live="polite"` to `#cardViewport` or a summary element, or add a visually-hidden status region that announces key metric updates.

---

## Verdict

**The input system is correctly wired.**

All six input surfaces (title field, description textarea, width slider, clamp slider, live expand button, per-card expand buttons) are connected to their corresponding output targets and update them correctly. The initial render on load is present. No broken wiring paths were found.

The issues noted are a style inconsistency (inline vs. addEventListener), a forward-looking documentation note, and an accessibility gap — none of these constitute broken wiring for the epic's stated goals.

**This task is aligned. No blocking defects.**

---

## Live Link

[https://sandbox-vm-kenyon.github.io/epic-wrap-renderer/](https://sandbox-vm-kenyon.github.io/epic-wrap-renderer/)
