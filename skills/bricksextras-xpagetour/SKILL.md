---
name: xpagetour
description: "Use when building or debugging a guided product tour / walkthrough with the Page Tour (xpagetour) and Page Tour Step (xpagetourstep) elements from BricksExtras — Shepherd.js-based popovers that point at page elements in sequence. Covers the trigger/show-again config, targeting elements by class via stepSelector, the button-style controls, and why step content lives in an invisible <template> until the tour runs."
---

**Requires:** BricksExtras 1.7.3+ with xpagetour and xpagetourstep elements enabled

# BricksExtras: Page Tour (xpagetour) + Page Tour Step (xpagetourstep)

Shipped by the **BricksExtras** plugin. Wraps **Shepherd.js** to build a guided walkthrough: a sequence of popovers, each pointing at (or centered over) an element on the page, with Back/Next/Finish navigation. `xpagetour` is the nestable parent (one instance = one whole tour, holds tour-wide config). `xpagetourstep` is a nestable child of it (one instance = one step in the sequence).


**Before building from any template below, read both elements' schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xpagetour.json` and `references/elements/xpagetourstep.json` (for the step child) inside the `bricksextras-element-schemas` skill directory and read them. If either file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for that element instead. The templates and examples below show documented required structure and common patterns only — the schema files (or live calls) are the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened these sources this session.

---

## Structure

```
xpagetour (tour-wide config: trigger, show-again, overlay, progress, nav buttons, etc.)
├── xpagetourstep (stepTitle, stepSelector, stepPosition, ...)
│   └── [any elements — the step's popover body content]
├── xpagetourstep
│   └── [...]
└── ...
```

Each step's own children are its popover body content — any elements (`text-basic`, `heading`, `image`, etc.), rendered inside `.shepherd-text`.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xpagetour.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xpagetour",
  "label": "Homepage Tour",
  "settings": {
    "trigger": "page_load",
    "show_again": "page_load",
    "navButtons": "enable",
    "showProgress": true,
    "useModalOverlay": true,
    "defaultHighlight": true,
    "defaultScrollTo": true
  },
  "children": [
    {
      "name": "xpagetourstep",
      "label": "Step 1",
      "settings": { "stepTitle": "The Player", "stepSelector": ".tour-target-player", "stepPosition": "bottom" },
      "children": [ { "name": "text-basic", "settings": { "text": "This is the video player." } } ]
    },
    {
      "name": "xpagetourstep",
      "label": "Step 2",
      "settings": { "stepTitle": "The Playlist", "stepSelector": ".tour-target-playlist", "stepPosition": "left" },
      "children": [ { "name": "text-basic", "settings": { "text": "Click any track to play it." } } ]
    }
  ]
}
```

---

## Targeting: `stepSelector` is a class, same rule as everywhere else

`stepSelector` (on the step, not the tour) is a plain CSS selector for the element this step should point at — leave it blank to center the step in the viewport instead. Give the actual target element a distinguishing class via `_cssClasses` and reference `.thatClass` here — never `_cssId`, same reasoning as every other targeting field in this plugin (see `bricksextras-start-here`). With `defaultHighlight`/`defaultScrollTo` on (set on the tour, see below), the target element gets an `x-page-tour-highlight` class and `shepherd-target` marker, and the popover correctly positions itself relative to it per `stepPosition`.

**`defaultHighlight`/`defaultScrollTo` are tour-wide settings only** — set them on `xpagetour`, not on individual `xpagetourstep`s. There is no per-step override for highlight/scroll behavior; every step in a tour shares the same highlight/scroll setting from its parent tour.

`componentScope` (on the tour, string `"true"`/`"false"`, not boolean) follows the same component-instance-scoping pattern as `xproslidercontrol`/`xproslider` sync — set it when the tour lives inside a component that might be duplicated.

---

## Step content renders inside an inert `<template>` — don't expect to see it in a plain DOM read

`xpagetourstep` renders its children wrapped in a literal `<template>{children}</template>` tag. Browsers never render `<template>` contents as visible DOM, and a page load without the tour actually running (or a raw `get-page-elements` read) will not show the step's content as visible markup — Shepherd.js reads each step's `<template>` and clones its content into a real `.shepherd-content`/`.shepherd-text` popover at runtime, one step at a time, appended near `<body>`. **Verify by triggering the tour and inspecting the live `.shepherd-element` in the browser, not by reading the static page HTML.** `document.querySelector('.shepherd-element')` only exists once the tour has actually started.

**Each step transition creates a new `.shepherd-element` — but the previous one isn't removed, it's left in the DOM with a `hidden` attribute.** Confirmed live: after clicking Next, the step-0 `<dialog>` persists with `hidden=""` rather than being deleted, while a new step-1 `<dialog>` is created and shown. A plain `document.querySelectorAll('.shepherd-element')` after any navigation returns every step visited so far, not just the current one — filter on `:not([hidden])` (or query the single active one via `[open]`) when checking the current step's content, not the raw class alone.

The tour's own wrapping `<div class="x-page-tour" data-x-page-tour="{...json config}">` **is** present in the raw HTML — the whole tour-wide config is serialized into that one attribute and read by `page-tour.js` on load.

### Rendered `.shepherd-element` structure (confirmed live)

The popover root is a real `<dialog>` element, carrying both Shepherd's own classes and this element's own (`brxe-xpagetourstep`):

```html
<dialog class="shepherd-element shepherd-has-cancel-icon shepherd-has-title brxe-xpagetourstep extras-theme shepherd-enabled"
        data-shepherd-step-id="step-0" open aria-labelledby="step-0-label" aria-describedby="step-0-description">
  <div class="shepherd-arrow" data-popper-arrow></div>
  <div class="shepherd-content">
    <header class="shepherd-header">
      <h3 id="step-0-label" class="shepherd-title">The Target</h3>
      <button class="shepherd-cancel-icon" aria-label="Close Tour" type="button"><span aria-hidden="true">×</span></button>
    </header>
    <div class="shepherd-text" id="step-0-description">
      <div class="brxe-text-basic">This is the target section.</div> <!-- the step's own nested children -->
    </div>
    <footer class="shepherd-footer">
      <button class="x-page-tour-button x-page-tour-button__back shepherd-button" type="button">Back</button>
      <button class="x-page-tour-button x-page-tour-button__complete shepherd-button" type="button">Finish</button>
    </footer>
  </div>
  <div class="shepherd-progress-container">
    <div class="shepherd-progress-bar" role="progressbar" aria-valuenow="100" aria-valuetext="Step 2 of 2"></div>
  </div>
</dialog>
```

Notes:

- **A step with no `stepSelector` (centered) gets an additional `.shepherd-centered` class** on the root `<dialog>`, alongside everything else — a real, distinct hook for styling centered steps differently from pointed ones.
- **This element's own button classes (`x-page-tour-button__next`/`__back`/`__complete`, per the "Button Style controls" section above) sit alongside Shepherd's own `.shepherd-button` class on the same `<button>`** — both are present together, not an either/or.
- The first step in a tour only gets a `Next` button (no `Back`); the last step gets `Back` + `Finish` instead of `Back` + `Next` — the footer's button set changes per step position, not just their text.
- `.shepherd-progress-bar`'s `aria-valuenow`/`aria-valuetext`/inline `width` are real, live-updated per step — not static markup.

---

## Button Style controls

Use `nextButtonBackground`/`backButtonBackground`/`completeButtonBackground` (+ Typography/Border/BoxShadow/Padding for each), gated by `navButtons != 'disable'` and targeting the actual rendered classes `.x-page-tour-button__next` / `.x-page-tour-button__back` / `.x-page-tour-button__complete`.

Per-step button text (`buttonPrevText`/`buttonNextText`/`buttonCloseText` on `xpagetourstep`) overrides the tour-wide default text for that step only.

---

## Custom navigation buttons instead of the built-in ones

Set `navButtons: "disable"` on the tour to remove the built-in Back/Next/Finish/Skip buttons entirely, then build your own buttons/links anywhere on the page with the classes `customBackSelector`/`customNextSelector`/`customCompleteSelector`/`customSkipSelector` point at (defaults: `.x-tour-back` / `.x-tour-next` / `.x-tour-complete` / `.x-tour-skip` — same "default is a UI placeholder, wire an explicit class" caveat as elsewhere). `autoHideCustomButtons` conditionally shows/hides them based on step state (e.g. hide Back on the first step).

---

## Other tour-wide settings worth knowing

- **`trigger`**: `page_load` (default, auto-shows on load) / `click` (needs `clickSelector`) / `interaction` / `window_event` (needs `trigger_event`).
- **`show_again`**: only relevant when `trigger != 'click'` — `page_load` (show every visit, default) / `never` / `until_complete` / `after` (needs `show_again_days`/`show_again_hours`) / `manual` (control via code).
- **`rememberProgress`** + `autoResumeOnPageLoad` (default true) + `saveNextStep` (`current` default / `next`, useful for tours spanning multiple pages) + `clearProgressOnCancel` (default true) — persistence for multi-page or interrupted tours.
- **`useModalOverlay`** + `overlayColor` + `exitOnOverlayClick` + `canClickTarget` (+ `modalOverlayOpeningPadding`/`Radius` when `canClickTarget: true`) — darkens the rest of the page and cuts a hole around the current target.
- **`showProgress`** + `progressPosition` (top/bottom) + `progressHeight`/`progressColor`/`progressBackground`.
- **`builderHidden`** (checkbox) — hides the tour only while working in the Bricks builder canvas; no effect on the frontend.

---

## Never do

- Do not expect step content to appear in a plain `get-page-elements`/raw-HTML check — it's inside a `<template>` tag, invisible until Shepherd runs. Trigger the tour and inspect the live `.shepherd-element` instead.
- Do not target elements with `_cssId` in `stepSelector` — use a `_cssClasses` class, same rule as every other targeting field in this plugin.
- Do not look for a per-step highlight/scroll override — `defaultHighlight`/`defaultScrollTo` are tour-wide only, set on `xpagetour`, and apply to every step uniformly.
- Do not assume `.shepherd-element` alone always matches only the current step — previous steps' dialogs persist in the DOM with `hidden` after navigating rather than being removed; filter on `:not([hidden])` or `[open]` when querying the active one.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real Shepherd.js instance from `window.xPageTour.Instances[dataXId]` (keyed by the `data-x-id` on the `.x-page-tour[data-x-page-tour]` root) in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), and drive it directly via Shepherd's own API (`tour.next()`, `tour.show()`, etc.). It's registered synchronously on init, no artificial delay to wait out.
