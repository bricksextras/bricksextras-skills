---
name: xreadingprogressbar
description: "Use when adding a scroll-based reading progress bar (xreadingprogressbar) from BricksExtras: tracks scroll position against a target container and fills a bar accordingly. Covers that containerSelector/start/end are all safe to omit (real JS fallbacks, not just cosmetic placeholders), and that progressPosition is applied via a CSS substring match against the raw JSON config attribute rather than being read by the JS."
---

**Requires:** BricksExtras 1.7.3+ with xreadingprogressbar enabled

# BricksExtras: Reading Progress Bar (xreadingprogressbar)

Shipped by the **BricksExtras** plugin. A non-nestable element: a thin bar that fills as the visitor scrolls through a target container. The positioning mechanism in particular isn't derivable from the schema alone.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xreadingprogressbar.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xreadingprogressbar` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `containerSelector`, `start`, `end` are all safe to leave unset — real JS fallbacks, not just placeholder text

Unlike most BricksExtras defaults, these three genuinely fall back sensibly if omitted:

- `containerSelector` unset → tracks `document.body` (the whole page).
- `start` unset → `'top'` (progress begins once the container's top edge reaches the top of the viewport).
- `end` unset → `'bottom'` (progress reaches 100% once the container's bottom edge reaches the bottom of the viewport).

If `containerSelector` is set but resolves to nothing on the page, the script logs `"BricksExtras: Content not found on page. Check the content selector is correct."` to the console and does nothing further — a useful first check when the bar isn't moving at all.

## `progressPosition` isn't read by the JS — it's matched via a CSS substring selector against the raw config JSON

This is a real mechanism worth understanding, not a guess: `render()` writes the *entire* JS config object (including `position`) into a single `data-x-progress="{...}"` attribute. The JS itself never reads `config.position`. Instead, `readingprogressbar.css` matches against that same attribute using substring selectors:

```css
[data-x-progress*=positionTop] { top: 0; bottom: auto; position: fixed; }
[data-x-progress*=positionBottom] { top: auto; bottom: 0; position: fixed; }
```

Because the JSON blob literally contains the text `positionTop` or `positionBottom` when that's the selected value, these attribute-contains selectors match without any JS or dedicated control-level CSS mapping involved.

**`progressPosition: custom` matches neither rule** — choosing it means the element gets no `position: fixed`/`top`/`bottom` from the plugin's own CSS at all, leaving positioning entirely to whatever the page sets directly on the element (Bricks' own `_position`/`_top`/`_bottom` controls, or a global class). This is the correct way to place the bar somewhere other than a page-wide fixed top/bottom strip — e.g. relative to a specific section rather than the viewport.

## Other settings

- **`ariaLabel`** — accessibility label, falls back to "Reading progress bar" if unset (a real PHP-level fallback, not just a placeholder).
- **`progressHeight`** — maps to `height` on the root element itself (placeholder `4px`).
- **`background`** — root element's own background (the track).
- **`progressBackground`** / **`progressBorder`** — the moving fill bar (`.x-reading-progress-bar_progress`).

## Rendered DOM (for custom CSS/targeting)

```html
<div class="brxe-xreadingprogressbar x-reading-progress-bar" role="progressbar" aria-label="Reading progress bar" aria-valuemin="0" aria-valuenow="0" aria-valuemax="100" data-x-progress="{&quot;position&quot;:&quot;positionTop&quot;}">
  <div class="x-reading-progress-bar_progress"></div>
</div>
```

Just the root (the track — style via `background`) wrapping one fill div (`.x-reading-progress-bar_progress`, styled via `progressBackground`/`progressBorder`) — nothing else is generated. `aria-valuenow` is updated live by the JS as the visitor scrolls (`0` only in the initial server-rendered state); the fill bar's actual width is a JS-set inline style, not a CSS class swap.

## Build workflow

1. Decide what should be tracked — the whole page (leave `containerSelector` unset) or a specific container (set it to a real class on that container).
2. Set `start`/`end` only if the defaults (top-of-viewport / bottom-of-viewport) aren't what's wanted.
3. Use `progressPosition: positionTop` or `positionBottom` for the common fixed-strip placement; use `custom` plus the element's own position controls for anything else.
4. If the bar never moves, check the browser console for the "Content not found" message before assuming a config problem elsewhere.
