# Alignment Check — Touchscreen Joystick (that also wo…)

**Date:** 2026-06-27  
**Epic fragment:** "Touchscreen joystick that also wo[rks]"  
**Reviewer task:** unharness-node-110888bd  
**Base checked against:** `master` as of commit 73ef70c

---

## Preliminary Note: Truncated Epic Name

The epic name delivered to this task is **incomplete** — it ends at "wo", most likely truncated mid-word.  
Probable full-name candidates:

| Candidate | Interpretation |
|---|---|
| "Touchscreen joystick that also **works** [with mouse/keyboard]" | Dual-input control: touch drag + mouse drag for the width/clamp sliders |
| "Touchscreen joystick that also **works as a scroll control**" | Joystick metaphor for scrolling the card viewport on touch devices |
| "Touchscreen joystick that also **works offline**" | PWA/offline extension of the renderer (unlikely given the "joystick" noun) |

The most plausible reading — and the one this check uses — is a **touch-friendly joystick/drag UI** for the renderer's interactive sliders (card width and clamp depth) that is also operable via mouse and keyboard, so the renderer is fully usable on mobile and tablet without relying on `<input type="range">`.

**If the actual full epic name differs, this alignment check should be re-run with the corrected name.**

---

## What the Epic (as interpreted) Requires

| Concern | Meaning |
|---|---|
| **Touchscreen input** | Sliders or controls must respond to touch events (`touchstart`, `touchmove`, `touchend`) on mobile/tablet |
| **Joystick metaphor** | A 2-D drag handle (or at minimum a 1-D draggable thumb) replacing or supplementing native `<input type="range">` |
| **"also works"** | The control must be a progressive enhancement — mouse and keyboard must continue to function; no regression for desktop users |

---

## Current Implementation Audit

### Controls that would be replaced or enhanced

| Control | Current implementation | Touch-friendly? |
|---|---|---|
| Card width slider | `<input type="range" id="widthSlider">` (line 219) | Partial — native range inputs are touch-operable in most browsers but provide no visual joystick affordance |
| Clamp-depth slider | `<input type="range" id="clampSlider">` (line 223) | Same as above |
| Card viewport resize | CSS `resize: horizontal` on `.card-viewport` (line 83) | ❌ Not touch-accessible — browser resize handles are mouse-only |

### Touch-related CSS/JS — current state

Grep of `renderer.html` for touch-handling:
- No `touch-action` CSS properties found
- No `addEventListener('touchstart' …)` found
- No pointer-events unification (`pointerdown`/`pointermove`) found
- No joystick or drag-handle markup found

**Verdict: the current renderer has zero touchscreen-specific implementation.**

---

## Gap Analysis

### Gap 1 — No touch/pointer event handlers (blocking)
The renderer relies entirely on native `<input type="range">` for its interactive controls. While these are technically touch-operable in modern iOS/Android browsers, they are small targets and provide no joystick-style affordance. The `resize: horizontal` viewport handle is entirely mouse-only.

**What is needed:** Either (a) replace sliders with custom drag handles using `PointerEvent` API (covers mouse, touch, and stylus in one code path), or (b) add explicit `touchstart`/`touchmove` listeners with `touch-action: none` to prevent scroll interference.

### Gap 2 — Viewport resize handle not touch-accessible (blocking)
`.card-viewport` uses `resize: horizontal`, a CSS property with no touch equivalent. On mobile, the user cannot resize the card to test wrap behavior at different widths — which is the primary purpose of the renderer.

**What is needed:** A drag handle element with pointer/touch events that sets `style.width` programmatically, replacing or supplementing `resize: horizontal`.

### Gap 3 — No `viewport meta` touch tuning (minor)
The `<meta name="viewport">` tag (line 5) uses defaults. Adding `touch-action` and ensuring no 300ms tap delay (or using `touch-action: manipulation` on interactive elements) would improve responsiveness.

### Gap 4 — No acceptance criteria for "joystick" fidelity (blocking for sign-off)
The epic name implies a specific interaction metaphor ("joystick") but provides no definition of what constitutes pass/fail. Required at minimum:
- Minimum touch target size (WCAG recommends 44×44 px)
- Expected behavior when dragging outside the control bounds
- Whether keyboard (arrow keys) must also adjust the joystick control
- Whether the joystick is 1-D (single axis, width only) or 2-D (width + clamp simultaneously)

### Gap 5 — Truncated epic name (process gap)
The deliverable epic name is cut at "wo". No implementation can be fully aligned against an incomplete specification. This is a data-quality issue in the task pipeline.

---

## Feature Coverage Matrix

| Epic concern | Implemented? | Notes |
|---|---|---|
| Touch-operable width control | ⚠️ Partial | Native range input works on touch but no joystick UI |
| Touch-operable clamp control | ⚠️ Partial | Same as above |
| Touch-operable viewport resize | ❌ No | CSS `resize` is mouse-only |
| Joystick drag-handle UI | ❌ No | Not present |
| Pointer/touch event unification | ❌ No | No `PointerEvent` or `touchmove` listeners |
| Keyboard accessibility of joystick | ❌ No | No custom keyboard handling beyond native slider |
| No regression for desktop/mouse users | ✅ Pre-condition met | Current sliders work with mouse |

---

## Overall Verdict

**Not aligned. Significant implementation gaps exist.**

The renderer's current interactive controls do not implement the joystick/touch interaction model implied by the epic name. Two of the three interactive surfaces (custom viewport resize) are completely inaccessible on touch devices. The "joystick" affordance and any associated drag-handle UI are entirely absent.

This epic requires meaningful new frontend work before alignment can be declared. The truncated name is also a process-level gap that should be resolved before implementation begins.

---

## Recommendations (Priority Order)

1. **Obtain the full epic name** — resolve the truncation and re-read scope.
2. **Define acceptance criteria** — minimum touch target size, joystick axis count, keyboard fallback requirement.
3. **Replace `resize: horizontal` with a JS drag handle** — use `PointerEvent` for mouse+touch in one handler.
4. **Optionally replace native range inputs** with styled drag controls — or at minimum wrap them in larger touch targets.
5. **Add `touch-action: none`** to draggable regions to prevent scroll hijacking during drag.
6. **Add a mobile viewport test case** to the preset grid (narrow, e.g., 320–375 px) to cover phone-width wrap behavior.

---

## Live Link (current renderer, pre-joystick)

[https://sandbox-vm-kenyon.github.io/epic-wrap-renderer/](https://sandbox-vm-kenyon.github.io/epic-wrap-renderer/)
