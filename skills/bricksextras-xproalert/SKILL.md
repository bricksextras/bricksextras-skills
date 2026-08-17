---
name: xproalert
description: "Use when building or debugging the Pro Alert element (xproalert) from BricksExtras: a nestable dismissible alert/banner whose show-again behavior (always, once ever, until dismissed, after a delay, or manual-only) is tracked entirely client-side via localStorage, keyed per element instance. Covers the show_again type table, the manual JS API for triggering alerts from Bricks Interactions, and the fixed-position gotcha."
---

**Requires:** BricksExtras 1.7.3+ with xproalert enabled

# BricksExtras: Pro Alert (xproalert)

Shipped by the **BricksExtras** plugin. Nestable — or, by default, backed by a wysiwyg editor field instead of children; see `alert_content` below.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xproalert.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xproalert` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## The alert is always in the page HTML — visibility is 100% client-side

`render()` never conditionally omits the element or adds `display:none`; every visit downloads and initially paints the full alert. Whether it then gets hidden is entirely up to `proalert.js` toggling `element.style.display` after `DOMContentLoaded`, based on `localStorage`. If JS fails to load or is blocked, the alert **stays visible indefinitely** regardless of `show_again` — there's no server-side or CSS-only fallback.

## `show_again` — five distinct behaviors, all tracked via `localStorage`

Each instance is tracked under localStorage keys `x-{identifier}-last-shown-time` and `x-{identifier}-dismissed`, where `{identifier}` is this element's own unique instance identifier (the same one written as `data-x-id`) — so the tracking is **per element instance, global across the whole browser/domain**, not per page or per session.

| `show_again` value | Behavior |
|---|---|
| *(unset / empty string)* | Always shows, every page load — falls through to the switch's default case. |
| `page_load` | Same as unset — always shows every load. |
| `never` | Shows once, ever (checks `last-shown-time` — if it's ever been recorded, never shows again). |
| `dismiss` | Shows on every load **until** the user clicks the close button once — after that, never shows again. |
| `after` | Shows again once `show_again_days` + `show_again_hours` have elapsed since it was last actually shown. |
| `manual` | Never auto-shows on load at all — only appears when explicitly triggered via the JS API below. Pair this with the `typeInfo` control's own on-screen note pointing to using it with custom code/interactions. |

`show_again_days`/`show_again_hours` only apply (and are only schema-visible) when `show_again: "after"`.

## Triggering manually via `bricksextras.alert` — needed for `manual` mode, and useful generally

`window.bricksextras.alert.show({target})` / `.hide({target})` / `.toggle({target})` — each expects `target` to be the actual alert's DOM element (must carry `data-x-id`, which every instance has automatically). This is the mechanism to wire up via **Bricks Interactions** (e.g. a button's click interaction running "Show Alert" against this element) when `show_again: "manual"` — there is no other way to reveal a manual-mode alert.

The close button and `xShowAlert`/`xCloseAlert` also dispatch real custom DOM events on the alert root — `x_alert:close` and `x_alert:show` — usable as hooks for other JS without touching the plugin's own functions.

## `alert_position: "fixed"` — position controls only apply once `data-viewport` is present

`alertTop`/`alertRight`/`alertBottom`/`alertLeft`/`alertZindex` all target the CSS selector `&[data-viewport]`, and `render()` only adds the `data-viewport` attribute when `alert_position` is explicitly `"fixed"` — leaving `alert_position` unset (default `relative`/static) means none of those four position controls have anywhere to apply, even if values are set on them.

## `alert_content` — wysiwyg or nestable, mutually exclusive

- **`wysiwyg`** (default) — content comes from the `alert_wysiwyg` editor field, parsed through Bricks' own editor-content rendering. Real fallback text `"Edit me. I am an alert."` only applies in the builder UI (schema `default`, not render-time) — per the general "schema defaults are UI-only" rule, omitting `alert_wysiwyg` on a real build renders nothing there.
- **`nestable`** — real nested children render instead; `alert_wysiwyg` is skipped entirely regardless of whether it has a value.

## Other settings

- **`aria_label`** on the dismiss button has a genuine PHP fallback (`"Dismiss"`) if left unset — safe to omit.
- **`dismiss_icon`** — same `empty()`-gated icon rendering pattern as elsewhere; omit to render no icon.
- **`builderHidden`** only affects the builder canvas (`v-show` in the Vue template) — has zero effect on the frontend.

## Rendered DOM (for custom CSS/targeting)

Fully server-rendered, no JS dependency for the markup itself (only visibility is JS-driven — see above). `alert_position: "fixed"` with position controls set, `dismiss_icon` set:

```html
<div id="brxe-{id}" class="brxe-xproalert" data-x-alert="{&quot;show_again&quot;:{&quot;type&quot;:&quot;dismiss&quot;,&quot;options&quot;:{&quot;days&quot;:0,&quot;hours&quot;:0}}}" data-viewport data-x-id="{id}">
  <p>This is an alert message.</p>
  <button class="x-alert_close" aria-label="Dismiss">
    <span class="x-alert_close-icon"><i class="fas fa-times"></i></span>
  </button>
</div>
```

Notes:

- **Content (wysiwyg-parsed HTML, or real children in `nestable` mode) and the close button are siblings directly inside the root — there's no separate content wrapper div.** Custom CSS targeting "the alert's content area" as a whole should target `.brxe-xproalert` itself (minus `.x-alert_close`), not a nonexistent inner wrapper class.
- **`data-viewport` is a bare boolean attribute (no value)** — present only when `alert_position: "fixed"`, matching the `&[data-viewport]` CSS selector the four position controls target (see above).
- `data-x-alert` carries the full `show_again` config as JSON (type + days/hours options) — useful as a debugging/inspection hook, not something to hand-write.
- The close button's icon markup only appears if `dismiss_icon` is set — omitting it renders `<button class="x-alert_close" aria-label="Dismiss"><span class="x-alert_close-icon"> </span></button>` with an empty icon span, not a missing button.

## Build workflow

1. Choose `show_again` deliberately — most real banners want `dismiss` (persistent close) or `after` (periodic reminder), not the default always-show behavior.
2. For `manual` mode, wire a Bricks Interaction (or custom JS) calling `bricksextras.alert.show({target: <element>})` against this alert — nothing else will reveal it.
3. Only set `alert_position: "fixed"` plus the position controls together — the position controls do nothing without it.
4. Pick `alert_content` first (`wysiwyg` vs `nestable`) before writing content — the other content path is fully ignored.
