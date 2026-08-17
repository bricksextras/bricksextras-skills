---
name: xheaderrow
description: "Use when building or debugging the Header Row element (xheaderrow) from BricksExtras: a styleable row wrapper for building multi-row headers, with per-row visibility and styling for sticky and overlay header states. Covers the exact hide/show/always semantics per state and the data attributes that drive them. See bricksextras-headerextras for the page/template settings that switch sticky/overlay behavior on in the first place."
---

**Requires:** BricksExtras 1.7.3+ with xheaderrow element enabled

# BricksExtras: Header Row (xheaderrow)

Shipped by the **BricksExtras** plugin. Nestable (default child: `container`). **Only meaningful inside a header template** — it's a structural row wrapper for building multi-row headers (e.g. a top-bar row stacked above a main-nav row), not a general-purpose section for regular pages. Every row in a header can independently control its own visibility and styling for the sticky and overlay states described below.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xheaderrow.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xheaderrow` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## This element doesn't switch sticky/overlay behavior on

`xheaderrow` has no "enable sticky" or "enable overlay" toggle of its own — that's a separate, page/template-level settings group. **See `bricksextras-headerextras` first** for `xStickyHeaderScroll`, `xOverlayHeader`, and the rest of that settings group, plus the settings-precedence chain (page > content template > header template) and the shared `data-x-sticky`/`data-x-sticky-active`/`data-x-overlay` attribute mechanism this element plugs into.

Everything below assumes Header Extras' relevant master switch is already on somewhere in that precedence chain — without that, `stickyDisplay`/`overlayDisplay`/the sticky style fields have nothing to react to.

## `stickyDisplay` (`hide` / `show` / `always`, default `always`)

From the plugin's own JS/CSS (`header.js`, `stickyheader.css`):

- **`always`**: row is visible both at the top of the page and once stuck. While stuck, it also receives `data-x-sticky-active="true"` (set by `header.js` on every row *not* marked `hide`), which is what makes the `stickyBackground`/`stickyHeight`/`stickyShadow`/`stickyTypography` overrides (all targeting `&[data-x-sticky-active*=true]`) actually apply.
- **`hide`**: row is visible at the top of the page, but `display: none !important` once the header becomes sticky-active (`#brx-header.x-header_sticky-active [data-x-sticky=hide]`). Use for content that should disappear once scrolled (e.g. a promo bar).
- **`show`**: the inverse — row is `display: none` at the top of the page, and only appears (`display: flex`) once the header is sticky-active. Use for content that should *only* exist in the stuck state (e.g. a compact nav that replaces a taller logo row).

The row's own `data-x-sticky` attribute (holding this row's `stickyDisplay` value) is what `header.js` reads to decide which rows get promoted to `data-x-sticky-active` on scroll — don't confuse it with `data-x-sticky-active` itself, which is the runtime true/false state, added/removed by JS as the user scrolls past the scroll-distance setting from Header Extras.

## `overlayDisplay` (`hide` / `show` / `always`, default `always`)

Same three-value shape, but scoped to the overlay (transparent, positioned-over-content) state rather than the sticky state:

- **`hide`**: row hidden while the header is in overlay mode (not scrolled/not sticky-active), visible otherwise.
- **`show`**: forced visible specifically during overlay mode.
- **`always`**: no forced hide/show — falls through to normal visibility, with its background driven by `overlayBackground` (see below) while overlay is active.

## `overlayBackground`

Sets `--x-overlay-header-background` on the row. The plugin's own default CSS (`#brx-header:not(.scrolling):not(.x-header_sticky-active) > .brxe-xheaderrow:not([data-x-sticky-active*=true]) { background: var(--x-overlay-header-background) !important; }`) is what actually applies it — **only while the header is genuinely in its overlay state for the page's current viewport** (i.e. only if Header Extras' `xOverlayHeader` breakpoint condition matches). Since `xOverlayHeader` can be scoped to a specific breakpoint (e.g. desktop only), the same row can show this background on desktop and fall back to its normal `headerBackground` on mobile, with no additional setting needed on `xheaderrow` itself.

## Rendered DOM (for custom CSS/targeting)

```html
<div class="brxe-xheaderrow" data-x-overlay="always" data-x-sticky="hide"></div>
```

`data-x-sticky`/`data-x-overlay` (this row's own `stickyDisplay`/`overlayDisplay` values) are always present on the root — that's static, server-rendered config, not a runtime state. `data-x-sticky-active` is the one truly dynamic attribute: `header.js` adds/removes it only on rows found via `headerTemplate.querySelectorAll('[data-x-sticky]:not([data-x-sticky=hide])')` — **scoped to the actual header template element specifically, not any `xheaderrow` anywhere on the page.** A row outside the real header template's DOM (e.g. accidentally nested elsewhere) never receives `data-x-sticky-active`, no matter what its `stickyDisplay` is set to.

## Never do

- Don't look for a sticky/overlay on-off switch on `xheaderrow` — see `bricksextras-headerextras`. This element only controls per-row visibility/styling once something else has turned the behavior on.
- Don't confuse `stickyDisplay`'s `show` with `always` — `show` means the row is normally hidden and *only* appears once stuck; `always` means visible in both states. Picking the wrong one either permanently hides content or shows a row twice (e.g. duplicate logos) that was meant to swap.
- Don't assume `overlayBackground` applies unconditionally — it's gated on the header actually being in overlay mode for the current breakpoint, which Header Extras' `xOverlayHeader` can restrict (e.g. desktop-only overlay).
