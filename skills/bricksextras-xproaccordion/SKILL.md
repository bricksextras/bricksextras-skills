---
name: xproaccordion
description: "Use when building, debugging, or styling Pro Accordion (xproaccordion) from BricksExtras: FAQ accordions, collapsible content panels, expandable item lists. Covers required hidden per-item structure, CSS classes, and attributes that make JS toggle behavior work."
---

**Requires:** BricksExtras 1.7.3+ with xproaccordion element enabled

# BricksExtras: Pro Accordion (xproaccordion)

Shipped by the **BricksExtras** plugin. A nestable container element — each accordion item is a `block` subtree nested inside it, not a repeater field on the parent. When a person adds this element in the builder UI, Bricks auto-inserts one fully-structured item (see template below) via the element's PHP `get_nestable_item()` default. **Building through the MCP does not get this for free** — nothing auto-inserts it, so every item has to be constructed to match this exact shape or the accordion renders as static, non-interactive content.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xproaccordion.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xproaccordion` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: the hidden structure is the mechanism, not decoration

The plugin's JS binds click handlers and toggles visibility by **CSS class**, not by element type or position. A header missing the `x-accordion_header` class — or a title built as the wrong element type — will render fine visually but never respond to clicks. Every class below is required, not stylistic:

| Piece | Element | Required hidden class | Notes |
|---|---|---|---|
| Item wrapper | `block` | `x-accordion_item` | One per accordion item, direct child of `xproaccordion` |
| Heading tag wrapper | `block` | `x-accordion_heading-wrapper` | **Required intermediate wrapper**, `tag: h4` — sits between the item and the header. Not optional, not legacy drift from an older plugin version — this is the current canonical structure |
| Header | `block` | `x-accordion_header` | The clickable toggle target, child of the heading-tag wrapper — needs `role="button"` + `tabindex="0"` (see below), `_direction: row`, `_justifyContent: space-between`, `_alignItems: center`, `_flexWrap: nowrap` |
| Title | `text-basic` (tag `span`) | `x-accordion_title` | **Not a `heading` element** — a `text-basic` rendered as `span`, sitting inside the header |
| Toggle icon | `icon` | `x-accordion_icon` | Default icon `ion-ios-arrow-down` / `ionicons` library, `iconSize: 16px` |
| Content wrapper | `block` | `x-accordion_content` | Second child of the item, sibling to the heading-tag wrapper |
| Content inner | `div` | `x-accordion_content-inner` | Wraps the actual content |
| Content | `text` | *(none)* | Rich text — this is the only piece with no required class |

**The header's `role`/`tabindex` are set via the `_attributes` repeater, not a checkbox control** — easy to miss:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproaccordion.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
"_attributes": [
  { "name": "role", "value": "button" },
  { "name": "tabindex", "value": "0" }
]
```

---

## Canonical per-item template (example default structure)

Build each item as this exact nested shape. The whole accordion — parent `xproaccordion` plus every item subtree — can be written in a single `add-element`/`set-page-elements` call.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproaccordion.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block",
  "label": "Accordion Item",
  "settings": {
    "_hidden": { "_cssClasses": "x-accordion_item" }
  },
  "children": [
    {
      "name": "block",
      "label": "Heading tag",
      "settings": {
        "tag": "h4",
        "_hidden": { "_cssClasses": "x-accordion_heading-wrapper" }
      },
      "children": [
        {
          "name": "block",
          "label": "Accordion header",
          "settings": {
            "_direction": "row",
            "_justifyContent": "space-between",
            "_alignItems": "center",
            "_flexWrap": "nowrap",
            "_hidden": { "_cssClasses": "x-accordion_header" },
            "_attributes": [
              { "name": "role", "value": "button" },
              { "name": "tabindex", "value": "0" }
            ]
          },
          "children": [
            {
              "name": "text-basic",
              "settings": {
                "tag": "span",
                "text": "Your question here",
                "_hidden": { "_cssClasses": "x-accordion_title" }
              }
            },
            {
              "name": "icon",
              "label": "Toggle icon",
              "settings": {
                "icon": { "icon": "ion-ios-arrow-down", "library": "ionicons" },
                "iconSize": "16px",
                "_hidden": { "_cssClasses": "x-accordion_icon" }
              }
            }
          ]
        }
      ]
    },
    {
      "name": "block",
      "label": "Content wrapper",
      "settings": {
        "_hidden": { "_cssClasses": "x-accordion_content" }
      },
      "children": [
        {
          "name": "div",
          "label": "Content",
          "settings": {
            "_hidden": { "_cssClasses": "x-accordion_content-inner" }
          },
          "children": [
            { "name": "text", "settings": { "text": "Your answer here." } }
          ]
        }
      ]
    }
  ]
}
```

**The heading-tag wrapper, header, and content-wrapper blocks are marked `deletable: false`** in the native template (the heading-tag wrapper itself doesn't carry `deletable: false` explicitly in the source, but the header and content-wrapper inside/alongside it do). That's a signal, not just a builder-UI restriction to mimic — these blocks are structural to the toggle mechanism and the heading semantics. If a user asks to "simplify" or "flatten" an accordion item, don't collapse the heading-tag wrapper or either of these two blocks away even if the visual result looks equivalent; doing so removes either the class hooks the JS depends on, or the semantic heading level (`h4`) the wrapper provides.

---

## Rendered DOM (for custom CSS/targeting)

Captured live, one item, closed state:

```html
<div class="brxe-xproaccordion x-accordion" data-x-id="{id}" data-x-accordion="{...}">
  <div class="brxe-block x-accordion_item">
    <h4 class="brxe-block x-accordion_heading-wrapper">
      <div id="brxe-{id}" class="brxe-block x-accordion_header" role="button" tabindex="0">
        <span class="brxe-text-basic x-accordion_title">Question one</span>
        <i id="brxe-{id}" class="ion-ios-arrow-down brxe-icon x-accordion_icon"></i>
      </div>
    </h4>
    <div class="brxe-block x-accordion_content">
      <div class="brxe-div x-accordion_content-inner">
        <div class="brxe-text"><p>Answer one.</p></div>
      </div>
    </div>
  </div>
</div>
```

All the required/removable structure above is already covered by the class table and "known failure modes" sections — this is just confirmation of what it actually resolves to in the browser, plus two things you can't get from the settings JSON at all:

- **No `aria-expanded` (or any open/closed state attribute) is present in the server-rendered HTML at all** — it's added to the header by JS only after the accordion script initializes/toggles. Don't expect `[aria-expanded=...]` selectors to match against a raw HTML fetch (e.g. `get-page-elements` or a non-JS HTTP request) — only a real browser render reflects it.
- Only the header (`x-accordion_header`) and icon (`x-accordion_icon`) keep a generated `id="brxe-{id}"` in this render — the item, heading-wrapper, and content wrapper blocks render with no `id` at all unless one is explicitly set. Don't rely on an `#brxe-{id}` selector at those levels; target by class instead.

---

## Known failure modes

**Missing classes entirely.** Building an item as a plain `block` containing a `heading`/`text-basic` question and a `div`/`text` answer — with real content, correct visual layout, even a manually-added icon — renders and looks complete in the static markup, but has no click/expand behavior, because none of the required classes are present for the JS to bind to.

**Flattened structure (dropping the heading-tag wrapper).** A second, more subtle failure: building the item with `x-accordion_item` → `x-accordion_header` directly (skipping the intermediate `x-accordion_heading-wrapper` block) *also* looks plausible and was previously miscategorized as acceptable — even as "possible drift from an older plugin version" when found on an existing page instance. **It is not drift; it is the correct current structure.** Always include the heading-tag wrapper. Do not simplify it away based on an existing instance appearing to omit it — cross-check against this template instead, since a hand-built instance could itself be wrong.

**Wrong element type for the title.** Using a `heading` element (e.g. `h4`) for the question text instead of `text-basic` rendered as `span` is also incorrect — the semantic heading level belongs on the wrapper block (via its `tag` setting), not on the title text itself.

Always use the template above verbatim for the structure/class/attribute skeleton; only the question text, answer text, and optionally the icon differ per item.

---

## Item-content variables

Default item title text is `"Title {item_index}"` — `{item_index}` is a live placeholder the plugin substitutes with the item's position. Replace the whole string with real content per item; don't try to preserve `{item_index}` in hand-authored items unless numbered titles are actually wanted.

---

## Query loop for dynamic items

To populate accordion items from posts/terms/users instead of hand-building each one, build **a single item** using the canonical template above, then put `hasLoop: true` directly on that item's `x-accordion_item` block (same block that carries the `_hidden` class) — not on the parent `xproaccordion`. The single item duplicates once per query result, in place of hand-building N separate items.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproaccordion.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block",
  "label": "Accordion Item",
  "settings": {
    "_hidden": { "_cssClasses": "x-accordion_item" },
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": ["post"], "posts_per_page": 10 }
  },
  "children": [ /* heading-tag wrapper → header → title/icon, content wrapper → content-inner → text, same as the static template */ ]
}
```

**`hasLoop` only duplicates the block — it does not make child content dynamic on its own.** Any text field left as a plain string (e.g. the answer body) repeats identically for every looped item; only fields you explicitly set to a dynamic tag (`{post_title}`, `{post_excerpt}`, etc.) actually vary per iteration. A common half-finished pattern: the title (`x-accordion_title` text) set to `{post_title}` but the content (`x-accordion_content-inner` → `text`) left as static placeholder text — that renders correctly (no error, no silent failure) but shows the same body copy under every question. If per-item body content is wanted, put a dynamic tag there too.

---

## Workflow

1. **Confirm plugin active, pull live schema** for `xproaccordion` via the adapter dispatch shown above.
2. **Build the parent `xproaccordion` with all its items nested inline**, using the canonical template above (including the heading-tag wrapper) for each item, with real question/answer text substituted in — one `add-element` or `set-page-elements` call.
4. **Verify rendered output**, not just settings read-back: confirm `x-accordion_item` / `x-accordion_heading-wrapper` / `x-accordion_header` / `x-accordion_title` / `x-accordion_icon` / `x-accordion_content` / `x-accordion_content-inner` classes and `role="button"`/`tabindex="0"` are all actually present in the DOM — those are the real behavior hooks, and a settings-only check can't confirm the JS will bind correctly.
