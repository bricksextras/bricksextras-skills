---
name: xdynamictable
description: "Use when building or debugging the Dynamic Table element (xdynamictable) from BricksExtras: a sortable/searchable/paginated data table (Grid.js-based) populated by a query loop or manual static rows. Covers the two population modes' different column shapes, per-column sort configuration, and how cell content (including images) is just a dynamic-tag text field. Also covers when this is the WRONG element — tables that need Bricks Query Filters or nested/rich cell content should use xnestabletable instead."
---

**Requires:** BricksExtras 1.7.3+ with xdynamictable element enabled

# BricksExtras: Dynamic Table (xdynamictable)

Shipped by the **BricksExtras** plugin. Wraps Grid.js. **Not nestable** — like `xcalendar`/`xdynamicchart`, the query loop (if used) lives directly on the table element's own settings, and columns/rows are repeater settings, not child elements.

---

## Wrong element if the table needs Bricks Query Filters or nested cell content — use `xnestabletable` instead

**Check this before picking an element, not after building.** Two common table requirements silently rule out `xdynamictable`:

1. **Filtering with Bricks Query Filters (`filter-select`, `filter-checkbox`, etc.).** Query Filters target a real Bricks query-producing element by `_id` — a `Container`/`Block`/`Div`/`Section` (or native Posts/Users/Terms element) carrying `hasLoop`/`query`. `xdynamictable`'s query loop lives on its own **settings** (`hasLoop`/`query` as plain repeater-adjacent fields on the table element itself), not on a real element with a query-loop role Query Filters can bind to. A `filter-select` with `filterQueryId` pointed at an `xdynamictable` renders and looks configured, but the target AJAX swap has nothing valid to hook into.
   - **Reach for `xnestabletable` instead.** Its `tbody > tr` is an actual `div` element with `hasLoop`/`query` on it — a genuine query-producing element, so `filterQueryId` can point at that row's id and Query Filters works normally. Verified end-to-end: a `filter-select` (taxonomy source) targeting an `xnestabletable` row's id narrows the rendered rows via AJAX correctly.
2. **Cell content that needs real nested elements** (an image + heading + button stacked in one cell, a nested query loop inside a cell, an icon next to dynamic text with its own settings, anything beyond a single dynamic-tag string). `xdynamictable`'s `content_items[].data` field is always a flat dynamic-tag/text string — there is no child-element slot per cell. `xnestabletable` cells are real nested elements (`text-basic`, `image`, `button`, or anything else), so arbitrarily rich cell content is a first-class case there and not here.

If neither of those two things is needed — a straightforward sortable/searchable/paginated table with simple text/image cells and no Bricks Query Filters — `xdynamictable` is still the right, simpler choice (Grid.js gives client-side search/sort/pagination for free, which `xnestabletable` doesn't have built in). Don't default to `xnestabletable` for every table; only switch when filtering or nested cell content is actually required.

See skill `bricksextras-xnestabletable` for that element's build pattern.

---

**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xdynamictable.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xdynamictable` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Two population modes — different column shapes, not just a data-source switch

`maybeDynamic` (`"dynamic"` default, or `"static"`) doesn't just toggle where data comes from — it switches which repeater setting is even active, and the **column** repeater itself has a different field set per mode:

| Mode | Active repeater(s) | Column has a `data` field? |
|---|---|---|
| `dynamic` | `content_items` (columns) + `hasLoop`/`query` | Yes — `data` is a dynamic tag evaluated per loop row |
| `static` | `content_items_static` (columns) + `row_items` (rows → cells) | No — column just defines heading/behavior; actual values live in `row_items` |

Don't write a `data` field into `content_items_static` expecting it to populate cells — it isn't in that repeater's field set at all. In static mode, cell content is entered per-row in `row_items[].cell[].cell_content_text` (or `cell_content_editor` when `cell_content_type: true` for rich text).

## Dynamic mode — query loop + columns as dynamic tags

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamictable.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamictable",
  "settings": {
    "maybeDynamic": "dynamic",
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": "post", "posts_per_page": 10 },
    "maybe_sortable": "true",
    "content_items": [
      { "title": "Title", "data": "{post_title}" },
      { "title": "ID", "data": "{post_id}", "data_type": true },
      { "title": "Featured Image", "data": "{featured_image}" }
    ]
  }
}
```

Renders one row per query result, correctly evaluating each column's `data` tag per row. A dynamic tag like `{featured_image}` needs no special "image column" control — the `data` field is always plain text, and Bricks resolves the tag to its own markup (an `<img>` in this case) inline in the cell. There's no dedicated image/media column type to look for in the schema; any dynamic tag that resolves to HTML works the same way.

## Per-column settings (inside `content_items`/`content_items_static`)

| Field | Purpose | Notes |
|---|---|---|
| `title` | Column heading | |
| `data` | Cell value (dynamic tag) | **Dynamic mode only** — absent from `content_items_static` |
| `width` | Min. column width (px) | |
| `attributes` | Repeater of `name`/`value` — custom HTML attributes on the cell | Nested repeater, not a simple field |
| `data_type` | Sort this column as numbers | Mutually exclusive with `data_type_date` via matching `required` conditions on each — don't set both |
| `data_type_date` | Sort this column as dates | See above |
| `date_format` | Date field order for sorting: `auto` (default), `dmy`, `mdy`, `ymd` | Only shown when `data_type_date: true`. `auto` renders no attribute; any explicit value is written to the header cell as `data-x-date-format="{value}"` and read by the JS sort comparator to correctly parse ambiguous numeric dates (e.g. `03/04/2025`) instead of guessing |
| `initial_sort` | `none`/`asc`/`desc` — which column (and direction) the table opens sorted by | Set on whichever column should be the default sort |
| `prevent_sortable` | Disable sorting for this specific column | Only relevant when the table-level `maybe_sortable` is on — see below |
| `scope` | `<th scope>` attribute: `none`/`col`/`row` | Accessibility, rarely needs changing from default |

**Sorting is two-tiered.** `maybe_sortable: "true"` at the table level is what makes *any* column show a sort arrow at all — it's off by default. Once enabled, every column is sortable unless that specific column sets `prevent_sortable: true` to opt back out. To make only one column sortable, enable `maybe_sortable` table-wide and set `prevent_sortable: true` on every other column.

**Numeric columns need `data_type: true` for correct sort order.** Without it, sort is lexicographic (string-based) and a value like `33292` can sort *before* `28839` (comparing character-by-character). Setting `data_type: true` on a post-ID column produces correct numeric ordering (28831, 28833, 28837, 28839, 33292) when sorted ascending.

**`sort_locale`** (table-level, requires `maybe_sortable: "true"`) sets the locale used for text-column sort comparisons (e.g. `"de"`, `"fr"`, `"es"`) — plain text control, placeholder `"Auto"`. Matters for correct alphabetical ordering of locale-specific characters (accents, umlauts, etc.) that plain lexicographic sort gets wrong under the browser's default locale. Leave on `Auto` unless the table's actual content language differs from the site's default and sort order looks wrong for that language.

## Static mode — manual rows

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamictable.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamictable",
  "settings": {
    "maybeDynamic": "static",
    "content_items_static": [
      { "title": "Name" },
      { "title": "Role" }
    ],
    "row_items": [
      { "title": "Row 1", "cell": [
        { "cell_content_text": "Jane" },
        { "cell_content_text": "Admin" }
      ] }
    ]
  }
}
```

`row_items[].cell[]` entries correspond to columns by position (first cell = first column, etc.), not by any explicit key. `cell_content_type: true` on a given cell switches it from the plain `cell_content_text` field to a rich-text `cell_content_editor` field — set per cell, not per row or per column.

## Search, pagination, mobile stacking

All optional, all off/on by their own top-level toggle (`maybe_searchable`, `maybe_pagination`, `maybeStack`) with their sub-settings gated behind that toggle via `required` — same pattern as most style-heavy BricksExtras elements. Nothing unusual here beyond the standard toggle-then-configure shape; check the live/bundled schema for the exact sub-field list rather than assuming, since there are ~20 pagination-footer style settings alone.

## Rendered DOM: the PHP-rendered table is a permanently-hidden data source — Grid.js builds the entire visible table itself

**`render()` outputs a real, complete `<table class="x-dynamic-table_table">` with real rows and cells — but `.x-dynamic-table_table { display: none; }` is a permanent, unconditional CSS rule, not a JS-toggled state.** This isn't progressive enhancement of a visible table; it's inert data, read once by Grid.js and used to build a second, entirely separate table into the empty `<div class="x-dynamic-table_container">` sibling that follows it:

```html
<div class="brxe-xdynamictable" data-x-table="{...}" data-x-id="{id}">
  <table class="x-dynamic-table_table"><!-- display:none always — real data, never shown -->
    <thead><tr>
      <th class="label" scope="col" data-x-type="string" data-x-column="" data-x-initial-sort="asc">Name</th>
      <th class="label" scope="col" data-x-type="string" data-x-column="">Role</th>
    </tr></thead>
    <tbody><tr><td data-x-type="string">Jane</td><td data-x-type="string">Admin</td></tr>...</tbody>
  </table>
  <div class="x-dynamic-table_container"></div>
</div>
```

Once Grid.js runs, it fills `.x-dynamic-table_container` with its own markup — `data-column-id` on each `<th>`/`<td>` is the **lowercased column title** (`"Name"` → `name`), useful as a stable CSS/JS hook independent of column order:

```html
<div class="gridjs gridjs-container">
  <div class="gridjs-head"><div class="gridjs-search"><input class="gridjs-input gridjs-search-input" placeholder="Search.."></div></div>
  <div class="gridjs-wrapper">
    <table class="gridjs-table">
      <thead class="gridjs-thead"><tr class="gridjs-tr">
        <th data-column-id="name" class="gridjs-th gridjs-th-sort gridjs-th-fixed" scope="col">
          <div class="gridjs-th-content">Name</div>
          <button class="gridjs-sort gridjs-sort-neutral" aria-label="Sort column ascending"></button>
        </th>
      </tr></thead>
      <tbody class="gridjs-tbody"><tr class="gridjs-tr">
        <td data-column-id="name" class="gridjs-td" data-x-mobile-label="Name"><span>Jane</span></td>
      </tr></tbody>
    </table>
  </div>
  <div class="gridjs-footer">
    <div class="gridjs-pagination">
      <div class="gridjs-summary" role="status">Showing <b>1</b> to <b>2</b> of <b>2</b> Results</div>
      <div class="gridjs-pages">
        <button title="Previous" aria-label="Previous" disabled>Previous</button>
        <button class="gridjs-currentPage" title="Page 1" aria-label="Page 1">1</button>
        <button title="Next" aria-label="Next" disabled>Next</button>
      </div>
    </div>
  </div>
</div>
```

Notes:

- **If the Grid.js script fails to load or is blocked, the table has no visible fallback at all** — since the source table is unconditionally `display:none`, a JS failure here means an empty-looking element, not a plain static table. This is different from most "JS enhances existing markup" patterns in this plugin.
- **`data-x-mobile-label` is written on every cell regardless of `maybeStack`** — it's the CSS attr-selector hook `maybeStack`'s responsive layout uses to show a label next to each cell's value when stacked; present unconditionally, only visually used once the stack breakpoint is active.
- Style real table content via the `gridjs-*` classes shown above (or the `data-column-id`/`data-x-mobile-label` hooks) — `.x-dynamic-table_table`'s own classes/structure are never visible and not useful as CSS targets.

## Never do

- Don't put a `data` field in `content_items_static` — it isn't part of that repeater's fields; static mode's values live in `row_items` instead.
- Don't set both `data_type` and `data_type_date` on the same column — they're mutually exclusive (each requires the other to be falsy).
- Don't expect column sorting to work with only `prevent_sortable: false` (or omitted) on a column — the table-level `maybe_sortable` must also be `"true"`, since that's the master switch.
- Don't look for a dedicated "image column" control type — image/media cells are just the same `data` text field holding a dynamic tag that happens to resolve to an `<img>`.
- Don't put `hasLoop`/`query` on a child element — this element isn't nestable; the loop lives directly on its own settings.
- Don't leave `date_format` on `auto` for a date column with ambiguous slash/dash-separated numeric dates (e.g. `03/04/2025`) — set `dmy`/`mdy`/`ymd` explicitly to match the actual data, since auto-detection can guess wrong.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real Grid.js instance from `window.xTable.Instances[dataXId]` (keyed by the `data-x-id` on the `.brxe-xdynamictable` root) in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), and drive it directly via Grid.js's own API (`grid.updateConfig()`, `grid.forceRender()`, etc.). It's registered synchronously on init, no artificial delay to wait out.
