---
name: xproslidergallery
description: "Use when populating xproslider slides dynamically from WordPress image gallery via xproslidergallery (BricksExtras), instead of hand-built manual slide blocks. Covers correct placement (inside slider, not outside), verified items value shape, captions, and vertical/gallery-mode interaction."
---

**Requires:** BricksExtras 1.7.3+ with xproslidergallery element enabled

# BricksExtras: Pro Slider Gallery (xproslidergallery)

Shipped by the **BricksExtras** plugin, alongside `xproslider` (the slider itself — see skill `xproslider`) and `xproslidercontrol` (external nav/progress/counter — see skill `xproslidercontrol`). Load the Pro Slider skill alongside this one — a Gallery element is never used on its own, always inside a slider.

This element populates a slider's slides dynamically from a WordPress image gallery (native media library picker), instead of hand-built nested slide elements. It has no slider behavior of its own — it's a slide-generating data source that lives *inside* the slider it feeds.

**A different mechanism from a query-looped slide populated from a dynamic-data image field (e.g. an ACF Gallery field).** That case doesn't use this element at all — it's a manual slide `block` with `hasLoop`/`query` set to BricksExtras' `queryLoopExtras` gallery sub-type (see `bricksextras-query-loop-extras`), per the Pro Slider skill's query-loop section. Check which one the task actually needs before defaulting to this element.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xproslidergallery.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xproslidergallery` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Placement

`xproslidergallery` goes inside the slider, as a direct child, in place of manual slide blocks — not alongside them.

Enable `galleryMode: true` on the parent `xproslider`'s own settings, then nest a single `xproslidergallery` element directly inside it:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslidergallery.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": { "galleryMode": true, "type": "fade", "_height": { "number": 600, "unit": "px" } },
  "children": [
    { 
      "name": "xproslidergallery", 
      "settings": { 
        "items": { 
          "images": [
            {
              "id": 34717,
              "full": "https://be.local/wp-content/uploads/2026/07/image-scaled.png",
              "url": "https://be.local/wp-content/uploads/2026/07/image-1024x568.png"
            },
            {
              "id": 30164,
              "full": "https://be.local/wp-content/uploads/2026/02/image2-scaled.jpg",
              "url": "https://be.local/wp-content/uploads/2026/02/image2-1024x683.jpg"
            }
          ], 
          "size": "large" 
        }
      } 
    }
  ]
}
```

Do not mix manual `block` slides and a Gallery child in the same slider — pick one population method per slider instance.

**Gotcha — two separate requirements:** `galleryMode` on the slider and the presence of a nested Gallery child must both be true, or the gallery is inert. Turning on `galleryMode` with no Gallery child renders an empty slider; nesting a Gallery element without enabling `galleryMode` on the parent does nothing.

When `galleryMode` is on, the slider's own `listTag` control becomes irrelevant — this element controls the list/slide tags instead via its own `listTag`/`slideTag`, defaulting to `ul`/`li`.

**No manual slide classes needed.** Unlike hand-built `block` slides (which require `x-slider_slide splide__slide` set manually — see the Pro Slider skill), this element applies those classes to its generated `<li>` slides automatically.

**Making gallery images fill the slide (both dimensions) needs two `xproslider`-level checkboxes, in a specific order.** `imageWidth` and `imageForceWidth` (both on the parent slider, not this element) write colliding CSS to the same image selectors — see the Pro Slider skill's gotchas section for the full mechanism. Short version: enable both, `imageWidth` before `imageForceWidth` in the settings write, or the image won't stretch to fill the slide's width correctly even though both checkboxes report as checked.

---

## Rendered DOM structure — why `galleryMode` has to be enabled on the slider

This is the mechanism behind the "two separate requirements" gotcha above, not just a rule to memorize. `xproslidergallery` itself renders as the `<ul>` list element — **it doesn't sit inside a list, it IS the list** — and generates all the `<li>` slides as its own children:

```html
<div id="brxe-qq155u" class="brxe-xproslider ... splide x-slider" data-x-slider="{...}">
  <div class="splide__track x-splide__track">
    <ul id="brxe-dgogz9" class="brxe-xproslidergallery splide__list x-splide__list x-slider-gallery" data-animation-type="fade">
      <li class="x-slider_slide splide__slide" data-x-caption="...">
        <div class="x-slider_slide-image" style="--x-slider-gallery-image: url(...)"></div>
      </li>
      <!-- one <li> per gallery image, generated automatically -->
    </ul>
  </div>
</div>
```

Compare this to the Pro Slider skill's manual-slide structure, where `xproslider` itself injects a plain `<div class="splide__list">` wrapper around hand-built `block` slides. `xproslidergallery`'s own `<ul>` already carries the `splide__list`/`x-splide__list` classes — it's a drop-in replacement for that wrapper, not content that goes inside it.

**This is exactly why `galleryMode: true` must be set on the parent slider.** It's the flag that tells `xproslider` *"don't inject your own `.splide__list` div — a child element is going to supply the list wrapper itself."* Without `galleryMode` enabled, the slider would inject its default list wrapper as normal, and `xproslidergallery`'s own `<ul class="splide__list">` would end up nested inside it — two list wrappers, one Splide track, broken/duplicated structure. With a manual `block` slide, there's no child capable of supplying its own list wrapper, so the slider's default injection is exactly what's needed — which is also why gallery mode and manual slide blocks can never be mixed in the same slider instance.

---

## Verified `items` value shape

The schema gives no `value_format` hint for `items` (`controlType: image-gallery`) — unlike almost every other control, which does. **The correct, native shape is an object, not a flat array** — matching a real builder-saved instance:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslidergallery.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "items": {
    "images": [
      { 
        "id": 1234, 
        "full": "https://example.com/wp-content/uploads/.../image-1-scaled.jpg",
        "url": "https://example.com/wp-content/uploads/.../image-1-1024x683.jpg"
      },
      { 
        "id": 5678, 
        "full": "https://example.com/wp-content/uploads/.../image-2-scaled.jpg",
        "url": "https://example.com/wp-content/uploads/.../image-2-1024x768.jpg"
      }
    ],
    "size": "large"
  }
}
```

**Always set `size: "full"` explicitly, especially for a full-width or large slider.** `size` selects which registered WordPress image size gets used for the rendered `<img>`. Leaving it unset (or using the flat-array shape below, which has no way to express it at all) risks the plugin falling back to a smaller registered size — fine for small thumbnails, but a visibly soft/blurry image once stretched across a full-width hero slide. Match `size` to how large the slider actually renders; `full` is the safe default for anything full-width.

**A flat array of bare `{ "id": 1234 }` objects (no `images`/`size` wrapper) also renders correctly** — this was the previously-documented shape here, and it isn't wrong exactly, but it's the minimal/degraded form: it has no `size` control, so you can't influence which image size gets used, and it doesn't match what the builder itself actually saves. Prefer the `images`/`size` object shape above for anything beyond a quick throwaway build.

Each `id` must be a real attachment ID on the target site — the requirement is the ID specifically, since that's what's needed to resolve real registered image sizes (not just a URL). Look these up rather than guessing:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslidergallery.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "ability_name": "bricks/find-media",
  "parameters": { "mimeType": "image/" }
}
```

Whatever media-lookup ability the current MCP connection exposes (e.g. `bricks/find-media` on the native Bricks MCP) returns each match's `id`, `url`, `filename`, and generated `sizes`, which is what you need to build the `items` array above. Images render in the exact order given, and the rendered `<img>` initially carries a blank SVG placeholder `src` with the real URL in `data-splide-lazy` — that's expected Splide lazy-loading, not a broken image; the real `src` swaps in as the slide becomes visible.

---

## Aspect ratio

`aspectRatio` sets `aspect-ratio` CSS directly on `.x-slider_slide-image` and the `img` inside it — a plain number control (e.g. `"16/9"`, `"1"`), not a units/dimension field. Use it to force uniform image proportions across a gallery-mode slider regardless of each source image's native dimensions; combine with `objectFit: cover` so images fill the ratio instead of being squashed.

## Captions

The `caption` checkbox (plus `captionTypography`/`captionBackground`/`captionBorder`/position controls) shows the WordPress attachment caption field as an overlay per image. **This is also the data source for `xproslidercontrol`'s `slideContent` control type when set to `slideContentType: caption`** — see the Control skill. There's no caption source for manual slide blocks; caption-driven readouts only work in gallery mode.

---

## Vertical sliders work with gallery mode

`direction: ttb` (set on the parent `xproslider`) combined with `galleryMode: true` renders correctly — the plugin's own stylesheet has explicit rules pairing `.splide--ttb` with `.x-slider-gallery`. As with any vertical slider, the parent needs an explicit `height` (or aspect-ratio) or it has no visible extent.

---

## Verified pattern: side-by-side gallery-mode sliders (fade + vertical)

Tested end-to-end: two independent gallery-mode sliders placed side by side in a flex row, sourcing the same WordPress images, one fade and one vertical.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslidergallery.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": { "galleryMode": true, "type": "fade", "height": "600px", "arrows": "true", "pagination": true },
  "children": [
    { "name": "xproslidergallery", "settings": { "items": { "images": [{ "id": 1234, "full": "https://example.com/.../image-1-scaled.jpg", "url": "https://example.com/.../image-1-2048x1365.jpg" }, { "id": 5678, "full": "https://example.com/.../image-2-scaled.jpg", "url": "https://example.com/.../image-2-2048x1365.jpg" }], "size": "full" }, "objectFit": "cover" } }
  ]
}
```

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslidergallery.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": { "galleryMode": true, "type": "slide", "direction": "ttb", "height": "600px", "perPage": 3, "gap": "10px", "arrows": "true", "pagination": false },
  "children": [
    { "name": "xproslidergallery", "settings": { "items": { "images": [{ "id": 1234, "full": "https://example.com/.../image-1-scaled.jpg", "url": "https://example.com/.../image-1-2048x1365.jpg" }, { "id": 5678, "full": "https://example.com/.../image-2-scaled.jpg", "url": "https://example.com/.../image-2-2048x1365.jpg" }], "size": "full" }, "objectFit": "cover" } }
  ]
}
```

Both rendered the correct images in the correct order with no manual slide blocks and no manually-set hidden classes. These two sliders were **not synced to each other** — for that, use `xproslider`'s own `isNavigation`/`syncSelector` (see the Pro Slider skill), just with one set to `direction: ttb`.

**Gallery mode also works combined with `isNavigation`/`syncSelector` sync** — two `xproslider` instances, both `galleryMode: true` with their own `xproslidergallery` child (same image set on both), one as the main (`type: fade`, `_cssId` set) and one as the synced thumbnail strip (`type: slide`, `isNavigation`/`syncSelector` targeting the main): correct images in order on both, and the click-a-thumbnail-drives-the-main-slide sync behavior works end-to-end in the browser. Full pattern in the Pro Slider skill.

---

## Workflow for a gallery-mode slider

1. **Look up real attachment IDs first** — via whatever media-lookup ability the current MCP connection exposes (e.g. `bricks/find-media` on the native Bricks MCP) — never guess them.
2. **Get the current schema** for `xproslidergallery` — check the bundled schema first per `bricksextras-element-schemas`; only if the bundle is missing or stale, call whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` with `parameters: {"elementName": "xproslidergallery"}` on the native Bricks MCP). Do the same for `xproslider` if not already done, since `galleryMode` lives on the slider's own settings.
3. **Set `galleryMode: true` on the slider**, then nest the Gallery element as its **direct child**, replacing any manual slide blocks.
4. **Build the `items` object** (`images` array + `size`) using the verified shape above — set `size: "full"` for anything full-width or otherwise large.
5. **Decide on captions** if a per-image text overlay or a `slideContent: caption` Control readout is wanted (see the Control skill for the latter).
6. **Insert and verify** the rendered output actually contains real `<img>`/`data-splide-lazy` URLs matching the intended attachment IDs, rather than trusting the settings alone.
