---
name: xcolorschemetoggle
description: "Use when building or debugging the Color Scheme Toggle element (xcolorschemetoggle) from BricksExtras: a light/dark/system theme switcher (dropdown, radio buttons, single button, or switch styles). Covers how it plugs into Bricks core's own dark-mode system, the 2-state vs 3-state (system) config, and the auto/system caveat."
---

**Requires:** BricksExtras 1.7.3+ with xcolorschemetoggle element enabled

# BricksExtras: Color Scheme Toggle (xcolorschemetoggle)

Shipped by the **BricksExtras** plugin. A UI control (dropdown, radio-button group, single button, or switch) for switching between light/dark/system color modes. **Not nestable.**

## It's a UI skin over Bricks core's own dark-mode system — not its own theming engine

This is the single most important fact about the element. It does **not** invent a parallel theming system. Bricks core already has a dark-mode mechanism: an `data-brx-theme="light"|"dark"` attribute on `<html>`, a `localStorage.brx_mode` value, and (in the design system) colors with `darkModeEnabled: true` carrying separate `light`/`dark` values that resolve against that attribute. Bricks' own Style Manager has a `defaultMode` setting (`light`/`dark`/`auto`) for the initial state before any user choice is stored.

This element's JS reads that existing state to set its own button/dropdown UI (which option looks "selected"), and on click, **writes to the exact same two places Bricks itself uses** — it sets `document.documentElement.dataset.brxTheme` and `localStorage.setItem('brx_mode', theme)`. It's a control surface for Bricks' native mechanism, not a separate one.

**Practical implication:** to make the rest of a page actually re-theme when this toggle is used, you don't write custom CSS/JS for the toggle itself — you make sure the design system's colors are dark-mode-enabled (`darkModeEnabled: true` with both `light` and `dark` values), via whatever color-palette ability the current MCP connection exposes (e.g. `bricks/create-color`/`bricks/update-color` on the native Bricks MCP, or `bricks-edit-color-palette-entry`'s `dark` parameter on Novamira). Once that's true, Bricks' own generated CSS already swaps automatically based on `[data-brx-theme]`, and this element just becomes the button that flips it. Adding this toggle to a page whose palette has no dark-mode-enabled colors will visibly change the attribute/localStorage but nothing will look different, because nothing on the page is listening for it yet — that's a design-system gap, not a toggle bug.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcolorschemetoggle.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xcolorschemetoggle` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## 2-state vs 3-state: it's entirely about `type`

`type` controls both the visual style and, critically, whether "System" is even an option:

| `type` | States | Notes |
|---|---|---|
| `single-button` | 2 (light/dark only) | Clicking cycles between the two. No system option exists for this type at all. |
| `switch` | 2 (light/dark only) | Same — toggle-switch visual, no system state. |
| `dropdown` | 2 or 3 | 3-state by default. Set `includeSystem: "false"` to drop to 2. |
| `buttons` (radio group) | 2 or 3 | Same as dropdown — `includeSystem` controls it. |

**For a 3-option (System/Light/Dark) toggle**, `type` must be `dropdown` or `buttons` — the other two types structurally can't show a third option, there's no override for them. `includeSystem` defaults to `"true"` (the schema placeholder), so:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcolorschemetoggle.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{ "name": "xcolorschemetoggle", "settings": { "type": "dropdown" } }
```

is already 3-state with no other settings needed. Renders a listbox with exactly `System` (selected by default) / `Light` / `Dark` options.

For a 2-option **dropdown or buttons** control (no system choice), set it explicitly:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcolorschemetoggle.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{ "name": "xcolorschemetoggle", "settings": { "type": "dropdown", "includeSystem": "false" } }
```

For the simplest 2-state control, skip `dropdown`/`buttons` entirely and use `single-button` or `switch` instead — cleaner than a dropdown forced into 2-state.

---

## "System" is a snapshot, not a live subscription

In the element's own JS (`colorschemetoggle.js`): selecting **System** does `localStorage.removeItem('brx_mode')`, reads `window.matchMedia('(prefers-color-scheme: dark)').matches` **once**, and sets `data-brx-theme` to whatever that resolves to at that moment. There is no `matchMedia(...).addEventListener('change', ...)` anywhere in the file.

**Consequence: if the visitor changes their OS-level light/dark setting while the page is already open, the page will not re-theme live.** It re-resolves only the next time the theme is set — on next page load (if "System" is still the stored/default state) or the next time a user interacts with the toggle. Don't tell a client "System mode follows OS changes in real time" — it follows OS preference at resolution time, not continuously.

---

## Field reference

All labels, icons, and most style controls are optional cosmetic overrides — the functional minimum is `type` (+ `includeSystem` if you want to override its default). Every field below is gated by a `required` condition keyed off `type` (and sometimes `includeSystem`/`iconStyle`/`animatedPreset`) — check the bundled schema (`bricksextras-element-schemas`, `elements/colorschemetoggle.json`) for the exact condition per field rather than assuming a control applies to every `type`.

| Setting | Purpose | Notes |
|---|---|---|
| `dropdownDisplay` | Trigger content for `dropdown` type: `both` (icon+text), `text-only`, `icon-only` | Dropdown only |
| `systemLabel`/`lightLabel`/`darkLabel` | Text for each option | `systemLabel` only applies where `includeSystem` is on |
| `systemIcon`/`lightIcon`/`darkIcon` | Icon per state | Standard Bricks icon object (`library`/`icon`/`svg`) |
| `iconStyle` | `icons` (your own custom icons) vs `animated-preset` (built-in morphing sun/moon) | `single-button`/`switch` only |
| `animatedPreset` | Which morphing icon animation: `css-rays`, `css-morph`, `lines` (SVG) | Only when `iconStyle != icons` |
| `enableTooltips` + `tooltipText`/`tooltipPosition` | Hover tooltip | `dropdown`/`single-button` only |
| `viewTransitionEffect` + duration/timing | Whole-page transition effect (fade/swipe/split) using the View Transitions API when the mode changes | Not visible in the builder preview — check on the live frontend |
| Dropdown-only style groups (`dropdownTrigger*`, `dropdown*`, `dropdownOption*`) | Full styling of the trigger button and the open dropdown panel | All gated to `type = dropdown` |

---

## Rendered DOM (for custom CSS/targeting) — real accessible markup per `type`, fully server-rendered

Unlike most JS-interactive elements in this plugin, all four `type`s render their complete, real markup server-side (including the dropdown panel's options) — JS only toggles state attributes/visibility, it doesn't build any of this from scratch. Each type uses proper ARIA widget roles:

- **`single-button`** — `<button role="switch" aria-checked="...">`, with the icon markup inside `.x-color-scheme-toggle__single-button-inner`. `iconStyle: animated-preset` (e.g. `css-rays`) renders a fixed set of empty `<span class="x-sun-moon-css-rays__part">` elements (9 for `css-rays`) — the morph animation is pure CSS keyframes/transforms on these parts, nothing to configure structurally.
- **`buttons`** — `<div role="radiogroup"><button role="radio" aria-checked="..." data-theme="light|dark|auto">` per option, with standard roving `tabindex` (only the checked/first button gets `tabindex="0"`, the rest `-1`).
- **`dropdown`** — trigger `<button aria-haspopup="listbox" aria-expanded="...">` plus a **fully server-rendered** `<div role="listbox" data-hidden="true">` containing one `<button role="option" aria-selected="...">` per choice — JS only flips `data-hidden`/`aria-expanded`, the options themselves are already in the initial HTML.
- **`switch`** — `<button role="switch" aria-checked="...">` wrapping `.x-color-scheme-toggle__switch-track` (the pill background) which itself wraps `.x-color-scheme-toggle__switch-thumb` (the moving circle) — `iconStyle: animated-preset` puts the same `x-sun-moon-*` markup as `single-button` inside the thumb specifically, not the track.

**The state attributes (`aria-checked`/`aria-selected`/`data-current-theme`) reflect the server's default assumption, not the visitor's actual stored preference** — a raw HTML fetch (`curl`, `get-page-elements`) always shows the unresolved default state; the real selected option is only correct after the client-side JS reads `localStorage.brx_mode`/`[data-brx-theme]` on load. Style the active/selected state via these attributes (`[aria-checked="true"]`, `[data-theme="dark"]`, etc.) rather than assuming a specific one is "the default" from a static read.

## Never do

- Don't expect `single-button` or `switch` to support a system/auto option — structurally 2-state only, no setting unlocks a third state for these types.
- Don't assume adding this element alone re-themes a page — it only flips the shared `data-brx-theme` attribute/`brx_mode` storage; the page's colors need `darkModeEnabled: true` light/dark pairs to actually respond.
- Don't claim "System" mode live-tracks OS-level changes while the page stays open — it's resolved once per theme-set, not subscribed to `matchMedia` change events.
- Don't hand-roll custom CSS keyed on a different attribute/class for dark styling on the same page — reuse `[data-brx-theme="dark"]`/the design system's dark-mode colors so this toggle and any other Bricks-native dark-mode UI stay in sync (they share the same storage key and event, `x-color-scheme-changed`, so multiple toggles on one page — e.g. header + footer — already stay synced with each other for free).
