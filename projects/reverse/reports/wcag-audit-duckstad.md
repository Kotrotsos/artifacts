# WCAG 2.1 AA Accessibility Audit: Virtuele Assistent Duckstad (Gemeente Meppel)

**URL:** https://virtuele-assistent.montferland.info/demo_duckstad
**Date:** 2026-04-10
**Auditor:** Automated code-level audit (HTML source, CSS, JavaScript)
**Standard:** WCAG 2.1 Level AA

**Tech stack:** Next.js (SSR), React, Tailwind CSS v4 (oklch colors), assistant-ui library, Radix UI primitives (tooltips), Framer Motion (animations), Geist font family.

---

## Executive Summary

The chatbot has several critical accessibility failures. The most severe are: (1) the page language is set to English instead of Dutch, (2) no ARIA live region exists for announcing new chat messages to screen readers, (3) the page title is a generic developer placeholder, (4) the subtitle text fails color contrast requirements, and (5) there are no skip-navigation links or landmark roles. The application does have some positive accessibility features, including Dutch aria-labels on interactive controls, focus-visible ring styles, and sr-only text for icon buttons.

**Summary counts:**
- PASS: 16
- FAIL: 15
- NEEDS MANUAL CHECK: 7

---

## 1. Perceivable

### 1.1.1 Non-text Content -- PASS (with caveats)

**Evidence:**
- All icon-only buttons have appropriate accessible names:
  - Scroll-to-bottom button: `<span class="sr-only">Scroll to bottom</span>` (English, not Dutch)
  - File attachment button: `<span class="sr-only">Bestand toevoegen</span>`
  - Send button: `aria-label="Bericht versturen"`
  - Stop generation: `aria-label="Stop genereren"`
- SVG icons use `aria-hidden="true"`, correctly hiding decorative icons from AT.
- **Minor issue:** "Scroll to bottom" sr-only text is in English while the rest of the interface is Dutch. Tooltip text is also "Scroll to bottom" and "Copy" in English.

### 1.2 Time-based Media -- N/A

No audio or video content detected.

### 1.3.1 Info and Relationships -- FAIL

**Evidence:**
- **No landmark roles:** The page has no `<main>`, `<nav>`, `<header>`, `<footer>`, or ARIA landmark roles. The entire page is a flat `<div>` structure inside `<body>`.
- **No heading hierarchy in the rendered HTML:** The welcome text uses `<div class="text-2xl font-semibold">` instead of a proper heading element (`<h1>`, `<h2>`). Screen readers cannot navigate by headings.
- **Chat messages** use `data-role="assistant"` and `data-role="user"` custom attributes, which are not recognized by assistive technology. No `role="log"`, `role="list"`, or semantic grouping is used.
- **Form structure:** The `<form>` element exists and the `<textarea>` has `aria-label="Bericht invoer"`, which is good. However, there is no visible `<label>` element associated with the textarea.
- **Suggestion buttons** are in a grid but not marked up as a list.

### 1.3.2 Meaningful Sequence -- PASS

The DOM order follows a logical reading sequence: welcome message at top, suggestion buttons in the middle, input area at the bottom. This matches the visual presentation.

### 1.3.3 Sensory Characteristics -- PASS

No instructions rely solely on shape, size, visual location, or sound.

### 1.3.4 Orientation -- PASS

No CSS or JS enforces a specific orientation. The layout uses `flex-col` with responsive classes that adapt to viewport size. `h-dvh` uses dynamic viewport height.

### 1.3.5 Identify Input Purpose -- FAIL

**Evidence:**
- The chat textarea (`<textarea name="input">`) lacks an `autocomplete` attribute. For a general message input this is somewhat acceptable, but there is no `autocomplete` attribute anywhere in the codebase.
- No `inputmode` attribute is set on the textarea.

### 1.4.1 Use of Color -- NEEDS MANUAL CHECK

**Evidence:**
- The suggestion buttons use border styling as the primary visual differentiator. They have text labels, so color is not the sole means of conveying information in the static state.
- Error states use `bg-destructive` (red) but the JS shows `aria-invalid` is also set, providing a non-color indicator.
- Chat message differentiation between user and assistant messages should be verified visually to ensure it does not rely solely on color.

### 1.4.2 Audio Control -- N/A

No audio content detected.

### 1.4.3 Contrast (Minimum) -- FAIL

**Evidence (computed approximate contrast ratios):**

| Element | Foreground | Background | Ratio | Required | Result |
|---------|-----------|------------|-------|----------|--------|
| Body text (foreground on background) | #18181b | #ffffff | 17.1:1 | 4.5:1 | PASS |
| Primary button text | #fbfbfb | #18181b | 17.1:1 | 4.5:1 | PASS |
| Muted-foreground on white | #71717a | #ffffff | 4.8:1 | 4.5:1 | PASS |
| **Subtitle text (muted-fg at 65% opacity)** | **~#a2a2a8** | **#ffffff** | **2.5:1** | **4.5:1** | **FAIL** |
| Placeholder text (muted-fg on muted bg) | #71717a | #f4f4f5 | 4.4:1 | 4.5:1 | **FAIL (borderline)** |
| Tooltip text (primary-fg on primary) | #fbfbfb | #18181b | 17.1:1 | 4.5:1 | PASS |

- **Critical failure:** The welcome subtitle "Hoe kan ik u vandaag helpen?" uses class `text-muted-foreground/65`, which applies 65% opacity to an already medium-gray color, producing an approximate contrast ratio of only **2.5:1** against white, far below the 4.5:1 minimum.
- **Borderline failure:** The textarea placeholder "Stel uw vraag aan de gemeente..." uses `text-muted-foreground` on `bg-muted`, yielding approximately **4.4:1**, just below the 4.5:1 threshold for normal text.

### 1.4.4 Resize Text -- NEEDS MANUAL CHECK

**Evidence:**
- Font sizes use relative units (`text-sm`, `text-base`, `text-2xl`) via Tailwind, which map to rem values. This is positive.
- The layout uses `max-h-[calc(50dvh)]` for the textarea and `h-dvh` for the page, which could cause issues at 200% zoom.
- Requires browser testing to confirm no content is cut off at 200% text zoom.

### 1.4.5 Images of Text -- PASS

No images of text detected. All text is rendered as actual text.

### 1.4.10 Reflow -- NEEDS MANUAL CHECK

**Evidence:**
- Responsive classes are present: `sm:grid-cols-2`, `md:mt-20`, `md:pb-6`, responsive padding.
- `max-w-[var(--thread-max-width)]` with `--thread-max-width: 48rem` and `--thread-padding-x: 1rem` suggests the layout should reflow.
- Third and fourth suggestion buttons use `[&:nth-child(n+3)]:hidden sm:[&:nth-child(n+3)]:block`, hiding them on small screens.
- Requires testing at 320px CSS width to confirm no horizontal scrolling.

### 1.4.11 Non-text Contrast -- FAIL

**Evidence:**
- **Border color on white background:** The suggestion button borders use `border-border` which resolves to oklch(92%) ~ #e4e4e7. Against the white background, this produces a contrast ratio of approximately **1.3:1**, far below the 3:1 minimum required for UI components.
- **Input border:** Same border color is used for the textarea borders, also failing at ~1.3:1.
- The send button when disabled loses additional contrast due to `disabled:opacity-50`.

### 1.4.12 Text Spacing -- NEEDS MANUAL CHECK

**Evidence:**
- No CSS properties explicitly prevent text spacing adjustment (no `!important` on line-height, letter-spacing, word-spacing, or paragraph spacing in the component CSS).
- The `whitespace-nowrap` class on buttons could cause issues if letter-spacing is increased. Needs testing with custom text spacing overrides.

### 1.4.13 Content on Hover or Focus -- PASS

**Evidence:**
- Tooltips use Radix UI Tooltip primitives, which follow proper patterns: they appear on hover/focus of the trigger, are dismissible, and persistent while hovering.
- Tooltip content uses `data-[state=closed]:animate-out` for dismissal animation.

---

## 2. Operable

### 2.1.1 Keyboard Accessible -- FAIL

**Evidence:**
- All buttons are native `<button>` elements, so they are keyboard-focusable by default.
- The textarea is a native `<textarea>`, which is keyboard-accessible.
- **However:** No keyboard shortcut is evident for submitting the form (e.g., Enter to send). The form uses `<form>` with a submit button, so Enter in the textarea may submit, but multi-line input typically requires Shift+Enter. The JS does not show explicit keyboard handling (`onKeyDown` is not found in the page bundle).
- The suggestion buttons are regular buttons and should be keyboard-accessible.
- **Issue:** The scroll-to-bottom button is visually hidden with `disabled:invisible` when disabled, which is acceptable.

### 2.1.2 No Keyboard Trap -- PASS

**Evidence:**
- The page uses standard HTML form elements (buttons, textarea) without custom focus trapping.
- No modal dialogs or custom focus management that could trap keyboard users.

### 2.1.4 Character Key Shortcuts -- PASS

No single-character keyboard shortcuts detected in the JS bundle.

### 2.4.1 Bypass Blocks -- FAIL

**Evidence:**
- No skip-navigation link is present in the HTML.
- No ARIA landmarks (`role="main"`, `<main>`, `<nav>`, etc.) are defined, so screen reader users cannot navigate by landmarks.
- For a single-purpose chatbot this is less severe, but still a failure.

### 2.4.2 Page Titled -- FAIL

**Evidence:**
- `<title>assistant-ui Starter App</title>` -- This is the default developer/template title, not a meaningful page title.
- Should be something like "Virtuele Assistent - Gemeente Meppel" or a Dutch equivalent.
- The `<meta name="description">` is also generic: "Generated by create-assistant-ui".

### 2.4.3 Focus Order -- PASS

**Evidence:**
- No positive `tabindex` values detected.
- DOM order matches visual order: welcome text, suggestion buttons, textarea, action buttons.
- Focus should flow logically through the interface.

### 2.4.4 Link Purpose -- PASS

No links are present in the initial page state. Chat responses may contain links generated dynamically. The markdown renderer in the JS includes link styling (`underline underline-offset-4`), but link text content depends on the AI response.

### 2.4.5 Multiple Ways -- FAIL

**Evidence:**
- The chatbot is a single-page application with no sitemap, search function, or alternative navigation method to reach content.
- This is inherent to the chatbot format but technically fails this criterion for the broader site context.

### 2.4.6 Headings and Labels -- FAIL

**Evidence:**
- No heading elements (`<h1>`-`<h6>`) exist in the rendered HTML. The welcome text uses styled `<div>` elements.
- The textarea has `aria-label="Bericht invoer"` (good).
- Buttons have aria-labels (good).
- But the overall page lacks any heading structure.

### 2.4.7 Focus Visible -- PASS

**Evidence:**
- Buttons use `focus-visible:ring-ring/50 focus-visible:ring-[3px]` and `focus-visible:border-ring`, providing a visible 3px ring on focus.
- The textarea container uses `focus-within:ring-2 focus-within:ring-black`, providing a black 2px ring.
- These are standard, visible focus indicators.

### 2.5.1 Pointer Gestures -- PASS

No multi-point or path-based gestures detected. All interactions use simple clicks/taps.

### 2.5.2 Pointer Cancellation -- PASS

All buttons use standard click events (activated on `pointerup`/`click`, not `pointerdown`). Native HTML button behavior allows cancellation by moving the pointer away.

### 2.5.3 Label in Name -- PASS

**Evidence:**
- Suggestion buttons have `aria-label` values that match their visible text content. For example: visible text "Wat zijn de openingstijden van het gemeentehuis?" matches `aria-label="Wat zijn de openingstijden van het gemeentehuis?"`.
- Icon buttons use sr-only text or aria-labels that describe the icon's function.

---

## 3. Understandable

### 3.1.1 Language of Page -- FAIL

**Evidence:**
- `<html lang="en">` -- The page language is set to English, but the interface is entirely in Dutch.
- Must be `<html lang="nl">` for proper screen reader pronunciation.
- This is a **critical** accessibility failure. Screen readers will attempt to pronounce all Dutch text using English phonetics, making the interface unintelligible.

### 3.1.2 Language of Parts -- FAIL

**Evidence:**
- Several text strings are in English within an otherwise Dutch interface:
  - "Scroll to bottom" (sr-only text and tooltip)
  - "Copy" (tooltip on copy button)
  - "assistant-ui Starter App" (page title)
  - "Open debug panel" / "Collapse debug panel" (aria-labels)
- None of these English strings are marked with `lang="en"`.

### 3.2.1 On Focus -- PASS

No context changes occur on focus. Buttons require activation (click/Enter) to trigger actions.

### 3.2.2 On Input -- PASS

The textarea does not trigger actions on input. The form requires explicit submission via the send button.

### 3.2.3 Consistent Navigation -- PASS

Single-page application with a single consistent layout. No navigation components that could be inconsistent.

### 3.2.4 Consistent Identification -- PASS

UI components are used consistently: all buttons follow the same pattern, the textarea is the single input method.

### 3.3.1 Error Identification -- NEEDS MANUAL CHECK

**Evidence:**
- The JS shows `console.error("Failed to fetch welcome message:")` but no visible error UI for failed API calls.
- The `aria-invalid` attribute support exists in the button styles, suggesting form validation may be handled.
- Need to test actual error scenarios (network failure, empty submission, API errors) to verify error messages are displayed and accessible.

### 3.3.2 Labels or Instructions -- PASS (partial)

**Evidence:**
- The textarea has `aria-label="Bericht invoer"` and `placeholder="Stel uw vraag aan de gemeente..."`.
- The welcome message provides context: "Welkom bij de assistent van gemeente Meppel."
- Suggestion buttons provide example queries, acting as instructions.
- **Minor:** The placeholder text disappears on input, so the label relies solely on `aria-label`.

### 3.3.3 Error Suggestion -- NEEDS MANUAL CHECK

Cannot verify from code alone. Depends on runtime behavior when invalid input or API errors occur.

### 3.3.4 Error Prevention -- PASS

The chatbot is a conversational interface with no legal, financial, or data-deletion actions. Messages can be edited (aria-label "Bericht bijwerken" exists) and the interface does not commit irreversible actions.

---

## 4. Robust

### 4.1.1 Parsing -- PASS

**Evidence:**
- The HTML is well-formed, generated by Next.js server rendering.
- No duplicate IDs detected in the HTML source.
- Elements are properly nested and closed.
- Note: WCAG 2.1 has deprecated this criterion as modern browsers handle parsing errors, but the HTML is valid regardless.

### 4.1.2 Name, Role, Value -- FAIL

**Evidence:**
- **Buttons** have proper roles (native `<button>` elements) and accessible names (aria-labels).
- **Textarea** has a proper role (native) and accessible name.
- **Chat messages** use only `data-role="assistant"` and `data-role="user"` custom attributes. These are not recognized by assistive technology. Messages should use proper ARIA roles or semantic elements.
- The toggle switch uses `role="switch"` with `aria-checked`, which is correct.
- **Issue:** The scroll-to-bottom button uses `disabled=""` attribute, which is correct for state.

### 4.1.3 Status Messages -- FAIL

**Evidence:**
- **No `aria-live` region detected** anywhere in the HTML or JavaScript. This is the most critical chat-specific accessibility failure.
- When the assistant responds, new content appears in the chat area, but screen reader users will not be notified of new messages.
- The loading spinner (`animate-spin`) has no accessible text alternative to announce "loading" or "generating response" states.
- The running/typing indicator (CSS `aui-pulse` animation with dot character) is purely visual.
- Chat messages need to be wrapped in or announced via an `aria-live="polite"` region, or use `role="log"` with `aria-live="polite"`.

---

## Critical Issues (Priority Order)

1. **No ARIA live region for chat messages (4.1.3)** -- Screen reader users cannot know when the assistant responds. This is the single most impactful accessibility barrier for a chat application.

2. **Wrong page language: `lang="en"` instead of `lang="nl"` (3.1.1)** -- Screen readers will mispronounce all Dutch text, making the entire interface unusable for Dutch-speaking screen reader users.

3. **Subtitle text contrast ratio of 2.5:1 (1.4.3)** -- The `text-muted-foreground/65` class on "Hoe kan ik u vandaag helpen?" produces a contrast ratio far below the 4.5:1 minimum.

4. **No landmark regions (1.3.1)** -- Screen reader users cannot orient themselves on the page. The entire content is an undifferentiated `<div>` tree.

5. **Generic page title (2.4.2)** -- "assistant-ui Starter App" provides no useful information. Screen reader users hearing this title will not understand what page they are on.

6. **No heading structure (2.4.6 / 1.3.1)** -- Welcome text uses styled divs, not headings. Screen readers cannot navigate by heading.

7. **UI component borders fail non-text contrast (1.4.11)** -- Button and input borders at ~1.3:1 contrast are barely visible to users with low vision.

8. **No skip navigation (2.4.1)** -- Though less critical for a single-purpose chatbot, it remains a failure.

---

## Positive Findings

- Dutch aria-labels on core interactive elements ("Bericht invoer", "Bericht versturen", "Stop genereren", "Bestand toevoegen")
- Proper `aria-hidden="true"` on decorative SVG icons
- `sr-only` class used for icon button text alternatives
- Focus-visible ring indicators on all interactive elements (3px ring)
- Native HTML form elements (button, textarea, form) used correctly
- Logical DOM reading order matching visual layout
- Responsive design with mobile considerations
- Suggestion buttons include full-text aria-labels matching visible content
- Edit/cancel functionality for messages (aria-labels present)
- Radix UI tooltip primitives with proper behavior

---

## Recommendations

### Immediate Fixes (Critical)

1. Change `<html lang="en">` to `<html lang="nl">`
2. Add `aria-live="polite"` to the chat message container, or wrap it in `<div role="log" aria-live="polite">`
3. Change `<title>` to a descriptive Dutch title, e.g. "Virtuele Assistent - Gemeente Meppel"
4. Increase subtitle text contrast: remove the `/65` opacity modifier from `text-muted-foreground/65`, or use a darker color
5. Add `<main>` landmark around the chat interface
6. Convert welcome text divs to proper heading elements (`<h1>`, `<h2>`)

### Important Fixes

7. Increase border contrast for buttons and inputs (use a darker border color, e.g. `--border: oklch(75% .004 286)`)
8. Add a skip-to-content link before the suggestion buttons
9. Mark English text strings with `lang="en"` attribute, or translate to Dutch ("Scroll to bottom" to "Naar beneden scrollen", "Copy" to "Kopieren")
10. Add `<meta name="description">` with meaningful Dutch content
11. Add visible label or persistent instructions for the textarea

### Nice-to-Have Improvements

12. Add `role="log"` to the chat message container
13. Add `prefers-reduced-motion` media query to respect user motion preferences (no motion queries found in CSS or JS)
14. Add `aria-label` to the form element describing its purpose
15. Consider `autocomplete="off"` on the chat textarea to prevent browser autofill interference
16. Add loading state announcement (e.g., `aria-busy="true"` on the message container while generating)
