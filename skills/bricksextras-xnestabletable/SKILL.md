---
name: xnestabletable
description: "Use when building or debugging Nestable Table (xnestabletable) from BricksExtras: HTML tables built with nested child elements (thead/tbody/tr/th/td), supporting query loops on rows. Covers required structure, custom tags, and query loop placement."
---

**Requires:** BricksExtras 1.7.3+ with xnestabletable element enabled

# BricksExtras: Nestable Table (xnestabletable)

Shipped by the **BricksExtras** plugin. A **nestable** HTML table builder where the table structure (thead, tbody, rows, cells) is created using nested child elements with custom tags, not repeater fields.

**Key difference from `xdynamictable`:** This is nestable — the table structure is built with child elements. `xdynamictable` uses repeater fields and is not nestable.

**This is the more modern, more flexible of the two table elements — default to this one.** `xdynamictable` exists for one thing this element doesn't have: Grid.js's built-in sort/search/pagination behavior. That's the only reason to reach for it instead.

**Pick this element (over `xdynamictable`) whenever either is true:**
- The table needs Bricks Query Filters (`filter-select`, `filter-checkbox`, etc.). Query Filters target a real query-producing element by `_id`, and this element's `tbody > tr` is a genuine `div` with `hasLoop`/`query` — a valid filter target. `xdynamictable`'s loop lives on the table's own settings, not on an element Query Filters can bind to, so a filter pointed at it has no usable target even though it looks configured. Verified end-to-end: `filter-select` (taxonomy source) → `filterQueryId` set to this element's row id → AJAX narrows the rendered rows correctly. **Note the asymmetry with `xdynamictable`'s own sorting:** Query Filters only apply when the table is populated by an actual query loop — they have nothing to filter against `xdynamictable`'s static/manual rows either way, so this consideration only comes up for query-driven tables in the first place.
- A cell needs real nested content (image + heading + button together, a nested loop, anything beyond one dynamic-tag string).

**Pick `xdynamictable` instead when its Grid.js sort/search/pagination is actually wanted and neither of the above applies.** That behavior works on both of `xdynamictable`'s population modes — dynamic (query loop) and static (hand-entered rows) — so it's available even for a fully static table, which is a case this element has no equivalent for (no built-in client-side sort/search/pagination here at all; only Query Filters' `sort`/`per_page` actions get close, and those need a query loop to work). See `bricksextras-xdynamictable`'s own "wrong element" section for the fuller writeup on the filtering/nesting side — this is the same fact from the other side.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xnestabletable.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xnestabletable` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Required structure

The table requires a specific nested structure using `div` elements with `customTag` settings to render proper HTML table elements:

```
xnestabletable (parent)
├── div (customTag: "thead") - Header
│   └── div (customTag: "tr") - Header row
│       ├── div (customTag: "th") - Header cell
│       │   └── text-basic - Cell content
│       └── div (customTag: "th") - Header cell
│           └── text-basic - Cell content
└── div (customTag: "tbody") - Body
    └── div (customTag: "tr") - Body row (hasLoop + query here)
        ├── div (customTag: "td") - Body cell
        │   └── text-basic - Cell content (dynamic data)
        └── div (customTag: "td") - Body cell
            └── text-basic - Cell content (dynamic data)
```

**Critical:** Each structural element must be a `div` with `tag: "custom"` and the appropriate `customTag` value (`thead`, `tbody`, `tr`, `th`, `td`).

---

## Default structure (from builder)

When a user adds this element in the builder, Bricks auto-inserts this default structure. **Building through MCP does not get this for free** — you must construct the full structure manually.

**Verified default structure:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xnestabletable.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xnestabletable",
  "children": [
    {
      "name": "div",
      "label": "Head",
      "settings": {
        "tag": "custom",
        "customTag": "thead"
      },
      "children": [
        {
          "name": "div",
          "label": "Row",
          "settings": {
            "tag": "custom",
            "customTag": "tr"
          },
          "children": [
            {
              "name": "div",
              "label": "Cell",
              "settings": {
                "tag": "custom",
                "customTag": "th"
              },
              "children": [
                {
                  "name": "text-basic",
                  "settings": {
                    "text": "Post title",
                    "tag": "span"
                  }
                }
              ]
            },
            {
              "name": "div",
              "label": "Cell",
              "settings": {
                "tag": "custom",
                "customTag": "th"
              },
              "children": [
                {
                  "name": "text-basic",
                  "settings": {
                    "text": "Date",
                    "tag": "span"
                  }
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "name": "div",
      "label": "Body",
      "settings": {
        "tag": "custom",
        "customTag": "tbody"
      },
      "children": [
        {
          "name": "div",
          "label": "Row",
          "settings": {
            "tag": "custom",
            "customTag": "tr"
          },
          "children": [
            {
              "name": "div",
              "label": "Cell",
              "settings": {
                "tag": "custom",
                "customTag": "td"
              },
              "children": [
                {
                  "name": "text-basic",
                  "settings": {
                    "text": "Cell content",
                    "tag": "span"
                  }
                }
              ]
            },
            {
              "name": "div",
              "label": "Cell",
              "settings": {
                "tag": "custom",
                "customTag": "td"
              },
              "children": [
                {
                  "name": "text-basic",
                  "settings": {
                    "text": "Cell content",
                    "tag": "span"
                  }
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

---

## Query loop placement

**The query loop goes on the row element (`tr`), not on the parent table.**

For a dynamic table populated by a query loop:

1. **Header row (`thead > tr`):** No query loop — this is static
2. **Body row (`tbody > tr`):** Add `hasLoop: true` and `query: {...}` here

**Example: Posts table with query loop**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xnestabletable.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xnestabletable",
  "children": [
    {
      "name": "div",
      "label": "Head",
      "settings": {"tag": "custom", "customTag": "thead"},
      "children": [
        {
          "name": "div",
          "label": "Row",
          "settings": {"tag": "custom", "customTag": "tr"},
          "children": [
            {
              "name": "div",
              "settings": {"tag": "custom", "customTag": "th"},
              "children": [
                {"name": "text-basic", "settings": {"text": "Post Title", "tag": "span"}}
              ]
            },
            {
              "name": "div",
              "settings": {"tag": "custom", "customTag": "th"},
              "children": [
                {"name": "text-basic", "settings": {"text": "Post ID", "tag": "span"}}
              ]
            }
          ]
        }
      ]
    },
    {
      "name": "div",
      "label": "Body",
      "settings": {"tag": "custom", "customTag": "tbody"},
      "children": [
        {
          "name": "div",
          "label": "Row",
          "settings": {
            "tag": "custom",
            "customTag": "tr",
            "hasLoop": true,
            "query": {
              "objectType": "post",
              "post_type": "post",
              "posts_per_page": 10
            }
          },
          "children": [
            {
              "name": "div",
              "settings": {"tag": "custom", "customTag": "td"},
              "children": [
                {"name": "text-basic", "settings": {"text": "{post_title}", "tag": "span"}}
              ]
            },
            {
              "name": "div",
              "settings": {"tag": "custom", "customTag": "td"},
              "children": [
                {"name": "text-basic", "settings": {"text": "{post_id}", "tag": "span"}}
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

---

## Cell content

Each cell (`th` or `td`) contains a child element for content. Most commonly:

- **`text-basic`** - For text or dynamic data tags
- **`heading`** - For header cells with heading tags
- **`image`** - For images
- **Any Bricks element** - Cells can contain any element

Dynamic data tags (like `{post_title}`) are placed in the `text` setting of the child element, not on the cell itself.

---

## Table-level settings

The `xnestabletable` parent element has styling settings but no content/structure settings:

- **Header styles** (`header_group`) - Typography, background, padding for `th` elements
- **Row styles** (`rows_group`) - Typography, background, padding for `td` elements, alternating row colors
- **Sticky header** (`sticky_header_group`) - Make header sticky on scroll
- **Stack vertically** (`mobile_group`) - Responsive behavior for mobile

These are all styling controls — the actual table structure comes from the nested children.

---

## Rendered DOM: real semantic tags, plus one auto-injected attribute per cell

The `div` + `customTag` construction genuinely swaps the rendered tag — the output is a real `<table>`/`<thead>`/`<tr>`/`<th>`/`<tbody>`/`<td>` tree, not `<div>`s styled to look like a table:

```html
<table class="brxe-xnestabletable" data-x-nestable-table="{&quot;stack&quot;:&quot;767&quot;}">
  <thead class="brxe-div"><tr class="brxe-div">
    <th class="brxe-div"><span class="brxe-text-basic">Name</span></th>
    <th class="brxe-div"><span class="brxe-text-basic">Role</span></th>
  </tr></thead>
  <tbody class="brxe-div"><tr class="brxe-div">
    <td class="brxe-div" data-x-mobile-label="Name"><span class="brxe-text-basic">Jane</span></td>
    <td class="brxe-div" data-x-mobile-label="Role"><span class="brxe-text-basic">Admin</span></td>
  </tr></tbody>
</table>
```

**Every `td` gets a `data-x-mobile-label` attribute automatically, matched from the corresponding column's header text — this is not something you write yourself, and it's present even when `maybeStack` (mobile stacking) is off.** The value is derived from the `thead` row's cell content at the same column position, not from any per-cell setting. This is the real mobile-stacking hook (same `data-x-mobile-label` pattern `xdynamictable` uses) — style the stacked label via CSS attribute selectors against it if a design needs to customize the label presentation, rather than assuming it needs to be authored per cell.

## Gotchas

- **Must use `div` with `customTag`** - Don't try to use actual `thead`/`tbody`/`tr`/`th`/`td` element names; Bricks doesn't have those. Use `div` with `tag: "custom"` and `customTag: "thead"` etc.
- **Query loop on the row, not the table** - `hasLoop` and `query` go on the `tbody > tr` element, not on `xnestabletable`
- **Each cell needs a child element** - The `th`/`td` div itself doesn't render text; it needs a `text-basic` or other element as a child
- **No auto-seeded structure via MCP** - Unlike the builder UI, creating an empty `xnestabletable` via MCP gives you an empty table; you must build the full thead/tbody/tr/th/td structure manually
- **Number of cells must match across rows** - Header row and body row must have the same number of cells, or the table layout breaks

---

## Build workflow

1. Create the `xnestabletable` parent
2. Add `div` (thead) child
3. Add `div` (tr) child to thead
4. Add `div` (th) children to the header row (one per column)
5. Add `text-basic` children to each th with header text
6. Add `div` (tbody) child to the table
7. Add `div` (tr) child to tbody with `hasLoop: true` and `query`
8. Add `div` (td) children to the body row (same count as header)
9. Add `text-basic` children to each td with dynamic data tags

**All in one `set-page-elements` or `add-element` call** — the full nested structure must be built together.

---

## Never do

- Don't put `hasLoop`/`query` on the `xnestabletable` parent — it goes on the row element
- Don't use actual HTML table element names — use `div` with `customTag`
- Don't forget `tag: "custom"` when setting `customTag` — both are required
- Don't put text directly in the cell div — it needs a child element like `text-basic`
- Don't mismatch column counts between header and body rows
- Don't expect auto-seeded default structure via MCP — you must build it manually
