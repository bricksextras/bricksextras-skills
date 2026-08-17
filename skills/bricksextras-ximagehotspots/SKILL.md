---
name: ximagehotspots
description: "Use when building or debugging the Image Hotspots element (ximagehotspots) from BricksExtras, or its Image Hotspot Marker child (ximagehotspotmarker): clickable/hoverable numbered pins positioned over an image, each opening a popover. Covers the two population modes (built-in markers repeater vs nestable marker elements), how each supports a query loop, and per-marker fields with no runtime fallback."
---

**Requires:** BricksExtras 1.7.3+ with ximagehotspots element enabled

# BricksExtras: Image Hotspots (ximagehotspots) + Image Hotspot Marker (ximagehotspotmarker)

Shipped by the **BricksExtras** plugin. `ximagehotspots` renders a base image with positioned marker buttons that reveal a popover (via Tippy.js) on click or hover. Nestable, but — unusually — nestability is conditional on its own `type` setting, not unconditional like most nestable elements.

**Before building from any template below, read both elements' schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/ximagehotspots.json` and `references/elements/ximagehotspotmarker.json` (for the marker child) inside the `bricksextras-element-schemas` skill directory and read them. If either file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for that element instead. The templates and examples below show documented required structure and common patterns only — the schema files (or live calls) are the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened these sources this session.

---

## Two population modes, switched by `type`

| `type` | Markers come from | Popover content |
|---|---|---|
| `hotspots` (default) | The `markers` repeater setting, directly on `ximagehotspots` | Plain text/rich-text only (`content`, an editor field) |
| `nestable` | Nested `ximagehotspotmarker` child elements | Arbitrary nested elements — anything can go inside the marker's own popover, not just text |

**`nestable` is the newer, more flexible mode** — the `markers` repeater (built-in mode) predates the dedicated marker element, and only supports plain popover content. Prefer `nestable` unless there's a specific reason to use the simpler repeater (e.g. very simple text-only popovers where nesting elements would be overkill).

Style/behavior settings (`icon`, `markerBackgroundColor`, `placement`, `interaction`, pulse animation, popover transition, etc.) live on `ximagehotspots` itself and act as the default for both modes. In `nestable` mode, `ximagehotspotmarker` repeats nearly the same field set — any of those a marker sets itself **override** the parent's default for that one marker; leaving them blank inherits from the parent (explicit in the marker's own schema: "Leave blank if wanting to inherit from parent Image Hotspots settings").

## Positioning

`position_x`/`position_y` are plain text fields, relative to the image — not a coordinate-picker control. They accept either a percentage string (e.g. `"30%"`) or a bare number (e.g. `30`, interpreted as a percentage); no need to append `%` when the source field is already numeric. Both modes use the same format, whether static or dynamic-tag-driven.

**`position_x` also accepts a combined `"x,y"` string sourced from a field-mapping plugin** — e.g. an ACF field type that stores hotspot coordinates as a single paired value — instead of two separate `position_x`/`position_y` dynamic tags. When the data source already produces one combined coordinate field, point `position_x` at it directly rather than trying to split it into two tags first; when position data comes from two separate fields (the more common case), keep using `position_x`/`position_y` as documented above.

## Built-in mode (`type: "hotspots"`) — manual

**This JSON is an example, not the schema.** It shows the required skeleton, but individual setting names/values may be incomplete or stale relative to the current element version. Before building from this, read `references/elements/ximagehotspots.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "ximagehotspots",
  "settings": {
    "image": { "id": 123 },
    "interaction": "mouseenter focus",
    "markers": [
      { "title": "1", "position_x": "22%", "position_y": "35%", "content": "Mountain" },
      { "title": "2", "position_x": "68%", "position_y": "15%", "content": "Sky" }
    ]
  }
}
```

`interaction` (a parent-level setting, not per-marker in this mode) defaults to `click` — set it to `mouseenter focus` for hover-triggered popovers.

## Built-in mode with a query loop — one marker per loop item

`hasLoop`/`query` go directly on `ximagehotspots` itself in this mode (it's one of the few BricksExtras elements where the loop lives on the element, not a wrapping child — same pattern as `xdynamicchart`/`xdynamictable`). The `markers` repeater's `checkLoop: true` flag marks it as loop-aware: write **one templated marker row** whose fields are dynamic tags, and it gets re-evaluated once per loop item — not repeated verbatim per query result.

**This JSON is an example, not the schema.** Check `references/elements/ximagehotspots.json` (or the live schema ability) before building from it — field names like `checkLoop` may be incomplete or stale relative to the current element version.

```json
{
  "name": "ximagehotspots",
  "settings": {
    "image": { "id": 123 },
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": "post", "posts_per_page": 10 },
    "markers": [
      { "title": "{post_title}", "position_x": "{cf_x_position}", "position_y": "{cf_y_position}", "content": "{post_excerpt}" }
    ]
  }
}
```

Real-world use of this needs `position_x`/`position_y` to come from **per-post custom fields** — each post/CPT row storing its own X/Y percentage — so every marker lands somewhere different. Using the same literal string for every loop iteration stacks all markers identically; dynamic tags re-evaluate correctly per item, so pointing `position_x`/`position_y` at real per-post fields produces distinct positions.

## Nestable mode with a query loop — loop lives on a wrapping block, not the marker

`ximagehotspotmarker` has **no `hasLoop`/`query` of its own** in its schema — unlike the parent element's built-in mode, the loop can't live directly on the marker. Instead, wrap it in a `block` (or `div`/`container`) carrying the loop, same general pattern as `xproslider` slides or `xcontenttimeline` horizontal mode:

**This JSON is an example, not the schema.** Check `references/elements/ximagehotspots.json` and `references/elements/ximagehotspotmarker.json` (or the live schema ability) before building from it — do not copy settings out of this block without confirming they still exist and mean what's shown here.

```json
{
  "name": "ximagehotspots",
  "settings": { "image": { "id": 123 }, "type": "nestable", "interaction": "mouseenter focus" },
  "children": [
    {
      "name": "block",
      "settings": {
        "hasLoop": true,
        "query": { "objectType": "post", "post_type": "post", "posts_per_page": 10 }
      },
      "children": [
        {
          "name": "ximagehotspotmarker",
          "settings": {
            "title": "{post_title}",
            "position_x": "{cf_x_position}",
            "position_y": "{cf_y_position}",
            "content": "{post_title}"
          }
        }
      ]
    }
  ]
}
```

A 4-post loop produces 4 distinct `.x-marker_marker` elements, each with its own dynamically-resolved title — per-iteration marker generation works the same way as the built-in-mode loop. Static positions stack markers visually identically; real coordinates need distinct per-post fields.

## Fields with no runtime fallback — same class of gotcha as icons elsewhere

`aria_label` (on both the built-in `markers` repeater rows and `ximagehotspotmarker`) falls back to `"Toggle popover"` when unset — but write it explicitly on every marker rather than relying on that fallback, per the general schema-defaults-are-UI-only rule. `search_icon`/`icon`-type fields elsewhere in BricksExtras have the same no-runtime-default behavior; this element's icon field (`icon`, default `ion-ios-pin`/`ionicons`) should be assumed to need the same explicit treatment unless verified otherwise.

## Rendered DOM (for custom CSS/targeting)

Both modes render `.x-hotspots_image` (a real responsive `<img>` with `srcset`/`sizes`) followed by the marker(s). **`hotspots` mode's markers are direct children of the root; `nestable` mode wraps its markers in an extra `.x-image-hotspots_inner` div** — a real structural difference between the two modes beyond just where the marker data comes from:

```html
<!-- hotspots mode -->
<div class="brxe-ximagehotspots" data-x-hotspots="{...}">
  <img class="x-hotspots_image" ...>
  <div class="x-marker" data-x-id="{id}_0">
    <button class="x-marker_marker x-marker_marker-trigger" style="left: 22%; top: 35%;" aria-label="..." aria-expanded="false">
      <span class="x-marker_marker-inner"><span class="x-marker_marker-title">1</span></span>
    </button>
    <div class="x-marker_popover"></div>
  </div>
</div>

<!-- nestable mode -->
<div class="brxe-ximagehotspots" data-x-hotspots="{...}">
  <img class="x-hotspots_image" ...>
  <div class="x-image-hotspots_inner">
    <div class="brxe-ximagehotspotmarker x-marker" data-x-id="{id}" data-x-hotspots-marker="[]">
      <button class="x-marker_marker x-marker_marker-trigger" style="left: 30%; top: 40%;" aria-label="..." aria-expanded="false">
        <span class="x-marker_marker-inner"><span class="x-marker_marker-title">1</span></span>
      </button>
      <div class="x-marker_popover"></div>
    </div>
  </div>
</div>
```

**`.x-marker_popover` is always empty in the server-rendered HTML, in both modes — the actual popover is real Tippy.js, built and portalled to `<body>` at interaction time**, same `data-theme="extras"`/`data-animation="extras"` pattern used by `xpopover`/`xmediacontrol`/`xfavorite`. Content (plain text in `hotspots` mode, real nested elements in `nestable` mode) ends up wrapped in `.x-marker_popover-content` inside the portalled `.tippy-box`:

```html
<div data-tippy-root>
  <div class="tippy-box" data-theme="extras" data-animation="extras" role="tooltip">
    <div class="tippy-content">
      <div class="x-marker_popover-content">
        <!-- hotspots mode: the plain-text/rich-text `content` field -->
        <!-- nestable mode: the marker's real nested children, e.g. <h3 class="brxe-heading">...</h3> -->
      </div>
    </div>
    <div class="tippy-arrow"></div>
  </div>
</div>
```

Don't expect popover content in a raw HTML/settings read — `.x-marker_popover`'s emptiness there is expected, not a sign anything is misconfigured; verify popover content by actually triggering the interaction in a browser.

## Never do

- Don't use the `markers` repeater in `nestable` mode, or nested `ximagehotspotmarker` children in `hotspots` mode — the two population modes are mutually exclusive, gated by `type`.
- Don't put `hasLoop`/`query` on `ximagehotspotmarker` directly — it has no such setting; wrap it in a looped block instead.
- Don't expect a coordinate-picker UI for `position_x`/`position_y` — it's a plain text field, though it accepts either a percentage string (`"30%"`) or a bare number (`30`).
- Don't assume identical/static `position_x`/`position_y` values across a query loop will spread markers out — every loop iteration using the same literal position stacks them exactly on top of each other. Use distinct per-item custom fields.
