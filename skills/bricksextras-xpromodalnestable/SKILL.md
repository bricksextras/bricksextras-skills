---
name: xpromodalnestable
description: "Use when building or debugging the Modal element (xpromodalnestable) from BricksExtras: a MicroModal.js-based dialog built directly from nested children, with a built-in multi-trigger system (page load, scroll, exit intent, element click/hover, etc.) instead of a single click-trigger field. Covers the triggers repeater's per-type fields, the inverted-value disableFocus gotcha, and the generic data-x-modal-close attribute mechanism."
---

**Requires:** BricksExtras 1.7.3+ with xpromodalnestable enabled

# BricksExtras: Modal (xpromodalnestable)

Shipped by the **BricksExtras** plugin, built on the third-party **MicroModal.js** library. A nestable element — its own children render directly inside `.x-modal_content`, no separate template needed (unlike the legacy `xpromodal`, which pulls content from a selected Bricks Template or a wysiwyg field). Always use this version for new work.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xpromodalnestable.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xpromodalnestable` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Built-in multi-trigger system — not a single `clickTrigger` field

Unlike `xoffcanvasnestable` (a single `clickTrigger` selector), this element has its own **`triggers` repeater** — multiple independent rows, each a different trigger *type*, and any number of rows can be added so a modal can open from several unrelated triggers at once (e.g. both an exit-intent trigger and a manual button click). Each `type` gates its own required sub-fields:

| `type` | Required sub-fields |
|---|---|
| `pageLoad` | `delay` (ms) |
| `pageLoadURLParameter` | `query_includes` (e.g. `referrer=facebook`), `delay` |
| `scroll` | `amount` (px scroll distance) |
| `scrolledToElement` | `selector`, `componentScope`, `delay` |
| `exitIntent` | none |
| `pageViews` | `views` |
| `timeInactive` | `inactive_time` (seconds) |
| `elementClick` | `selector`, `componentScope`, `aria_controls` (checkbox), `delay` |
| `elementHover` | `selector`, `componentScope`, `delay` |

For `scrolledToElement`/`elementClick`/`elementHover`, `selector` follows the same class-not-`_cssId` targeting rule as everywhere else in BricksExtras — give the real target element a `_cssClasses` value and point `selector` at it. `componentScope` (string `"true"`/`"false"`, not boolean) scopes that specific trigger row's lookup to the current component instance, same pattern as `xproslidercontrol`.

## `disableFocus`'s options are inverted relative to its own key name

`disableFocus`'s select options are `{"false": "Enable", "true": "Disable"}` — choosing **"Enable"** writes the literal string `"false"`, and choosing **"Disable"** writes `"true"`. This is consistent internally (`render()` computes the JS config's `disableFocus` as `'false' !== $value`, so `"false"` → JS `disableFocus: false` → auto-focus enabled, and `"true"` → JS `disableFocus: true` → auto-focus disabled) — but the setting key name and its own "Enable" option value point in opposite directions, so double-check which literal string is being written rather than assuming "Enable" means `true`.

## Any element can close the modal via `data-x-modal-close` — not just the built-in button

The built-in close button works by carrying a `data-x-modal-close` attribute (confirmed in `render()`, alongside its own `aria-label`/class). `maybe_remove_close` (checkbox) removes that built-in button entirely — but the *mechanism* isn't exclusive to it: the schema's own description on `maybe_remove_close` says "Elements with the `data-x-modal-close` attribute can close the modal." Add that same attribute (via `_attributes`) to any custom element nested inside the modal (a "Cancel" button, a link, etc.) to make it close the modal too, independent of whether the built-in close button is present or removed.

The backdrop uses the identical mechanism: whenever `clickBackdropClose` isn't set to `"none"` (default enabled), `render()` adds `data-x-modal-close` to `.x-modal_backdrop` itself — clicking outside the modal content closes it through the same generic attribute, not separate JS.

## Two distinct aria-label settings for two different targets

- **`aria_label`** (real PHP fallback: `"Close modal"` if unset) — the close button's own accessible name.
- **`maybeCustomAriaLabel`/`customAriaLabel`** — the modal *dialog container's* own aria-label. Only applied when `maybeCustomAriaLabel: "custom"`; otherwise the container gets no explicit aria-label at all (relying on `role="dialog"` plus whatever accessible-name the browser derives from content).

Don't conflate the two — setting one doesn't affect the other.

## Other settings worth knowing

- **`disableScroll`** (checkbox) locks page scroll while open, and also sets `data-lenis-prevent` on the root — relevant if the site uses the Lenis smooth-scroll library, same attribute pattern seen on `xoffcanvasnestable`.
- **`show_once`/`show_once_session`** are mutually exclusive checkboxes (each schema-gated to the other being off).
- **`preview_animation`** follows the general `builderPreview`/`*Preview` rule in `bricksextras-start-here` — builder-canvas preview only, never sent to the frontend. The real open/close transition is driven by `start_*`/`end_*` translate/scale controls plus `transitionDuration`, all written as CSS custom properties.
- **Query-loop instance uniqueness**: inside a running query loop, the config gets `isLooping`/`isLoopingComponent` written in automatically, same mechanism as `xoffcanvasnestable`/`xwsforms`.
- **`close_icon` has no runtime fallback** — PHP only renders an icon when the setting is explicitly present (`empty($settings['close_icon']) ? false : render_icon(...)`); omitting it produces a close button with an empty, invisible icon slot (`<span class="x-modal_close-icon"></span>`), not a missing button. Same no-default-at-render-time pattern as `xheadersearch`/`xbacktotop`/`xbeforeafterimage`. The schema shows a `default` of `{"library": "themify", "icon": "ti-close"}` — write that explicitly rather than relying on it.

## Rendered DOM (for custom CSS/targeting)

Captured live in a real browser (server HTML alone won't show this — see note below), one `pageLoad` trigger, open state:

```html
<div class="brxe-xpromodalnestable x-modal x-modal_open" data-x-id="{id}" data-x-modal="{...}"
     id="brxe-{id}" data-x-ready="true" aria-hidden="false">
  <div class="x-modal_backdrop" tabindex="-1" data-x-modal-close>
    <div class="x-modal_container" aria-modal="true" role="dialog" aria-label="Modal title">
      <div class="x-modal_content">
        <h3 class="brxe-heading">Modal title</h3>
        <div class="brxe-text"><p>Modal body content.</p></div>
        <button class="x-modal_close" aria-label="Close modal" data-x-modal-close>
          <span class="x-modal_close-icon"><i class="ti-close"></i></span>
        </button>
      </div>
    </div>
  </div>
</div>
```

Notes on this structure:

- **The backdrop, container, and close button are entirely auto-generated — none of this is something you build via `children`.** Your nested content only ever lands inside `.x-modal_content`, alongside the auto-inserted close button (unless `maybe_remove_close` is set). Don't try to hand-build a backdrop/container wrapper yourself; nest content directly on the element and the rest is injected.
- **`aria-label`/`aria-labelledby` on `.x-modal_container` is entirely client-JS-computed, never server-rendered** (checked the PHP — it only ever sets `aria-modal`/`role` there). The actual runtime logic (`promodal.js`, the `extrasModal` function): if `customAriaLabel` is set, use that verbatim; otherwise, if the modal contains no `h1`/`h2`/`h3`/`h4`/`h6` at all, `aria-label="modal"` (literal fallback string); otherwise take the *first* heading found — `aria-labelledby` pointing at that heading's `id` if it has one, else `aria-label` set to that heading's plain text. A raw HTML fetch (`get-page-elements`, a non-JS HTTP request) will never show any of this — only a real browser render will. Don't use its absence from a static check as evidence the modal has no accessible name, and don't assume which of the two attributes will be present without knowing whether the first heading has an `id`.
- `x-modal_open` (the open-state class) and `aria-hidden="false"` only appear once the trigger actually fires — this capture used `pageLoad` with a delay, so the closed/pre-trigger state looks different (no `x-modal_open`, `aria-hidden="true"`). Capture whichever state actually matters for the styling task at hand.
- Both the backdrop and the close button carry the generic `data-x-modal-close` attribute — confirms the "any element can close the modal via this attribute" mechanism from above is the same one driving the built-in pieces, not special-cased JS per element.

---

## Build workflow

1. Add one `triggers` repeater row per way the modal should open — most modals only need one, but multiple independent triggers are supported natively.
2. For `elementClick`/`elementHover`/`scrolledToElement` triggers, give the real target element an explicit class and point that trigger row's `selector` at it.
3. Nest real content directly as children.
4. If a custom in-modal element (not the default close button) should close it, add `data-x-modal-close` to that element via `_attributes`.
5. Double-check `disableFocus`'s value against what's actually wanted — its "Enable" option is the string `"false"`.
