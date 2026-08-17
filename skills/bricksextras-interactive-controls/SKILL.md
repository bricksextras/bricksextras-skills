---
name: bricksextras-interactive-controls
description: "Use when adding scroll parallax, a floating/bobbing effect, a hover tilt effect, or a hover tooltip to a native Bricks core element (container, heading, button, image, etc.) via BricksExtras' generic 'Interactive' and 'Tooltip' style-tab controls (x_parallax, x_floating, x_tilt, x_tooltips/x_tooltip_content). These are NOT BricksExtras x*-prefixed elements — they're extra controls injected onto ~40 native Bricks elements. Covers which elements get them, each effect's data-attribute output, and the builder-preview limitation."
---

**Requires:** BricksExtras 1.7.3+, Bricks 2.4+

# BricksExtras: Generic Interactive/Tooltip controls on native Bricks elements

BricksExtras injects a "Interactive" style-tab control group (parallax, floating, tilt) and a separate "Tooltip" style-tab control group onto a fixed, hardcoded list of **native Bricks core elements** — not onto BricksExtras' own `x*` elements. Registered via the `bricks/elements/{element}/control_groups` and `bricks/elements/{element}/controls` filters, applied per-element-name to: `container`, `section`, `block`, `div`, `heading`, `text-basic`, `text`, `button`, `icon`, `image`, `video`, `divider`, `icon-box`, `social-icons`, `list`, `accordion`, `accordion-nested`, `tabs`, `tabs-nested`, `form`, `map`, `alert`, `animated-typing`, `countdown`, `counter`, `pricing-tables`, `progress-bar`, `pie-chart`, `team-members`, `testimonials`, `html`, `code`, `template`, `logo`, `facebook-page`, `image-gallery`, `audio`, `carousel`, `slider`, `slider-nested`, `svg`, `wordpress`, `posts`, `pagination`, `nav-menu`, `sidebar`, `search`, `shortcode`, and the `post-*` single-post elements.

**None of BricksExtras' own `x*` elements get these controls through this mechanism** — if a build needs a parallax/floating/tilt/tooltip effect on, say, an `xproaccordion`, wrap it in a plain `container`/`block`/`div` that carries the effect instead (or check that specific element's own skill for an equivalent built-in setting, if one exists).

All three effects (and the tooltip attribute) are written via a shared `render_attributes` filter that **only runs on the real frontend** (`bricks_is_frontend()`) — none of them render or preview inside the Bricks builder canvas at all (the control group's own `info` text says exactly this: "Interactions are not visible when editing in the builder"). Always verify these on the live frontend, never by inspecting the builder preview.

## `x_parallax` — Rellax.js scroll parallax

Checkbox. When enabled, writes `data-rellax-speed` (from `scrollSpeedDefault`) and `data-rellax-xs-speed` (from `scrollSpeedMobilePortrait`) onto the element — **both default to `'0'`** if left unset (a real PHP fallback, not a schema-only default). `scrollSpeedDesktop`/`scrollSpeedTablet`/`scrollSpeedMobile` (mapped to `data-rellax-desktop-speed`/`-tablet-speed`/`-mobile-speed`) have **no fallback at all** — omitting them means Rellax's own library default applies at that breakpoint rather than `0`. All five speed fields are numbers in the -10 to 10 range; the control's own description recommends staying within that range. Breakpoints referenced (992px / 768–991px / 479–767px / ≤478px) match Bricks' own standard breakpoint set.

## `x_floating` — CSS floating/bobbing animation

Checkbox. Writes `data-x-floating` — empty string for the default vertical direction, or the literal string `"horizontal"` when `x_floating_direction` is set to `horizontal`. `x_floating_duration`/`x_floating_delay` are **plain text fields taking a literal CSS time string with unit** (e.g. `"6000ms"`, `"0ms"`) — not number+unit controls — written directly as the `--x-floating-duration`/`--x-floating-delay` CSS custom properties. `x_floating_distance` is a real number+unit control (px, default `-20px`) mapped to `--x-floating-distance`. Requires `floating.css` (auto-enqueued when the checkbox is on).

## `x_tilt` — VanillaTilt.js hover tilt effect

Checkbox. Builds a single JSON config written to `data-x-tilt`: `{"config": {"max", "scale", "startX", "startY", "speed", "perspective", "glare"?, "max-glare"?}, "breakpoint"?}`. Each `config` key is only included if its corresponding setting (`x_tilt_max`/`x_tilt_scale`/`x_tilt_start_x`/`x_tilt_start_y`/`x_tilt_speed`/`x_tilt_perspective`) is explicitly set — no fallback values are written into the JSON for omitted ones, meaning VanillaTilt's own internal defaults apply instead (leave unset rather than guessing a "safe" value if the goal is library-default behavior). `x_tilt_glare` (checkbox) adds `glare: true` plus `max-glare` (from `x_tilt_max_glare`, defaulting to `'1'` only when the glare checkbox itself is on). `x_tilt_breakpoint` is written as a **top-level `breakpoint` key**, sibling to `config`, not nested inside it — matches the control's own "Disable" separator label, i.e. it's the viewport width below which the tilt effect turns off, not a tilt-effect parameter itself.

## `x_tooltips`/`x_tooltip_content` — generic per-element tooltip text

Checkbox (`x_tooltips`) + textarea (`x_tooltip_content`, dynamic-data capable via `bricks_render_dynamic_data()`). When both are set, writes `data-x-tooltip` as `{"content": "..."}` on the element. This is the actual mechanism behind `xpopover`'s `maybeDynamicContent` mode — see `bricksextras-xpopover` for how a popover element reads this attribute from matched target elements. `x_tooltip_content` alone with `x_tooltips` unset (or vice versa) writes nothing; both must be set together.

## Build workflow

1. Confirm the target element is actually in the native-element list above before assuming these controls exist on it — they're absent on all `x*` elements and on any native element not explicitly listed (e.g. not on `filter-*` or WooCommerce-specific elements).
2. Verify parallax/floating/tilt effects on the real frontend, never the builder preview — they're explicitly inert there.
3. For `x_floating_duration`/`x_floating_delay`, always include the unit in the string (`"6000ms"`) — these are plain text fields, not number+unit controls, so an omitted unit isn't auto-appended.
4. For a tooltip driven by `xpopover` in dynamic mode, set both `x_tooltips` and `x_tooltip_content` together on the target element — one without the other produces no `data-x-tooltip` attribute at all.
