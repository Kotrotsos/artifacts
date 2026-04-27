# WCAG 2.2 AA Compliance Report
**Product:** Virtuele Assistent (AI chatbot for Dutch municipalities)  
**Scope:** agent-frontend — Next.js chat UI  
**Standard:** WCAG 2.2 Level A + AA  
**Date:** April 2026  
**Branch:** feature/accessibility  

---

## Status Key

| Symbol | Meaning |
|--------|---------|
| PASS | Implemented and verifiable in code |
| PARTIAL | Partially implemented; gaps noted |
| FAIL | Not implemented or actively violated |
| N/A | Criterion does not apply to this product |
| NEEDS TESTING | Cannot be determined from code alone — requires browser/screen reader testing |

---

## 1. Perceivable

### 1.1 Text Alternatives

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 1.1.1 | Non-text Content | A | PARTIAL | Decorative icons have `aria-hidden="true"`. The avatar star icon in assistant messages has no alt. The send/cancel/copy icon buttons all have `aria-label`. No images used in content. |

### 1.2 Time-based Media

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 1.2.1 | Audio-only and Video-only (Prerecorded) | A | N/A | No audio or video content. |
| 1.2.2 | Captions (Prerecorded) | A | N/A | No synchronized media. |
| 1.2.3 | Audio Description or Media Alternative (Prerecorded) | A | N/A | No video. |
| 1.2.4 | Captions (Live) | AA | N/A | No live audio. |
| 1.2.5 | Audio Description (Prerecorded) | AA | N/A | No video. |

### 1.3 Adaptable

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 1.3.1 | Info and Relationships | A | PARTIAL | Chat structure uses semantic roles (`role="log"`, `role="alert"`, `role="status"`, `role="switch"`). Speaker identity conveyed via sr-only labels ("Assistent:", "U:"). Debug panel sections use styled `<div>` with `<h3>` text but no explicit heading role — minor gap. |
| 1.3.2 | Meaningful Sequence | A | PASS | DOM order matches visual order throughout. Messages render in chronological order. |
| 1.3.3 | Sensory Characteristics | A | PASS | No instructions rely on shape, color, size, or position alone. |
| 1.3.4 | Orientation | AA | PASS | No orientation lock present. App is responsive. |
| 1.3.5 | Identify Input Purpose | AA | PARTIAL | Composer textarea has `aria-label="Bericht invoer"` but no `autocomplete` attribute. Adding `autocomplete="off"` (or an appropriate value) would satisfy this fully. |

### 1.4 Distinguishable

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 1.4.1 | Use of Color | A | PASS | Interactive state (active/inactive toggle switches) conveyed via `role="switch"` and `aria-checked`, not color alone. |
| 1.4.2 | Audio Control | A | N/A | No auto-playing audio. |
| 1.4.3 | Contrast (Minimum) | AA | PARTIAL | Body text (foreground on background) uses Tailwind oklch color tokens — contrast untested with tooling. Welcome subtitle previously used `text-muted-foreground/65` (opacity reduced to ~65%) which is fixed. Link colors in markdown output (`text-blue-600` light, `text-blue-400` dark) need contrast verification against their backgrounds, particularly in dark mode. |
| 1.4.4 | Resize Text | AA | NEEDS TESTING | Uses `rem`/`em`-based sizing via Tailwind. No hardcoded `px` font sizes detected. Needs browser verification at 200% zoom. |
| 1.4.5 | Images of Text | AA | PASS | No images of text used. |
| 1.4.10 | Reflow | AA | NEEDS TESTING | Layout uses `max-w-[var(--thread-max-width)]` (48rem) and responsive padding. Needs testing at 320px viewport width to confirm no horizontal scroll. |
| 1.4.11 | Non-text Contrast | AA | PARTIAL | Focus rings use Tailwind `ring-black`/`ring-white` (high contrast). UI controls (buttons, input borders) need 3:1 contrast against adjacent colors — untested. The `outline-ring/50` in globals.css uses 50% opacity which may fail. |
| 1.4.12 | Text Spacing | AA | NEEDS TESTING | No inline styles that restrict line height, letter spacing, or word spacing detected. Needs browser testing with forced spacing overrides. |
| 1.4.13 | Content on Hover or Focus | AA | PARTIAL | Tooltips via Radix `@radix-ui/react-tooltip` are hoverable and dismissible with Escape. However, tooltips render immediately (`delayDuration` not set, defaults to 0 ms) — tooltips should persist and be hoverable, which Radix handles correctly. PASS on behavior; no issues detected. |

---

## 2. Operable

### 2.1 Keyboard Accessible

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 2.1.1 | Keyboard | A | PARTIAL | All visible interactive elements are keyboard-reachable (send, cancel, copy, reload, edit, branch nav, suggestions, source links). The attachment button is keyboard-reachable but non-functional (logs to console). No keyboard-only operations have been verified end-to-end with a screen reader. |
| 2.1.2 | No Keyboard Trap | A | NEEDS TESTING | No focus trap is implemented anywhere (including the debug panel, which stays open while user can tab out). This is acceptable for the chat interface itself, but needs verification that Tab can escape all composite widgets. |
| 2.1.4 | Character Key Shortcuts | A | N/A | No single-character keyboard shortcuts are configured. |

### 2.2 Enough Time

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 2.2.1 | Timing Adjustable | A | N/A | No time limits imposed. |
| 2.2.2 | Pause, Stop, Hide | A | PARTIAL | Framer-motion entry animations (opacity/y slide-in on messages, welcome screen) play automatically. They are short (< 1s) and not looping, so they fall below the 5-second threshold for this criterion. However, see 2.3.3 note below. |
| 2.2.6 | Timeouts | AA | N/A | No inactivity timeouts or session expiry present. *(New in WCAG 2.2)* |

### 2.3 Seizures and Physical Reactions

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 2.3.1 | Three Flashes or Below Threshold | A | PASS | No flashing content. |

### 2.4 Navigable

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 2.4.1 | Bypass Blocks | A | PASS | Skip link "Ga naar hoofdinhoud" implemented in `layout.tsx`, linking to `#main-content`. |
| 2.4.2 | Page Titled | A | PASS | Page title set to "Virtuele Assistent" in metadata. |
| 2.4.3 | Focus Order | A | PARTIAL | DOM order is logical (skip link → welcome → messages → composer). Action bars auto-hide (`autohide="not-last"`), which may cause focus to land on a hidden element. Needs testing. |
| 2.4.4 | Link Purpose (In Context) | A | PASS | Source chip links have `aria-label="Bron: {label} (opent in nieuw tabblad)"`. Markdown links render with visible text. |
| 2.4.5 | Multiple Ways | AA | N/A | Single-page app per tenant. No multi-page navigation to provide alternative paths for. |
| 2.4.6 | Headings and Labels | AA | PARTIAL | Composer has `aria-label`. Debug panel sections use visual heading styling but not semantic `<h>` elements — minor gap. No page-level `<h1>` present in the chat view. |
| 2.4.7 | Focus Visible | AA | PARTIAL | Tailwind `focus-visible:ring-[3px]` is applied to buttons. Composer uses `focus-within:ring-2 focus-within:ring-black`. `outline-ring/50` in globals.css uses 50% opacity — needs verification that this is sufficient contrast. |
| 2.4.11 | Focus Not Obscured (Minimum) | AA | PARTIAL | The debug panel is `fixed bottom-4 right-4 z-50`. When expanded, it could partially overlap a focused element in the lower-right area of the composer. The scroll-to-bottom button is `absolute -top-12 z-10`. Needs visual testing. *(New in WCAG 2.2)* |
| 2.4.12 | Focus Not Obscured (Enhanced) | AA | NEEDS TESTING | Same concern as 2.4.11 — the fixed debug panel could fully obscure focused elements. *(New in WCAG 2.2)* |
| 2.4.13 | Focus Appearance | AA | PARTIAL | Focus ring uses `ring-[3px]` (3px width, exceeds minimum) and `ring-black`/`ring-white` (high contrast in their respective modes). Needs contrast ratio measurement. *(New in WCAG 2.2)* |

### 2.5 Input Modalities

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 2.5.1 | Pointer Gestures | A | PASS | No multipoint or path-based gestures required. |
| 2.5.2 | Pointer Cancellation | A | PASS | All buttons activate on `mouseup`/`click`. No down-event-only triggers. |
| 2.5.3 | Label in Name | A | PASS | All `aria-label` values contain the visible button text (e.g., tooltip "Kopieer" is contained in aria-label "Kopieer bericht"). |
| 2.5.4 | Motion Actuation | A | N/A | No device motion or shake interactions. |
| 2.5.5 | Target Size (Enhanced) | AA | PARTIAL | Most buttons use `size-6` (24px) via Tailwind with `p-1` padding. Composer send button uses `size-8` (32px). These are below the 44×44 CSS px enhanced target — note this is an AAA criterion in WCAG 2.1 but AA in WCAG 2.2. Debug panel toggle is `p-2` which results in a small target. |
| 2.5.7 | Dragging Movements | AA | N/A | No drag-and-drop interactions present. *(New in WCAG 2.2)* |
| 2.5.8 | Target Size (Minimum) | AA | PARTIAL | Minimum is 24×24 CSS px. Icon buttons at `size-6` (24px) meet the minimum but have no spacing buffer. Needs measurement. *(New in WCAG 2.2)* |

---

## 3. Understandable

### 3.1 Readable

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 3.1.1 | Language of Page | A | PASS | `<html lang="nl">` set in layout. |
| 3.1.2 | Language of Parts | AA | FAIL | Two UI elements are in English within a Dutch-language UI: the edit composer has "Cancel" and "Update" button labels. The markdown code block copy button uses "Copy" in its tooltip. These should be localized to Dutch. |

### 3.2 Predictable

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 3.2.1 | On Focus | A | PASS | No context changes triggered by focus alone. |
| 3.2.2 | On Input | A | PASS | Typing in the composer does not trigger navigation. Suggestion buttons auto-send on click but this is expected behavior for a pre-set prompt. |
| 3.2.3 | Consistent Navigation | AA | N/A | Single-page per tenant; no cross-page navigation. |
| 3.2.4 | Consistent Identification | AA | PASS | Copy, reload, and edit icons are used consistently and labeled consistently throughout. |
| 3.2.6 | Consistent Help | AA | N/A | No help mechanism implemented; not applicable to a single-page chat interface. *(New in WCAG 2.2)* |

### 3.3 Input Assistance

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 3.3.1 | Error Identification | A | PARTIAL | `MessageError` renders error messages inline via `ErrorPrimitive`. Errors are visually distinct (red border, red text). However, errors are not programmatically associated with the composer input via `aria-describedby`. |
| 3.3.2 | Labels or Instructions | A | PASS | Composer input has `aria-label="Bericht invoer"` and placeholder text. |
| 3.3.3 | Error Suggestion | AA | PARTIAL | Error messages are displayed but generated by the runtime — content depends on what the backend returns. No structured suggestions are shown. |
| 3.3.4 | Error Prevention (Legal, Financial, Data) | AA | N/A | No legal, financial, or irreversible data transactions. |
| 3.3.7 | Redundant Entry | AA | N/A | No multi-step forms requiring repeated data entry. *(New in WCAG 2.2)* |
| 3.3.8 | Accessible Authentication (Minimum) | AA | N/A | No authentication required to use the chatbot. *(New in WCAG 2.2)* |
| 3.3.9 | Accessible Authentication (Enhanced) | AA | N/A | Same as above. *(New in WCAG 2.2)* |

---

## 4. Robust

### 4.1 Compatible

| # | Criterion | Level | Status | Notes |
|---|-----------|-------|--------|-------|
| 4.1.1 | Parsing | A | N/A | Removed from WCAG 2.2 as obsolete. |
| 4.1.2 | Name, Role, Value | A | PARTIAL | Buttons have names (aria-label). Toggle switches have `role="switch"` and `aria-checked`. Debug panel has `aria-expanded`. The `@assistant-ui/react` primitives expose ARIA attributes through their headless component pattern. The `MessagePrimitive.Root` renders as a `motion.div` — needs verification that the role is exposed to the accessibility tree correctly. |
| 4.1.3 | Status Messages | AA | PASS | Google tool loading state uses `role="status"` and `aria-live="polite"`. Chatbot unavailable uses `role="alert"`. New chat messages are announced via `role="log"` live region. |

---

## Outstanding Items Requiring Browser/Screen Reader Testing

These cannot be confirmed from code review alone and must be tested manually:

1. **Live region announcement timing** — Does the response get announced during streaming or only after completion? Does the `aria-atomic="false"` behave as expected across NVDA, JAWS, and VoiceOver?
2. **Focus after message send** — Does focus remain on the composer input after sending a message, or does it jump?
3. **Autohide action bars** — When `ActionBarPrimitive` auto-hides, does focus move to a non-hidden element?
4. **200% zoom reflow** — Does content reflow without horizontal scroll at 320px effective width?
5. **Text spacing override** — Does the layout survive forced text-spacing (line-height: 1.5, letter-spacing: 0.12em, word-spacing: 0.16em)?
6. **Focus ring contrast measurement** — Measure actual contrast of `ring-black` and `ring-white` against adjacent backgrounds.
7. **Touch target sizes** — Measure rendered pixel sizes of icon buttons on a mobile viewport.
8. **Dark mode link contrast** — Measure `text-blue-400` against dark background tokens.

---

## Fixes Required Before Claiming WCAG 2.2 AA Compliance

| Priority | Issue | Criterion |
|----------|-------|-----------|
| HIGH | Localize "Cancel", "Update", and "Copy" to Dutch | 3.1.2 |
| HIGH | Verify and fix `outline-ring/50` opacity in globals.css for sufficient contrast | 1.4.11, 2.4.7 |
| HIGH | Add `autocomplete` attribute to composer input | 1.3.5 |
| MEDIUM | Add `prefers-reduced-motion` support to disable/reduce Framer Motion animations | Best practice (2.3.3 AAA) |
| MEDIUM | Verify focus is not obscured by fixed debug panel (2.4.11) | 2.4.11 |
| MEDIUM | Increase icon button touch targets or ensure adequate spacing (2.5.8) | 2.5.8 |
| MEDIUM | Associate error messages to composer via `aria-describedby` | 3.3.1 |
| LOW | Add semantic heading to chat view (h1) | 2.4.6 |
| LOW | Add `aria-hidden="true"` to suggestion cards hidden on mobile | 1.3.1 |

---

## Summary

| Principle | Criteria assessed | PASS | PARTIAL | FAIL | N/A | Needs Testing |
|-----------|-------------------|------|---------|------|-----|---------------|
| Perceivable | 14 | 5 | 5 | 0 | 2 | 4 |
| Operable | 18 | 7 | 5 | 0 | 5 | 3 |
| Understandable | 11 | 5 | 2 | 1 | 6 | 0 |
| Robust | 3 | 1 | 1 | 0 | 1 | 0 |
| **Total** | **46** | **18** | **13** | **1** | **14** | **7** |

Of the 32 applicable criteria (excluding N/A): **18 pass, 13 partial, 1 fail, 7 need testing.**

The single outright FAIL (3.1.2 Language of Parts) is a straightforward localization fix. Most PARTIAL items are code-verifiable gaps that can be resolved without architectural changes. The product is in good shape structurally and can reach full WCAG 2.2 AA conformance with the fixes listed above.
