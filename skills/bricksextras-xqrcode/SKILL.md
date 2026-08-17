---
name: xqrcode
description: "Use when adding a QR code (xqrcode) from BricksExtras: a canvas/SVG QR code generator with dot/corner styling. Covers that the data field has no fallback and must be set, and the mode/error-correction options."
---

**Requires:** BricksExtras 1.7.3+ with xqrcode enabled

# BricksExtras: QR Code (xqrcode)

Shipped by the **BricksExtras** plugin. A non-nestable QR code generator (canvas or SVG output) with per-region styling (dots, corner squares, corner dots).

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xqrcode.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xqrcode` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `data` has no fallback at all — not even a UI-only default

Unlike most controls covered by `bricksextras-start-here`'s "schema defaults are UI-only" rule, `data` (the actual encoded content) has no `default` key — only a `placeholder` illustrating the expected format (the site's own home URL, shown as an example). There is nothing to fall back to: omit it and the QR code encodes nothing. Always set it explicitly to whatever URL/text the code should actually encode — dynamic tags work here too (e.g. `{post_url}`).

## Other settings

- **`shape`**: `square`/`circle` — the overall code's outer shape.
- **`outputType`**: `canvas`/`svg` (default `canvas`) — SVG is the better choice for lossless scaling (a raster canvas gets blurry when scaled up). **Not meaningfully more CSS-stylable than canvas, despite being real markup** — see Rendered DOM below.
- **`errorCorrectionLevel`**: `L`/`M`/`Q`/`H` (default `M`, ~15%) — higher levels tolerate more damage/overlay (e.g. a logo in the center) at the cost of a denser pattern.
- **`mode`**: `Numeric`/`Alphanumeric`/`Byte`/`Kanji` (default `Byte`) — must match the actual character set of `data`; `Byte` is the safe default for arbitrary text/URLs.
- **Dots**: `dotsColor`, `dotsType` (`square`/`rounded`/`dots`/`classy`/`classy-rounded`/`extra-rounded`).
- **Corner squares/dots**: `cornersSquareColor`/`cornersSquareType`, `cornersDotColor`/`cornersDotType` — same type vocabulary as dots, styled independently from the body pattern.

## Rendered DOM: style exclusively through this element's own controls, in both output modes

**`canvas` mode** — same situation as `xdynamicchart`: the whole code is pixels drawn onto an inert `<canvas>`, nothing inside to target with CSS.

```html
<div class="brxe-xqrcode" data-x-id="{id}" data-x-qr-code="{...}"><canvas width="600" height="600"></canvas></div>
```

**`svg` mode is real markup, but not a meaningfully bigger CSS surface.** The library (`qr-code-styling`) builds one `<clipPath>` per visual region (dots, corners) out of dozens/hundreds of individual `<rect>`/`<path>`/`<circle>` primitives — one shape per QR module — then paints each region with a single large background `<rect>` clipped to that path, colored via an inline `fill="{hex}"` attribute:

```html
<svg width="300" height="300" viewBox="0 0 300 300">
  <defs>
    <clipPath id="clip-path-background-color-1"><rect .../></clipPath>
    <clipPath id="clip-path-dot-color-1"><path .../><rect .../>...<!-- dozens of module shapes --></clipPath>
  </defs>
  <rect clip-path="url('#clip-path-background-color-1')" fill="transparent"></rect>
  <rect clip-path="url('#clip-path-dot-color-1')" fill="#ff0000"></rect>
</svg>
```

There are no per-dot classes or ids to hook into, and color comes from an inline `fill` attribute (which a CSS `fill` rule can override, but only for the whole dot-color region at once — the same single value `dotsColor` already controls). **SVG mode's real advantage is lossless scaling, not CSS styling access** — use this element's own dot/corner color and type controls either way; there's no finer-grained customization available by inspecting the SVG structure.

## Build workflow

1. Always set `data` explicitly — there's no fallback.
2. Set `mode` to match the actual content type if it's not general text/URL (`Byte`).
3. Style dots/corners as needed; these are purely cosmetic and safe to omit if default square styling is fine.
