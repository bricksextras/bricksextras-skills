---
name: xcontenttimeline
description: "Use when building or debugging the Content Timeline element (xcontenttimeline) from BricksExtras: a vertical or horizontal timeline of marker + content + meta items, optionally populated by a query loop, with alternating left/right layout. Covers the required nested child structure (not derivable from the schema alone), where hasLoop goes for vertical vs horizontal/slider mode, and the alternating-layout gotcha."
---

**Requires:** BricksExtras 1.7.3+ with xcontenttimeline element enabled

# BricksExtras: Content Timeline (xcontenttimeline)

Shipped by the **BricksExtras** plugin. Nestable, but unlike most nestable BricksExtras elements it has **no dedicated per-item element type** — the schema just says `childElement: "div"`, which on its own doesn't tell you the required internal shape. **This skill exists because that shape is not discoverable from the schema — it must be copied from a real working example or this doc.**

## Required per-item structure (exact, not optional)

Every timeline item is a fixed 3-branch nested `div` tree, each branch carrying a specific `_cssClasses` marker via `_hidden._cssClasses`. This is the structure the builder itself generates when you add the element — reproduce it exactly:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcontenttimeline.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "div",
  "settings": { "_hidden": { "_cssClasses": "x-content-timeline_item" } },
  "label": "Timeline Item",
  "children": [
    {
      "name": "div",
      "settings": { "_hidden": { "_cssClasses": "x-content-timeline_content" } },
      "label": "Content (Structure)",
      "children": [
        {
          "name": "div",
          "settings": { "_hidden": { "_cssClasses": "x-content-timeline_content-inner" } },
          "label": "Content inner",
          "children": [
            { "name": "heading", "settings": { "text": "Content here" } }
          ]
        }
      ]
    },
    {
      "name": "div",
      "settings": { "_hidden": { "_cssClasses": "x-content-timeline_marker" } },
      "label": "Marker  (structure)",
      "children": [
        {
          "name": "div",
          "settings": { "_hidden": { "_cssClasses": "x-content-timeline_marker-inner" } },
          "label": "Marker inner",
          "children": [
            {
              "name": "icon",
              "settings": {
                "icon": { "icon": "ion-ios-calendar", "library": "ionicons" },
                "_hidden": { "_cssClasses": "x-content-timeline_marker-icon" }
              }
            }
          ]
        }
      ]
    },
    {
      "name": "div",
      "settings": { "_hidden": { "_cssClasses": "x-content-timeline_meta" } },
      "label": "Meta (Structure)",
      "children": [
        {
          "name": "div",
          "settings": { "_hidden": { "_cssClasses": "x-content-timeline_meta-inner" } },
          "label": "Meta inner",
          "children": [
            { "name": "text-basic", "settings": { "text": "Meta content" } }
          ]
        }
      ]
    }
  ]
}
```

Wrap that whole `div` in an `xcontenttimeline` parent, and you have a single default item — exactly what a person gets adding this element in the builder.

**The `x-content-timeline_content`, `x-content-timeline_marker`, `x-content-timeline_marker-inner`, and `x-content-timeline_meta` wrapper divs are structural and not meant to be deleted** — the marker wrapper and marker-inner both carry `deletable: false`, and while `content`/`meta` wrappers don't set that flag explicitly, they're part of the same fixed skeleton the builder always generates. The `content-inner`/`meta-inner` divs and what's inside them (heading, text-basic, icon) are the actual customizable slots — swap those for whatever content/meta/marker-icon you need, but keep the wrapper divs and their classes intact.

Content is a `heading`, meta is a `text-basic`, marker is an `icon` (default `ion-ios-calendar`/`ionicons`) — but none of these element *types* are enforced by the plugin. They're just what the builder scaffolds by default; any element works in those slots as long as the wrapper class structure around it is preserved.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcontenttimeline.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xcontenttimeline` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Vertical, query-loop-populated timeline

Multiple items from a query loop: put `hasLoop`/`query` **on the `x-content-timeline_item` div itself** — same pattern as the general BricksExtras query-loop rule (parent container has no loop, the repeating child does). Everything else in the per-item structure stays exactly as above, with dynamic tags substituted where needed (e.g. `{post_title}` in the content heading):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcontenttimeline.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xcontenttimeline",
  "children": [
    {
      "name": "div",
      "settings": {
        "_hidden": { "_cssClasses": "x-content-timeline_item" },
        "hasLoop": true,
        "query": { "objectType": "post", "post_type": "post", "posts_per_page": 5 }
      },
      "children": [ /* same content/marker/meta structure as above */ ]
    }
  ]
}
```

A 5-post loop renders 5 stacked items, one marker/line dot each, in query order.

---

## Alternating left/right layout

`directionEven` (`row`/`row-reverse`) and `metaAlignEven` (`flex-start`/`flex-end`) alternate every second item's content side and meta alignment, via their own `css` mappings targeting `.x-content-timeline_item:nth-child(2n)` and `...meta-inner` respectively:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcontenttimeline.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xcontenttimeline",
  "settings": {
    "directionEven": "row-reverse",
    "metaAlignEven": "flex-end"
  }
}
```

Odd items render content-left, even items content-right, meta flipping sides accordingly.

---

## Horizontal mode — needs a slider, and the loop moves to a different level

`horizontal: "true"` alone does not make this a slider — it only sets up the CSS hooks (`data-x-horizontal="true"` attribute, `horizontalDirection` controlling whether marker/meta sits above or below the content card). Actually scrolling/paging through items horizontally requires wrapping the whole thing in an **`xproslider`**, with the query loop moved to the slide level:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcontenttimeline.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": { "perPage": "3", "arrows": "true", "pagination": true },
  "children": [
    {
      "name": "block",
      "settings": {
        "_hidden": { "_cssClasses": "x-slider_slide splide__slide" },
        "hasLoop": true,
        "query": { "objectType": "post", "post_type": "post", "posts_per_page": 5 }
      },
      "children": [
        {
          "name": "xcontenttimeline",
          "settings": {
            "horizontal": "true",
            "horizontalDirection": "column-reverse",
            "directionEven": "row-reverse",
            "metaAlignEven": "flex-end"
          },
          "children": [
            {
              "name": "div",
              "settings": { "_hidden": { "_cssClasses": "x-content-timeline_item" } },
              "children": [ /* content/marker/meta structure, NO hasLoop here */ ]
            }
          ]
        }
      ]
    }
  ]
}
```

**This is the opposite of the vertical pattern**: `hasLoop`/`query` live on the *slider's slide block* (`xproslider` > `block`), one level **above** the `xcontenttimeline`, not on the `x-content-timeline_item` div inside it. Each slide contains one full `xcontenttimeline` with exactly one (non-looped) item — the slider's own loop is what produces multiple items, one per slide. Putting `hasLoop` on the item div in this mode would be wrong; it already sits inside an already-looped slide.

Meta/marker render above the content card, per `column-reverse`.

---

## Rendered DOM (for custom CSS/targeting)

The plugin wraps everything you write in two levels you never author yourself: an `.x-content-timeline_list` div around all the `x-content-timeline_item`s, and a sibling `.x-content-timeline_line`/`.x-content-timeline_line-active` pair after the list — the connecting line/progress-line graphic, driven by `scrollEffects` (`data-x-scroll` on the root).

Vertical (two items):

```html
<div id="brxe-{id}" class="brxe-xcontenttimeline" data-x-horizontal="false" data-x-scroll="false" data-x-id="{id}">
  <div data-x-horizontal="false" class="x-content-timeline_list">
    <div class="brxe-div x-content-timeline_item">
      <div class="brxe-div x-content-timeline_content">
        <div class="brxe-div x-content-timeline_content-inner"><h3 class="brxe-heading">Item One</h3></div>
      </div>
      <div class="brxe-div x-content-timeline_marker">
        <div class="brxe-div x-content-timeline_marker-inner"><i class="ion-ios-calendar brxe-icon x-content-timeline_marker-icon"></i></div>
      </div>
      <div class="brxe-div x-content-timeline_meta">
        <div class="brxe-div x-content-timeline_meta-inner"><div class="brxe-text-basic">Jan 2026</div></div>
      </div>
    </div>
    <!-- one such .x-content-timeline_item per item, in order -->
  </div>
  <div class="x-content-timeline_line">
    <div class="x-content-timeline_line-active"></div>
  </div>
</div>
```

Horizontal mode (`horizontal: "true"`) produces the exact same wrapper structure — `.x-content-timeline_list` and the trailing `.x-content-timeline_line`/`.x-content-timeline_line-active` pair are both still present, just with `data-x-horizontal="true"` on the root and on `.x-content-timeline_list` instead of `"false"`. The 3-branch item structure itself is unaffected; only `horizontalDirection` (controlling whether marker/meta sits above or below the content card, via CSS) and the item order in the horizontal slider change the visual layout.

Notes:

- **`.x-content-timeline_list` and `.x-content-timeline_line`/`.x-content-timeline_line-active` are auto-injected — never write them yourself.** Only the `x-content-timeline_item` divs (and everything inside them) belong in the `children` you author; the list wrapper and line elements are generated by the element's own `render()`.
- `.x-content-timeline_line-active` is the progress indicator — its fill/length is driven by `scrollEffects` (`data-x-scroll` reflects whether that's enabled) rather than being a static decorative line.
- `data-x-horizontal` is written in two places (root and `.x-content-timeline_list`) — both always match the `horizontal` setting.

## Never do

- Don't build a bare `xcontenttimeline` > `div` with arbitrary content and expect it to look like a timeline — the marker/content/meta 3-branch structure with its exact `_cssClasses` is what the plugin's CSS actually targets. A plain div child renders as an unstyled blob.
- Don't put `hasLoop` on the item div when building the horizontal/slider variant — it belongs on the slide block instead. Putting it on both, or on the wrong one alone, breaks the loop-per-slide relationship.
- Don't delete or restructure the `x-content-timeline_marker`/`x-content-timeline_marker-inner` wrapper divs — they're marked non-deletable in the builder for a reason (the marker/line-dot positioning CSS depends on this exact nesting).
