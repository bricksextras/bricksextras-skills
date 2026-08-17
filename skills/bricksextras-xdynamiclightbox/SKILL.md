---
name: xdynamiclightbox
description: "Use when building or debugging the Dynamic Lightbox element (xdynamiclightbox) from BricksExtras: a lightbox/modal built on GLightbox. Covers the four lightboxContent modes and — critically — that in inline mode the element's own children are the lightbox CONTENT, not the trigger; the trigger is a separate link element outside it."
---

**Requires:** BricksExtras 1.7.3+ with xdynamiclightbox element enabled

# BricksExtras: Dynamic Lightbox (xdynamiclightbox)

Shipped by the **BricksExtras** plugin, built on the GLightbox library (`.gcontainer`/`.gslide`/`.gclose` classes in rendered markup). Nestable, but **what "nested" means depends entirely on `lightboxContent` mode** — this is the single biggest thing to get right before building anything with this element.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xdynamiclightbox.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xdynamiclightbox` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: four `lightboxContent` modes, and they place the trigger and content in opposite structural roles

| Mode | Where the **trigger** lives | Where the **content** comes from |
|---|---|---|
| `inline` (default) | A separate link element, **outside** the `xdynamiclightbox` element, matched via `linkSelector` | The `xdynamiclightbox` element's own **nested children** |
| `manual` | A separate link element, **outside** the `xdynamiclightbox` element, matched via `linkSelector` | Whatever the matched link's `href` points to — GLightbox auto-detects image/video/YouTube/Vimeo from the URL. No children needed inside `xdynamiclightbox`. |
| `iframe` | **Nested inside** `xdynamiclightbox` as its children | `contentSource` (a URL setting) — optionally scoped to one part of that page via `contentSelector` |
| `gallery` | **Nested inside** `xdynamiclightbox` as its children | `imageGallery` — see "Gallery mode" section below for its exact value shape |

**The easy mistake: building `inline` mode with the trigger label nested inside `xdynamiclightbox` and the actual content in a separate hidden block elsewhere.** That's backwards for this mode specifically — it's exactly the opposite arrangement `iframe`/`gallery` use, and `inline` is the mode most people reach for first (it's the default). Correct structure:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamiclightbox.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xdynamiclightbox",
  "label": "Post Lightbox",
  "settings": {
    "lightboxContent": "inline",
    "linkSelector": ".lightbox-trigger"
  },
  "children": [
    { "name": "heading", "settings": { "text": "{post_title}", "tag": "h3" } },
    { "name": "image", "settings": { "useDynamicData": "{featured_image}" } }
  ]
}
```

...with the trigger as a **sibling, not a child**, sitting outside the `xdynamiclightbox` element, and genuinely a link element (Bricks `button`, which renders as `<a>`) — not a `block`, `div`, or `heading`:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamiclightbox.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "button",
  "label": "Lightbox Trigger",
  "settings": {
    "text": "View",
    "_cssClasses": "lightbox-trigger",
    "link": { "type": "external", "url": "#" }
  }
}
```

`linkSelector` is the same class-based cross-element targeting pattern used elsewhere in BricksExtras (see `bricksextras-start-here`'s "Targeting another element" section) — point it at a class on the trigger link, not an `_cssId`. `componentScope` exists on this element too, for the same component-duplication-safety reason as `xproslidercontrol`/`xcopytoclipboard`.

For `manual` mode, the same trigger-outside/`linkSelector` wiring applies, but there's no content to nest — the trigger's own `href` is the content source (e.g. a YouTube URL renders as a video lightbox automatically).

---

## Gallery mode: the `imageGallery` value shape

`imageGallery` (type `image-gallery`) isn't a generic repeater — it's a dedicated control type, and its value shape differs depending on whether the images are picked manually or sourced from a dynamic field. Confirmed live from real builder-saved settings:

**Manually picked images** — an `images` array of objects, each with `id`, `full` (original/unscaled URL), and `url` (the URL at whatever size the gallery is set to display), plus a top-level `size` (a literal `WIDTHxHEIGHT` string, not a named Bricks image size like `large`):

```json
{
  "lightboxContent": "gallery",
  "imageGallery": {
    "images": [
      { "id": 35547, "full": "https://example.com/wp-content/uploads/photo-1-scaled.jpg", "url": "https://example.com/wp-content/uploads/photo-1-1024x835.jpg" },
      { "id": 35546, "full": "https://example.com/wp-content/uploads/photo-2-scaled.jpg", "url": "https://example.com/wp-content/uploads/photo-2-1024x683.jpg" }
    ],
    "size": "2048x2048"
  }
}
```

**Dynamic source** (e.g. an ACF gallery field on the current loop post) — same `useDynamicData` + `size` shape as a normal Bricks `image` control, not the `images` array:

```json
{
  "lightboxContent": "gallery",
  "imageGallery": {
    "useDynamicData": "{acf_gallery}",
    "size": "1536x1536"
  }
}
```

**`maybeGrouping: true` (+ `groupingScope: "queryloop"` inside a loop) is required for gallery mode too — not just for the separate multi-trigger/multi-instance case described below.** Without it, a multi-image gallery has no prev/next navigation between its own images; grouping is what connects a gallery's images into one browsable set, exactly the same mechanism that connects separate lightbox instances, just applied within a single instance's own image list here.

---

## Grouping: two different mechanisms, both producing one navigable set

`maybeGrouping: true` turns a set of lightbox slides into a connected group the visitor can navigate between without closing the lightbox (prev/next arrows, `maybeLoop` to wrap around, `draggable` for swipe, `slideEffect`: `slide`/`fade`/`none`). Which slides end up in that group depends on which of two situations is in play — they aren't variations of the same thing, they're structurally different:

**1. One `xdynamiclightbox` element, several manually-placed triggers.** A single element's `linkSelector` matches multiple trigger links (all sharing the same class) — each matched trigger becomes one slide in the group. This is the case for a hand-built set of items (e.g. several separate image links placed around a page that should all open into one browsable lightbox). `groupingScope` isn't the relevant mechanism here — the grouping comes purely from one `linkSelector` matching many triggers.

**2. The `xdynamiclightbox` element itself is duplicated — inside a query loop or a component.** Each loop iteration (or each component instance) naturally produces its own separate `xdynamiclightbox`, each with its own single trigger and its own content (e.g. "click this post's card to see its full details, then arrow to the next post"). Left alone, each duplicated instance would behave as a fully independent lightbox. `groupingScope` is what makes the plugin instead treat that whole set of duplicated instances as one connected group, so the visitor can arrow between them without the page needing separate per-item wiring:
- `"queryloop"` — group only with the other instances produced by the same query loop.
- `"component"` — group only with the other instances inside the same component instance.
- `"global"` (default) — group with every `xdynamiclightbox` instance on the page that has grouping enabled, regardless of which loop/component produced it.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xdynamiclightbox.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "maybeGrouping": true,
  "maybeLoop": true,
  "groupingScope": "queryloop",
  "slideEffect": "slide"
}
```

---

## Rendered DOM (for custom CSS/targeting)

**Closed state (`inline` mode) — present in raw server HTML, content hidden in a `<template>`:**

```html
<div class="brxe-xdynamiclightbox" data-x-id="{id}" data-x-lightbox="{...}">
  <div class="x-dynamic-lightbox_content" tabindex="0" id="x-dynamic-lightbox_content-{id}-0">
    <template data-x-dynamic-lightbox-template="x-dynamic-lightbox_content-{id}-0">
      <h3 class="brxe-heading">Lightbox Title</h3>
      <div class="brxe-text"><p>Lightbox content here.</p></div>
    </template>
  </div>
</div>
```

Same "content invisible until JS clones it out of a `<template>`" pattern as `xpagetourstep` — a raw HTML fetch will never show the lightbox as visibly open, only this closed shell.

**Open state — GLightbox portals this to a wrapper appended near `<body>`, entirely separate from where the element sits in the page:**

```html
<div id="brxe-{id}" class="glightbox-wrapper brxe-xdynamiclightbox" tabindex="-1">
  <div class="goverlay"></div>
  <div class="gcontainer">
    <div id="glightbox-slider" class="gslider">
      <div class="gslide loaded current">
        <div class="gslide-inner-content">
          <div class="ginner-container">
            <!-- inline mode: -->
            <div class="gslide-media gslide-inline">
              <div class="x-dynamic-lightbox_content ginlined-content" id="x-dynamic-lightbox_content-{id}-0" data-original-content="saved">
                <h3 class="brxe-heading">Lightbox Title</h3>
                <div class="brxe-text"><p>Lightbox content here.</p></div>
              </div>
            </div>
            <!-- manual/image mode instead renders: -->
            <!-- <div class="gslide-media gslide-image"><img src="..." class="zoomable"></div> -->
          </div>
        </div>
      </div>
      <!-- one .gslide per grouped item when maybeGrouping is on; only one .gslide carries "current" -->
    </div>
    <button class="gnext gbtn" aria-label="Next">...</button>
    <button class="gprev gbtn" aria-label="Previous">...</button>
    <button class="gclose gbtn" aria-label="Close">...</button>
  </div>
</div>
```

Notes on this structure:

- **The element's own `id`/classes get transplanted onto `.glightbox-wrapper` (the portalled node), not left behind on the original element in the page.** This is why normal Bricks style-panel controls (typography, background, border, etc. — anything compiling to `#brxe-{id} { ... }`) just work on the open lightbox with zero special handling — the plugin's JS makes sure that id ends up wherever the content actually visually renders. You don't need to think about the portal for standard styling.
- This capture is the single-element/multi-trigger grouping case (see "Grouping" above) — one `xdynamiclightbox`, `linkSelector` matching two trigger links, `manual` mode. Each matched trigger becomes one `.gslide`, with that trigger's own `href` supplying the slide's content (image/video auto-detected). The query-loop/component-duplication grouping case produces the same `.gslide`-per-item structure, just sourced from separate duplicated element instances instead of separate triggers.
- Arrow disabled state is real markup, not just CSS: the edge slide's inactive direction button gets an extra `disabled` class (`gnext gbtn disabled` at the last slide, `gprev gbtn disabled` at the first) — style/hide via that class if the disabled state needs different treatment, since both buttons are otherwise always present regardless of position.

## Never do

- Do not nest the trigger label inside `xdynamiclightbox` in `inline` mode — that's the `iframe`/`gallery` arrangement. In `inline` (and `manual`) mode, children are content (or absent), and the trigger is an external link element wired via `linkSelector`.
- Do not use a `block`, `div`, or `heading` as the trigger — it has to be an actual link element (e.g. Bricks `button`, which renders `<a>`), matching the setting's own label ("Selector (Link)").
- Do not use `_cssId` for the `linkSelector` pairing — use a class, same as every other cross-element targeting field in this plugin.
- Do not leave `groupingScope` on its `"global"` default inside a query loop unless every lightbox instance on the whole page really should share one navigable set — use `"queryloop"` to scope grouping to just that loop's instances.

## MCP write notes

- An inverted trigger/content structure in `inline` mode produces a "valid" write — settings and children are all accepted — but the lightbox opens empty or the trigger does nothing, with no error anywhere in the response.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real GLightbox instance from `window.xDynamicLightbox.Instances[dataXId]` (keyed by the `data-x-id` on the `.brxe-xdynamiclightbox` root) in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), and drive it directly via GLightbox's own API (`lightbox.open()`, `lightbox.setElements()`, etc.). It's registered ~300ms after init, so wait past that before reading it.