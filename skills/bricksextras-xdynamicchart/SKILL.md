---
name: xdynamicchart
description: "Use when building or debugging the Dynamic Chart element (xdynamicchart) from BricksExtras: Chart.js-based bar/line/doughnut charts fed by manual data, a query loop (one dataset or one datapoint per item), or a flat JSON array via {query_array}. Covers the three query-loop modes, dataset coloring, and which controls apply per chart type."
---

**Requires:** BricksExtras 1.7.3+ with xdynamicchart element enabled

# BricksExtras: Dynamic Chart (xdynamicchart)

Shipped by the **BricksExtras** plugin. Renders a Chart.js `bar`, `line`, or `doughnut`/pie chart. **Not nestable** — like `xcalendar`, the query loop (if used) lives directly on the chart element's own settings (`hasLoop`/`query`), not on a child block. There's no per-datapoint child element.

The chart-type/mode gating facts below (which controls apply per `chart_type`, which fields per `queryLoopDatapoints` mode) come from the schema's `required` conditions and `controlGroups`.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xdynamicchart.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xdynamicchart` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Three completely different ways to feed it data

Controlled by `queryLoopDatapoints` (only relevant when `hasLoop` is on) plus the manual/no-loop default. These are mutually exclusive data models, not layered options — picking one hides the settings for the other two.

| Mode | `hasLoop` | `queryLoopDatapoints` | Data comes from |
|---|---|---|---|
| Manual | off | — | `content_items` repeater, hand-entered |
| Datasets | on | `dataSets` (default) | One query item = one dataset/series. Categories still come from `content_items`, but each item's `data` field is a dynamic tag evaluated **per loop iteration** |
| Data points | on | `dataPoints` | One query item = one point on a single series |
| JSON array | on | `array` | A flat JSON array (Bricks' native `query_array` loop), single- or multi-dataset depending on whether a `dataset` key is present per row |

### Manual mode

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamicchart.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamicchart",
  "settings": {
    "chart_type": "bar",
    "content_items": [
      { "title": "January", "data": "20", "color": { "hex": "#1da69a" } },
      { "title": "February", "data": "70", "color": { "hex": "#072027" } }
    ]
  }
}
```

Per-item `color` only applies with a **single** dataset (i.e. manual mode, or loop modes that resolve to one series) — see Dataset coloring below.

### Datasets mode — one series per query item

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamicchart.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamicchart",
  "settings": {
    "chart_type": "line",
    "hasLoop": true,
    "queryLoopDatapoints": "dataSets",
    "query": { "objectType": "post", "post_type": "post", "posts_per_page": 5 },
    "content_items": [
      { "title": "Jan", "data": "{cf_jan_value}" },
      { "title": "Feb", "data": "{cf_feb_value}" }
    ],
    "datasetLabel": "{post_title}",
    "legendDisplay": "true"
  }
}
```

`content_items` is **not hidden** in this mode (only `dataPoints`/`array` hide it) — it defines the shared x-axis categories, and each item's `data` field is a dynamic tag that gets re-evaluated inside every loop iteration, producing one value per category per looped post. `datasetLabel` (usually `{post_title}` or similar) names each resulting series in the legend. Looping 5 posts with `content_items` mapped to per-post custom fields produces 5 distinct series, correctly labeled from the loop context — not 5 flattened points on one line.

### Data points mode — one point per query item

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamicchart.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamicchart",
  "settings": {
    "chart_type": "line",
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": "post", "posts_per_page": 10 },
    "queryLoopDatapoints": "dataPoints",
    "queryLoopDatapoints_Label": "{post_title}",
    "queryLoopDatapoints_Value": "{post_id}"
  }
}
```

Simplest loop mode: each query item becomes one x-axis label/value pair on a single series. Good for "value per post/term/user" charts. This mode produces exactly one dataset — see Dataset coloring below, since `content_items`-style per-point coloring doesn't apply here.

### JSON array mode — flat `query_array` data

Uses Bricks' native `array` query type (`objectType: "array"`, `arrayEditor` holding a literal JSON array or a dynamic tag resolving to one) plus three chart-specific tag-mapping fields. **The tag syntax for this element is `{query_array @key:'...'}` — no `:raw` modifier.** The general Bricks array-loop docs show `{query_array:raw @key:'...'}` for other contexts; that variant renders as literal unresolved text here and fails silently (no console error, no error banner — just one bad datapoint). Use the exact placeholder format shown in this element's own control hints.

Single dataset — omit `flatArray_DatasetIdentifier` or leave it unmapped:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamicchart.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamicchart",
  "settings": {
    "chart_type": "line",
    "hasLoop": true,
    "queryLoopDatapoints": "array",
    "query": {
      "objectType": "array",
      "arrayEditor": "[{\"datapoint\":\"Q1\",\"value\":65},{\"datapoint\":\"Q2\",\"value\":59},{\"datapoint\":\"Q3\",\"value\":80}]"
    },
    "flatArray_XAxisLabel": "{query_array @key:'datapoint'}",
    "flatArray_Value": "{query_array @key:'value'}"
  }
}
```

Multi dataset — add a `dataset` key per row and map `flatArray_DatasetIdentifier`; rows are grouped by that key into separate series automatically, including sparse rows (a dataset missing an entry for a given category just has a gap there, not an error):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamicchart.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "arrayEditor": "[{\"dataset\":\"Product A\",\"datapoint\":\"Q1\",\"value\":65},{\"dataset\":\"Product B\",\"datapoint\":\"Q1\",\"value\":25},{\"dataset\":\"Product B\",\"datapoint\":\"Q4\",\"value\":45}]",
  "flatArray_DatasetIdentifier": "{query_array @key:'dataset'}",
  "flatArray_XAxisLabel": "{query_array @key:'datapoint'}",
  "flatArray_Value": "{query_array @key:'value'}"
}
```

A 9-row multi-dataset array correctly groups into 3 series across 4 categories, with a category missing from a series simply producing no bar there.

---

## Dataset coloring

Two entirely different mechanisms depending on series count, controlled by `maybeDynamicColors` (`manually` default, or `dynamic`):

- **Single dataset** (manual mode, or any loop mode that resolves to one series): each `content_items` row's own `color` field applies directly to that bar/point/slice. This is the only case where per-item colors work.
- **Multiple datasets** (`dataSets` loop mode, or multi-dataset `array` mode): per-item `content_items.color` is **not** used for series color. Instead:
  - `maybeDynamicColors: "manually"` (default) → `datasetItems` repeater, one color entry per expected series, applied in series order.
  - `maybeDynamicColors: "dynamic"` → single `datasetColor` field, a dynamic tag re-evaluated per loop iteration (e.g. `{cf_brand_color}` on a Datasets-mode posts loop).

**A loop-mode chart with no dataset-color settings configured at all renders every series/point in Chart.js's default black.** This isn't a bug to work around — it just means dataset coloring is a separate, opt-in step from wiring up the data itself. Don't assume colors will "just work" once the loop renders correctly.

---

## Chart-type-gated controls

`chart_type` (`bar` / `line` / `doughnut`) hides entire groups of controls via `required` conditions:

| Group | Shown for |
|---|---|
| `chartDirection`, `stacked`, `reverseDatasets` | `bar`, `line` (hidden for `doughnut`) |
| `lineColor` | `line`, `doughnut` |
| `tension`, `lineWidth`, point size/border (`linePointRadius`, `linePointBorderWidth`) | `line` only |
| `barBorderRadius`, `barBorderColor`, `barBorderWidth` | `bar` only (i.e. `chart_type != line/doughnut`) |
| `pieBorderWidth`, `pieCutOut`, `rotation`, `circumference` | `doughnut` only |
| The entire **Axes** control group | hidden for `doughnut` (pie charts have no x/y axes) |

Also gated, independent of chart type:
- `xAxisUnits`/`xAxisUnitPosition` require `xAxisDisplay != false`.
- Every axis title/label/grid/border control requires its own axis's `xAxisDisplay`/`yAxisDisplay` to be enabled — turning an axis off hides ~10 downstream controls, not just itself.
- Tooltip, legend, and data-label sub-controls each gate the same way behind their own `...Display` toggle.
- `datasetItems` requires `maybeDynamicColors != dynamic`; `datasetColor` requires `maybeDynamicColors = dynamic` — exact inverse of each other, never both relevant at once.

---

## Rendered DOM: a canvas — style exclusively through this element's own controls, not CSS

```html
<div class="brxe-xdynamicchart" data-x-id="{id}" data-x-dynamic-chart="bar" style="aspect-ratio: 2 / 1;">
  <canvas role="img" aria-label="Chart"></canvas>
</div>
```

That's the entire structure. Chart.js draws everything — bars, lines, slices, axes, legend, tooltips, grid — directly onto the `<canvas>` pixel surface. **There is no inner markup to target with CSS at all**; every visual aspect of the chart itself (not the root wrapper) has to go through this element's own color/style controls (`content_items.color`, `datasetItems`, `datasetColor`, axis/legend/tooltip style groups, etc.) — there's no CSS fallback for anything Chart.js draws.

**Global Bricks color variables (`var(--primary)`, a color palette swatch, etc.) still work correctly in this element's color controls, despite the canvas being unable to read CSS custom properties natively.** The plugin's own JS detects when a color setting resolves to a `var(--...)` string and reads the actual computed value via `getComputedStyle(document.documentElement).getPropertyValue(...)` before handing it to Chart.js — confirmed live: a bar colored `var(--primary)` and another colored `var(--secondary)` both rendered in their real palette colors, not literal `var()` strings or Chart.js's black default. This means using the site's actual color palette for chart colors (rather than hardcoded hex) works as expected and stays in sync if the palette changes later.

## Never do

- Don't use `{query_array:raw @key:'...'}` for the flat-array fields — this element expects `{query_array @key:'...'}` without `:raw`. The `:raw` variant fails silently (renders as literal text, no error).
- Don't expect `content_items.color` to apply in a multi-dataset context (Datasets loop mode, or multi-dataset array mode) — use `datasetItems` or `datasetColor` instead.
- Don't assume a loop-mode chart will have any series colors without explicitly configuring `maybeDynamicColors` + `datasetItems`/`datasetColor` — the unstyled default is Chart.js black for every series.
- Don't put `hasLoop`/`query` on a child element — this element isn't nestable; the loop lives directly on its own settings, same pattern as `xcalendar`.

## MCP write notes

- After any loop-mode or array-mode write, render the page and check the browser console — a bad `{query_array}` tag or missing loop data fails silently (flat/empty chart, no PHP warning, no JS error) rather than erroring.
- Verify dynamic tags used in `content_items.data` (Datasets mode) or `queryLoopDatapoints_Value` (Data points mode) against real loop data before relying on them — an unresolvable tag just renders as literal text or `0`, not an error.

## If needed: custom behavior via the live instance

This element's controls are a curated subset of Chart.js (e.g. `chart_type` only offers `bar`/`line`/`doughnut`). For anything beyond that, get the real Chart.js instance from `window.xChart.Instances[dataXId]` (keyed by the element's `data-x-id`) in a Code element with `executeCode: true` set, and mutate it directly — `chart.config`, `chart.options`, `chart.update()`, etc. Wait ~31ms after `DOMContentLoaded` before reading it (init is delayed ~30ms, so `Instances[id]` is `undefined` before that). Confirmed live: mutating `chart.config.type` + `chart.update()` swaps in a fully working `polarArea` chart, bypassing the 3-option `chart_type` limit entirely.
