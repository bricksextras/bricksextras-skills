---
name: headerextras
description: "Use when enabling or debugging sticky or overlay header behavior from BricksExtras: the 'Header Extras' control group injected into page settings and header-template settings (not any single element's own settings). Covers the settings list, page/content-template/header-template precedence, and the data-attribute mechanism (data-x-sticky, data-x-sticky-active, data-x-overlay) that drives which header-row-level elements react to it."
---

**Requires:** BricksExtras 1.7.3+

# BricksExtras: Header Extras (sticky & overlay header settings)

Not an element — a **control group the plugin injects into page settings and template settings**, via the `builder/settings/page/controls_data` and `builder/settings/template/controls_data` filters. It's how sticky and overlay header behavior get switched on for a site; individual header-row-level elements (`xheaderrow`, `xnotificationbar`) then react to that state through shared data attributes, each with their own per-element visibility/style settings.

**Where it appears:** page settings (for the page's `content` area) and header-template settings. Explicitly excluded from footer and popup template settings.

**Before writing any of these settings, read `references/headerextras.json` in this skill directory now** — the same discipline as reading an element's schema JSON before configuring it. Unlike elements, there is no live ability that can substitute for this: `bricks/get-page-settings` / `bricks/get-template-settings` (and their `set-` counterparts) only read/write the stored values as an opaque object, with no field list, types, or defaults returned; `bricks/list-settings-schema` doesn't cover this group either — it only lists Bricks core's own global settings (Settings > general/performance/WooCommerce/etc.), not settings groups plugins inject into page/template settings via a filter, like this one. So `references/headerextras.json` — hand-derived from the plugin's own header-extras registration code — is the only schema source for this settings group, live or bundled, and there's no fallback ability to check it against if the plugin version drifts. If BricksExtras is updated past `1.7.3`, treat this file as possibly stale and re-derive it from that registration code directly rather than trusting it blindly.

## Settings (all under a "Header Extras" group)

| Setting | Purpose |
|---|---|
| `xOverlayHeader` | Breakpoint at/above which the header becomes an overlay (transparent, absolutely positioned over the page's first section) — `none`, `always`, or a specific registered breakpoint width |
| `xHeaderZindex` | Base z-index for `#brx-header` |
| `xStickyHeaderScroll` | Enable/disable sticky header behavior entirely (`true`/`false`) — the master switch everything else in this group depends on |
| `xStickyHeaderAbove` | Breakpoint at/above which sticky applies. **No schema default — unset behaves as `none`, which permanently disables sticky at every viewport width regardless of `xStickyHeaderScroll`.** Must be explicitly set to `always` (sticky at all widths) or a specific breakpoint width to actually activate. See the dedicated note below. |
| `xStickyHeaderScrollDistance` | Pixels scrolled down before the header becomes sticky |
| `xStickyHeaderZindex` | z-index once stuck (default effectively 999) |
| `xStickyHeaderFadeDuration` | Transition duration for the header itself fading in as sticky |
| `xStickyHeaderTransitionDuration` | Transition duration for individual header rows animating into their sticky styles |
| `xStickyHeaderHide` | A **second, independent** behavior: hide the sticky header entirely after scrolling further (`true`/`false`) |
| `xStickyHeaderHideEffect` | `fade` or `slideUp`, only relevant when `xStickyHeaderHide` is on |
| `xStickyHeaderHideDistance` / `xStickyHeaderHideTollerance` | Scroll distance and tolerance (px) for the hide-after-further-scrolling behavior |

`xStickyHeaderHide` is easy to mistake for a duplicate of `xStickyHeaderScroll` — it isn't. `xStickyHeaderScroll` makes the header stick in place at all; `xStickyHeaderHide` is a layered-on behavior that then hides that already-stuck header once the visitor scrolls past a second, further distance (e.g. a nav that sticks immediately, then disappears entirely if they keep scrolling down, reappearing on scroll-up — this uses the bundled Headroom.js library).

### `xStickyHeaderScroll: true` alone does nothing — `xStickyHeaderAbove` is a required companion setting, not an optional refinement

This is the single most common way sticky "silently doesn't work" despite every other setting (including `xStickyHeaderScrollDistance`) being correctly configured and persisted.

`xStickyHeaderAbove` has no `default` in its control schema. When unset, the plugin's server-side header-attribute output writes that straight into the header's `data-x-break` attribute as the literal string `"none"`. The frontend `header.js` reads that attribute as `mediaWidth` and gates all sticky activation on `window.innerWidth >= mediaWidth` — comparing a real number against the string `"none"` coerces to `NaN`, and any comparison against `NaN` is always `false`. So the scroll listener that would ever add the sticky-active class is never attached, at any viewport width, no matter how far the visitor scrolls or what `xStickyHeaderScrollDistance` says.

**Practical rule: whenever you set `xStickyHeaderScroll: true`, set `xStickyHeaderAbove` in the same update.** Its real values are `always` (sticky at every viewport width — the common case for "just make it sticky") or a specific Bricks breakpoint width (sticky only above that width; the UI label is `"{label} ( > {width+1}px )"`, and `always` internally becomes effectively `> 1px` via the same code path, not a special-cased bypass).

`xOverlayHeader` takes the same options list (`none`/`always`/breakpoint width) and has the identical unset-means-inert behavior, but the mechanism is different: it gates a server-rendered `@media (min-width: Xpx)` CSS block, not a JS runtime comparison. The failure mode (nothing happens) looks the same from the outside, but if you're diagnosing overlay-not-appearing, check the generated inline CSS for a `@media` wrapper rather than the `data-x-break` attribute — that one's for sticky specifically (`data-x-overlay` is the separate attribute overlay actually uses at runtime for the data-attribute side of things).

## Setting precedence

`HeaderExtras::get_current_template_setting()` resolves each key in this order, first match wins:

1. **Page settings** (the specific page/post being viewed)
2. **Content template settings** (the template assigned to the page's content area, if not itself a page-level override)
3. **Header template settings** (the header template's own settings)

A page-level override always wins over whatever the header template itself specifies, and the content template overrides the header template. Set site-wide defaults on the header template, and only override on page settings for exceptions.

## The mechanism: data attributes, not classes alone

This is what actually connects Header Extras' page/template-level switches to individual header-row elements:

- **`#brx-header`** gets class `x-header_sticky` whenever `xStickyHeaderScroll` is enabled, and class `x-header_sticky-active` added/removed by JS (`header.js`) as the visitor crosses `xStickyHeaderScrollDistance`.
- **Any header child carrying a `data-x-sticky` attribute** (its own per-element "sticky display" setting — `xheaderrow`'s `stickyDisplay` is one example, `xnotificationbar` has its own equivalent) is a participant in this system. When the header becomes active, JS selects every such child *except* those marked `data-x-sticky="hide"` and sets `data-x-sticky-active="true"` on them; that attribute is removed again when the header goes inactive.
- **`data-x-overlay`** works the same way for overlay mode, set per-child from that element's own overlay-display setting, read against `#brx-header`'s overlay/scrolling state by the plugin's default CSS.
- **`xnotificationbar` gets extra, dedicated handling beyond the shared `data-x-sticky` mechanism when it's placed inside the header template** — `header.js` specifically looks it up and factors its height into the header's total height for the `xStickyHeaderScrollDistance` calculation, live-recalculating when the bar is dismissed (see `bricksextras-xnotificationbar`'s "Rendered DOM" section for the mechanism). This only applies when the bar is actually inside the header template.

**Practical implication:** Header Extras alone does nothing visually to a specific row/child — it only puts the header itself into the right state (classes, CSS variables, `data-x-overlay` breakpoint attribute). Every visible effect (a row hiding, a row appearing only once stuck, a background color while overlaid) is the *child element's own* settings reacting to that state. Don't expect enabling `xStickyHeaderScroll` alone to change how any specific row looks — check that row's own element skill (e.g. `bricksextras-xheaderrow`) for what it does with the resulting state.

## Never do

- Don't look for these settings on `xheaderrow` or any other single element — they're page/template settings, injected by a filter, not part of any element's own control schema.
- Don't set `xStickyHeaderHide` expecting it to be how you enable sticky at all — `xStickyHeaderScroll` is the master switch; `xStickyHeaderHide` only controls the secondary hide-after-further-scroll behavior on top of an already-sticky header.
- Don't assume a header template's own settings will apply if the page (or its assigned content template) also defines the same key — page settings win, then content template, then header template.
- Don't expect visual changes from Header Extras settings alone — they set header-level state; the actual per-row visibility/styling comes from each header child's own settings reacting to that state (see `bricksextras-xheaderrow`).
- Don't set `xStickyHeaderScroll: true` without also setting `xStickyHeaderAbove` — leaving it unset is silently equivalent to `none`, which permanently blocks sticky activation at every viewport width via a `NaN` comparison in `header.js`, even though every other setting (including scroll distance) is correct and saved. Set it to `always` unless a specific breakpoint cutoff is actually wanted.
