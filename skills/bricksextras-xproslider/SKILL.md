---
name: xproslider
description: "Use when building, debugging, or styling Pro Slider (xproslider) from BricksExtras: image/content carousels, hero sliders, synced thumbnail strips. Covers manual, gallery-mode, and code-element slide population, slider-to-slider sync, and type/fade/loop gotchas."
---

**Requires:** BricksExtras 1.7.3+ with xproslider element enabled

# BricksExtras: Pro Slider (xproslider)

Shipped by the **BricksExtras** plugin. Built on **Splide**. Functionally an upgrade to Bricks' own native nestable slider element — same nestable-slide-builder pattern, far more configuration surface (loop/fade transitions, autoplay vs continuous autoscroll, slider-to-slider sync, per-slide dynamic styling, gallery mode).

This is the core slider element — the nestable container that actually holds and moves slides. It's one of three related BricksExtras elements:
- **`xproslider`** (this skill) — the slider itself
- **`xproslidercontrol`** — external nav/progress/counter/dots that target a slider from outside. See skill `xproslidercontrol`.
- **`xproslidergallery`** — populates a slider's slides dynamically from a WP image gallery, nested inside the slider. See skill `xproslidergallery`.

Load the relevant sibling skill(s) alongside this one whenever the task involves external controls or gallery-mode image population — this skill covers the slider element's own mechanics only.

If the task is scoped to a specific existing page (the user names one, or you already have its `postId`), read that page's element tree first and copy the nearest existing `xproslider` pattern rather than starting from scratch — there's no site-wide "find every page using this element" ability in this environment, so this only applies when a page is already in scope.

**Nestable:** Pro Slider supports nested children. You can include slide blocks directly in the children array when creating the slider.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xproslider.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xproslider` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Slide/behavior settings

This element wraps Splide (https://splidejs.com/guides/options/ for the full upstream option list), but only exposes a **subset** of Splide's options, and **our setting names frequently differ from Splide's own key names.** Don't assume a Splide option name works verbatim as a setting key — check against the live schema (or the table below) first.

| Splide option | Our setting(s) | Notes |
|---|---|---|
| `type` | `type` | `slide` / `loop` / `fade` |
| `rewind` | `rewind` | mutually exclusive with `type: loop` — see gotchas below |
| `rewindByDrag` | `rewindByDrag` | |
| `speed` | `speed` | transition speed, ms |
| `rewindSpeed` | `rewindSpeed` | separate speed just for the rewind snap-back |
| `height` | `height` | slider/track height |
| `fixedWidth` / `fixedHeight` | `fixedWidth` / `fixedHeight` | |
| `autoWidth` / `autoHeight` | `autoWidth` / `autoHeight` | |
| `start` | `start` | starting slide index |
| `perPage` | `perPage` | |
| `perMove` | `perMove` | |
| `focus` | `focus` | plain text control — which slide position counts as "active" when `perPage > 1` (e.g. `"center"` keeps the active slide centered as you page through, instead of snapping by whole pages of `perPage`) |
| `gap` | `gap` | |
| `padding` | `padding` ("Edge padding") | reveals part of the previous/next slide by adding padding to the Splide track itself — not CSS padding on the slide content. Number + units control, breakpoint-responsive, no CSS mapping (raw Splide option), disabled (`required: type != fade`) on a fade slider since there's no track to pad. |
| — (padding on the slide's own content) | `slidePadding` | **not a Splide option at all** — this is our own CSS-mapped control (`padding` on `.x-slider_slide`), for spacing a slide's content in from its own edges. Don't confuse this with `padding` above — they target completely different things despite the similar name. |
| `arrows` | `arrows` | select, string `"true"`/`"false"` — see gotchas below |
| `pagination` | `pagination` | checkbox |
| `easing` | `easing` | |
| `drag` | `drag` | |
| `snap` | `snap` | |
| `dragMinThreshold` | `dragMinThreshold` | |
| `flickPower` / `flickMaxPages` | `flickPower` / `flickMaxPages` | |
| `autoplay` (boolean) + `interval` | `autoplayscroll` (select: `autoplay` / `autoscroll` / `none`) + `interval` | Splide's single boolean `autoplay` is split into a 3-way mode select here — see the two-modes gotcha below for `autoplay` vs `autoscroll` |
| `pauseOnHover` / `pauseOnFocus` | `pauseOnHover` / `pauseOnFocus` | |
| `lazyLoad` | `lazyLoad` | |
| `preloadPages` | `preloadPages` | |
| `keyboard` | `keyboard` | |
| `wheel` / `wheelSleep` / `releaseWheel` | `wheel` / `wheelSleep` / `releaseWheel` | |
| `direction` | `direction` | includes `ttb` — see vertical-slider gotcha below |
| `isNavigation` | `isNavigation` | plus our own `syncSelector`/`componentScope` for the sync target — see slider-to-slider sync section below |
| `trimSpace` | `trimSpace` | |
| `omitEnd` | `omitEnd` | |
| `role` / `label` / `labelledby` / `heightRatio` / `clones` / `cloneStatus` / `paginationKeyboard` / `paginationDirection` / `easingFunc` / `noDrag` / `arrowPath` / `resetProgress` / `wheelMinThreshold` / `cover` / `slideFocus` / `focusableNodes` / `updateOnMove` / `mediaQuery` | **not exposed** | present in upstream Splide, no corresponding control on this element |

Additionally, `edgeEffect` (fade mask at loop seams, gated to `type: loop`) and `galleryMode` are this plugin's own additions, not Splide options. All of the above are plain settings with no CSS mapping — set them directly on the element's `settings` object. Only `imageWidth`/`imageForceWidth` and `slidePadding` are CSS-mapped among this group (see the `imageWidth`/`imageForceWidth` gotcha below).

---

## Slide population: three methods, pick one

### 1. Manual slides (default)

Each slide is a **`block`** element nested directly inside the slider, carrying two hidden CSS classes that make Splide and BricksExtras recognize it as a slide:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block",
  "label": "Slide",
  "settings": {
    "_hidden": { "_cssClasses": "x-slider_slide splide__slide" }
  },
  "children": [
    { "name": "heading", "settings": { "text": "Slide" } }
  ]
}
```

**These classes are not optional styling — they're the mechanism that makes the element function as a slide at all.** Without both, Splide won't treat the block as part of the track, and this element's own slide-targeted CSS (`fixedHeight`, `slideBorder`, `slideBackground`, etc., all selecting `.x-slider_slide`) won't apply. Any content can go inside the block — the block itself is just the required slide wrapper. Add one such block per slide, each as a direct child of `xproslider`.

**Query loop for dynamic slides:** To populate slides dynamically from posts/terms/users, put **both** `hasLoop: true` and `query` **on the slide block itself** — this is standard Bricks query-loop syntax (any looping element carries its own `hasLoop`/`query`, same as a query-looped `div` or `container` anywhere else in Bricks). `hasLoop` does **not** go on the parent `xproslider` — the slider itself is never a looping element, it's just the container. `hasLoop` on the slider is silently ignored (Splide receives no query results, so only the one literal block you wrote renders as a single slide); moving `hasLoop` onto the block is what actually makes it loop.

**Slides sourced from a dynamic-data image field (e.g. an ACF Gallery field) are a different `query.objectType`, not the generic `post`/`term`/`array` loop.** Don't reach for a generic array loop (`objectType: "array"` + `arrayEditor` on a raw ACF tag) here — BricksExtras registers a purpose-built loop type for exactly this case. Load `bricksextras-query-loop-extras` and use its `gallery` sub-type (`query: {"objectType": "queryLoopExtras"}`, `extrasQuery: "gallery"`, `x_gallery_data: "<dynamic tag>"`) on the slide block instead — see that skill for the full shape, including how the image element inside consumes each iteration (`{post_id}`, not an array-loop tag).

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": { 
    "perPage": 1, 
    "autoplay": true 
  },
  "children": [
    {
      "name": "block",
      "settings": {
        "_hidden": { "_cssClasses": "x-slider_slide splide__slide" },
        "hasLoop": true,
        "query": {
          "objectType": "post",
          "post_type": ["post"],
          "posts_per_page": 5
        }
      },
      "children": [
        { "name": "heading", "settings": { "text": "{post_title}" } }
      ]
    }
  ]
}
```

Each loop iteration creates one slide with the current post's dynamic data available via `{post_title}`, `{post_excerpt}`, etc.

### 2. Gallery mode

Enable `galleryMode: true` on this element's settings, then nest a single `xproslidergallery` element as a **direct child**, in place of manual blocks — not alongside them. **Full details, the verified `items` value shape, and gotchas live in the Pro Slider Gallery skill** (`xproslidergallery`) — load it whenever gallery mode is in play. One thing worth knowing here: gallery-generated slides get the hidden slide classes above applied automatically — you never set them manually in gallery mode.

**Gotcha:** `galleryMode` (on this element) and the presence of a nested Gallery child are two separate requirements — both must be true or the gallery is inert.

**Gallery mode best practices:**

For a main gallery with synced thumbnail slider (common pattern):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": {
    "_cssId": "main-gallery",
    "galleryMode": true,
    "type": "fade",
    "perPage": "1",
    "fixedHeight": "600px",
    "imageWidth": true,
    "imageForceWidth": true,
    "arrows": "false"
  },
  "children": [
    {
      "name": "xproslidergallery",
      "settings": {
        "items": { "images": [...], "size": "1536x1536" },
        "objectFit": "cover"
      }
    }
  ]
}
```

Thumbnail slider synced to main:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": {
    "galleryMode": true,
    "type": "slide",
    "isNavigation": true,
    "syncSelector": "#main-gallery",
    "fixedWidth": "20rem",
    "fixedHeight": "20rem",
    "gap": "1rem",
    "imageWidth": true,
    "imageForceWidth": true,
    "preloadPages": "5",
    "arrows": "false",
    "pagination": true
  },
  "children": [
    {
      "name": "xproslidergallery",
      "settings": {
        "items": { "images": [...], "size": "large" },
        "objectFit": "cover",
        "lazyLoadSupport": "splide"
      }
    }
  ]
}
```

**Key settings explained:**
- `fixedHeight` sets individual slide height (Splide option), not container height
- `imageWidth: true` + `imageForceWidth: true` makes images fill slide dimensions (both required, in that order)
- `preloadPages` should match roughly how many slides are visible (e.g., "5" for ~5 visible thumbnails) to avoid lazy loading visible content
- `perPage` values are strings (e.g., `"1"`), not numbers - Bricks outputs all settings as strings
- Use `fixedWidth` + `fixedHeight` for thumbnails instead of `perPage` for consistent sizing

### 3. Code element (last resort, when 1 and 2 can't reach the needed content)

If the slide content can't be produced by a manual block, a query loop, or gallery mode, a **`code`** element can populate the slides directly — reusing the same list-suppression mechanism gallery mode uses, just with hand-written PHP output instead of `xproslidergallery`.

1. Set `galleryMode: true` on the parent `xproslider` — same flag as method 2, telling the slider not to inject its own `.splide__list` wrapper because a child is supplying one.
2. Add a single `code` element as the slider's direct child.
3. Give that code element's root the **`splide__list`** class (via `_cssClasses` or a global class) — Splide only checks for the class, not the tag, so a `code` element's `div` root works the same as `xproslidergallery`'s `ul`.
4. The PHP echoes one `<div class="x-slider_slide splide__slide">...</div>` per slide — same two required classes as manual slides — with whatever content is needed inside each:

```html
<div class="x-slider_slide splide__slide">
  ...
</div>
<div class="x-slider_slide splide__slide">
  ...
</div>
```

This is a last resort, not a first choice — reach for it only when methods 1–2 genuinely can't get the needed data/content, since it requires the code-execution capability and is harder to audit than a query loop.

---

## Rendered DOM structure (manual slides) — the track/list wrapper is auto-injected

Manual `block` slides are written as **direct children of `xproslider`** (see above) — you never build a `.splide__track`/`.splide__list` wrapper yourself. BricksExtras' render layer injects that wrapper around whatever children you give it:

```html
<div id="brxe-89igab" class="brxe-xproslider ... splide x-slider" data-x-id="89igab" data-x-slider="{...rawConfig JSON...}">
  <div class="splide__track x-splide__track">
    <div class="splide__list">
      <div id="brxe-6dhstp" class="brxe-block x-slider_slide splide__slide bricks-lazy-hidden">...</div>
      <div id="brxe-c6xb6o" class="brxe-block x-slider_slide splide__slide bricks-lazy-hidden">...</div>
      <!-- one such div per manual slide block -->
    </div>
  </div>
</div>
```

Only the slide `block`s themselves belong as direct children in the JSON you write; the wrapper divs are not something you author.

This wrapper is why gallery mode (see the Pro Slider Gallery skill) has to actively suppress the slider's own list wrapper — `xproslidergallery` supplies its own `<ul class="... splide__list ...">` instead, so both can't be present at once without duplicating the list structure.

---

## Slider-to-slider sync (built-in, this element's own mechanism)

Settings: `isNavigation` (checkbox) + `syncSelector` (text: CSS selector, or inside a query loop, the other slider's element ID) + `componentScope` (select, required only when `isNavigation` is enabled; defaults to false — **send the string `"true"`/`"false"`, not a boolean**, same as `xproslidercontrol`'s field of the same name; a boolean is silently ignored).

Use this when **one full slider** (its own real slides — typically a thumbnail strip) should drive **another full slider**. Enabling `isNavigation` on slider A with `syncSelector` pointing at slider B makes slider A's slides clickable to navigate slider B. Both remain independent `xproslider` instances.

**This is different from `xproslidercontrol`'s own targeting fields** (`slider`/`sliderSelector`), which are for non-slider utility widgets (a button, a bar, a counter) with no slide content of their own — see skill `xproslidercontrol` for that mechanism. Rule of thumb: if the "other side" has its own slides, use this element's `isNavigation`; if it's just a button/bar/counter/dots, that's the Control element's job instead.

### Targeting `syncSelector` — prefer a class over `_cssId`

Bricks elements do not expose their internal element ID as the DOM `id` by default — they render as `id="brxe-{id}"` unless the standard Bricks `_cssId` field is set, which outputs the raw string with no `brxe-` prefix. `syncSelector` (and an `xproslidercontrol`'s `sliderSelector`) is a plain CSS selector, so `#someId` only resolves if that exact string is set via `_cssId` on this slider — omitting `_cssId` and using `#{elementId}` does not work, since the rendered id is `brxe-{elementId}`, not `{elementId}`.

**But prefer a class selector (`_cssClasses` + `.thatClass`) over an id selector for this, per `bricksextras-start-here`'s "Targeting another element" section.** `_cssId` is baked into a component's definition — if this slider (or the element syncing to it) ever ends up inside a component, every instance of that component renders the identical literal DOM id, and `syncSelector`/`sliderSelector` targeting silently resolves to the *first* instance's slider on the page, not its own. A class doesn't have that failure mode. If the synced pair does live inside the same component, also set `componentScope: "true"` (the string) on whichever element has that option — present on both this element's own `isNavigation` settings and on `xproslidercontrol` — so the class lookup stays scoped to that component instance.

### Sizing a synced thumbnail + main pair when the user gives no spec

When the user asks for a thumbnail strip synced to a main slider and doesn't specify sizing, default to a **CSS Grid row with the size ratio baked into the columns**:

- Wrap both sliders in one `block`/`container` with `_display: "grid"` and `_gridTemplateColumns` set to a ratio like `"4fr 1fr"` (main : thumbnail). **Always wrap each fraction in `minmax(0, ...)`** (e.g. `"minmax(0, 4fr) minmax(0, 1fr)"`), on every breakpoint override too — a bare `fr` track has an implicit `min-width: auto`, which lets the slider's own intrinsic content size force that grid column (and the whole page) wider than the viewport, causing horizontal scroll. This applies to any `xproslider` sitting in a grid column, not just this synced pair pattern.
- Give the main slider's slide `image` an `_aspectRatio` — `"1"` (square) is a safe default absent other direction. Its rendered height then follows from its own column width: `mainHeight = mainWidth / mainRatio`.
- Give the **thumbnail slider itself** `direction: "ttb"` + a matching `_aspectRatio`, chosen algebraically so its height equals the main slider's at any viewport width: `thumbAspectRatio = thumbColumnFraction / (mainColumnFraction × mainAspectRatio)`. For `4fr 1fr` columns with a square (`ratio 1`) main slider: `1 / (4 × 1) = .25`.
- A vertical slider (`direction: "ttb"`) needs either the Splide `height` option or `autoHeight: true` set to have a defined size — **prefer `autoHeight: true` paired with the `_aspectRatio` above** over a fixed pixel `height`, since the aspect-ratio keeps the thumbnail column's height correctly tied to the main slider's height as the layout's width changes across breakpoints, where a fixed `height` wouldn't.
- Set `preloadPages` to roughly the number of thumbnails visible at once on desktop (e.g. `4` if 4 show without scrolling), so the initially-visible thumbnails aren't lazy-load-blank on first paint.
- For a horizontal thumbnail strip variant at a smaller breakpoint, reuse the same `xproslider` instance with breakpoint-suffixed setting overrides (e.g. `"direction:mobile_portrait": "ltr"`, `"perPage:mobile_portrait": "4"`, `"_aspectRatio:mobile_portrait": "unset"`) alongside a matching `"_gridTemplateColumns:{breakpoint}": "minmax(0, 1fr)"` on the wrapping grid to stack the columns, instead of building a second parallel element tree.
- **Put `pagination`/`arrows` on the main slider only, not the thumbnail slider.** The thumbnail strip's slides already act as navigation (that's what `isNavigation` is for) — dots or arrows on the thumbnail slider itself are redundant next to the visible thumbnails. Default: `pagination: true`/`arrows: "true"` on the main slider, `pagination: false` and `arrows: "false"` on the thumbnail slider unless the user asks otherwise.
- **Always set `preloadPages` explicitly — the schema's own placeholder default is `1`, and a "page" is not "everything currently visible."** `preloadPages` controls how many *pages* (groups of `perPage` slides) around the active one get eagerly loaded — it is not a count of visible slides. When `perPage` is left unset and sizing comes from `fixedWidth`/`fixedHeight` instead (the normal way to build a thumbnail strip — see above), Splide's internal page size defaults to **1 slide per page**, regardless of how many slides the fixed width actually shows at once. That means the default `preloadPages: 1` only eagerly loads 2 slides total (active + 1 neighbor page) even when 6–7 thumbnails are visibly sitting in the strip on page load — the rest sit as blank/spinner placeholders until Splide advances far enough to reach them.

  The fix: work out how many slides are actually visible at once on desktop at the fixed-width/gap values you built, and set `preloadPages` to that count (e.g. `"7"` for ~7 visible fixed-width thumbnails) so the whole visible strip loads on first paint instead of only the active slide's immediate neighbor. When the container width and the slide's `fixedWidth`/`gap` are all known values you set yourself, the visible count is just arithmetic (container width ÷ (fixedWidth + gap)) — no need to screenshot-check something you can compute directly. Only fall back to a real screenshot right after navigation when the container width is itself unknown or responsive (e.g. inherited from a parent you didn't size, or intentionally fluid) so the visible count can't be reliably worked out in advance — a settings read-back won't reveal a wrong `preloadPages` value either way, only the rendered images will.

---

## Gotchas worth knowing before configuring

- **Slide content centers both axes by default.** `slideAlignHorizontal`/`slideAlignVertical` both default to `center` in the schema, so a slide's children sit centered inside it out of the box — the same thing a human gets adding this element in the builder. That's a sensible starting point, not a rule to preserve — every slider's actual layout need is different (left-aligned text over an image, content pinned to the bottom, etc.), so set these explicitly whenever the design calls for something other than centered.
- **`type: fade` disables slide-geometry controls.** `perPage`, `fixedWidth`, `gap`, `perMove` all require `type != fade` — fade shows one full-width/full-height slide at a time by definition, so these silently don't apply.
- **`type: loop` and `rewind` are mutually exclusive strategies.** `rewind` (snap back to start at the end) only applies when `type != loop`. `edgeEffect` (fade mask at the loop seams) only applies when `type = loop`. Don't combine loop mode with rewind — loop already handles wrap-around natively.
- **`arrows` and `pagination` default to visible ("on").** If a design calls for neither, both need to be explicitly turned off — they aren't opt-in. `pagination` is a checkbox-type control — send `"pagination": false` to turn it off (omitting the key also works, since absent and explicit `false` are both treated as off). `arrows` is a select with string `"true"`/`"false"` options and turns off when sent `"false"`.
- **Slides ship with real, non-zero default CSS padding — for a full-bleed image slide (image meant to fill 100% of the slide box), `slidePadding` must be explicitly zeroed.** Unlike most of this element's controls, this isn't a case of an unapplied schema placeholder — the plugin's own base stylesheet puts real padding on `.x-slider_slide` regardless of what `slidePadding` shows as its placeholder. Leaving `slidePadding` unset on a slider built for edge-to-edge images (a gallery-mode-style main slider, a thumbnail strip, any slide whose content is a single cover image) renders visible gutters around each image. Fix: `"slidePadding": {"top": "0", "right": "0", "bottom": "0", "left": "0"}` on the slider.
- **A Bricks core `image` element's `caption` control effectively defaults to showing the attachment caption, not to "no caption" — for an image meant to be undecorated (a thumbnail, a plain gallery image, anything not intentionally using the caption as an overlay), `caption` must be explicitly set to `"none"`.** This is different from the usual "schema placeholder is UI-only, omitting the key means off" rule elsewhere in BricksExtras — `caption`'s schema placeholder is `"Attachment"`, and an `image` element inside a slide with no `caption` key at all renders the WordPress attachment caption as a `<figcaption>`. Only omit this (or set `"caption": "attachment"`/`"custom"` deliberately) when a caption overlay is actually wanted — see the caption pattern earlier in this skill for that case.
- **For a gallery/photo slider, set an explicit `_aspectRatio` on the slide's `image` element itself, not just on the slider or slide.** The image element's own `_aspectRatio` control (paired with `_objectFit: "cover"` and `_width: "100%"`, no `_height` needed) keeps the image's proportions consistent across every breakpoint regardless of how the slider or slide box happens to be sized at that breakpoint — relying only on `_height: 100%` to fill a parent whose own height changes per breakpoint (or per `type: fade` vs `type: slide`) can let the image's rendered proportions drift. This is the default to reach for on any image-gallery slider unless the request specifically calls for images at their natural/unconstrained aspect ratio instead.
- **`.splide__arrows` and `.splide__pagination` are both `position: absolute` by default, anchored to the slider's own box — not flowing elements.** `margin` does nothing to reposition either. This element exposes dedicated positioning controls for both, so **reach for these before writing custom CSS**: `paginationTop`/`paginationBottom`/`paginationLeft`/`paginationRight` for the dots, and `nextArrowTop`/`nextArrowBottom`/`nextArrowLeft`/`nextArrowRight`/`nextArrowMargin` + the matching `prevArrow*` set for the arrows (each maps straight to the real CSS property on the actual `.splide__pagination`/`.splide__arrow--next`/`.splide__arrow--prev` selectors). This applies generally, not just when combined with other effects — e.g. an active-slide scale/transform (see the Splide `transform` constraints section below) can grow the active slide into the pagination's absolute-positioned box regardless of cause, and the fix is the same: push `paginationBottom` (or the relevant offset) far enough to clear it.

  When the desired placement is outside the slider's own bounding box entirely — a separate layout region, a fixed external toolbar, dots placed beside unrelated content — the built-in offset controls won't reach that far cleanly. Reach for `xproslidercontrol` instead: it's a standalone element built to target and drive a slider from anywhere on the page, with normal in-flow placement of its own. See skill `xproslidercontrol`.
- **Autoplay has two distinct modes that don't share settings.** `autoplayscroll: autoplay` (discrete interval-based advance) uses `interval` (ms) and `autoplayPaused`. `autoplayscroll: autoscroll` (continuous marquee-style scroll) uses `autoScrollSpeed` instead — setting `interval` while in `autoscroll` mode (or vice versa) has no effect.
- **`imageWidth` and `imageForceWidth` are two separate checkboxes whose CSS collides — order matters.** Despite the names, `imageWidth`'s actual label is *"Force images to be 100% slide **height**"* (sets `width: auto; height: 100%` on `.x-slider_slide img` / `.x-slider_slide-image`) and `imageForceWidth`'s label is *"Force images to be 100% slide **width**"* (sets `width: 100%` on the same two selectors). Both target the `width` property on the same selectors, so with equal CSS specificity, **whichever control's rule is emitted later in the compiled stylesheet wins the `width` value** — the two don't compose automatically. To make an image fill **both** dimensions of the slide (the usual goal, especially in gallery mode — see the Gallery skill), both checkboxes must be enabled, **with `imageWidth` set before `imageForceWidth`** in the settings write, so `imageForceWidth`'s `width: 100%` is the later, winning rule while `imageWidth`'s `height: 100%` (untouched by the other control) still applies. Sending `{"imageWidth": true, "imageForceWidth": true}` in that key order round-trips correctly; reversing the order — or setting only one — produces the wrong final `width` value even though both checkboxes read back as `true`. A settings read-back showing both as `true` is **not** sufficient to confirm this is working — the actual rendered `width`/`height` on `.x-slider_slide img` needs checking, since the bug is in cascade order, not in whether the values saved.
- **Vertical sliders (`direction: ttb`) have a hard requirement: the slider box itself needs a real, defined height — nothing about `direction: ttb` gives it one on its own.** A `direction: ttb` slider with neither `perPage` nor `autoHeight` set (e.g. only `height` on its own) has no way to size its slides — this is a silent failure, not an error, so it's easy to ship a vertical slider that looks fine in the settings read-back but has zero visible extent or misshapen slides on the frontend.

  The requirement is "give the box a real height" — it's not specifically that the literal `height` setting must be set. There are two ways to satisfy it:
  - **A literal `height`** (e.g. `"600px"`) — the simplest option, but fixed in pixels: it won't adapt as the slider's own width changes across breakpoints (e.g. a thumbnail column that gets narrower on mobile).
  - **An `_aspectRatio` on the slider itself** (see the grid-ratio sizing pattern above) — the height is then derived from whatever width the slider actually renders at, so it stays responsive as the layout's width changes. **This is usually the better choice for a thumbnail strip synced to a main slider**, since it keeps the thumbnail column's height correctly tied to the main slider's height at every viewport width, where a fixed pixel `height` wouldn't.

  Either source of height then needs to be paired with a slide-count control — **prefer `autoHeight` over `perPage` by default**:
  - **`autoHeight`** (preferred): each slide keeps its own natural height instead of an equal split. It still needs the same real box height (literal `height` or `_aspectRatio`) on the slider to compute against — it does not create height on its own.
  - **`perPage`** (fixed number of equal-height slides visible at once): Splide divides the height (from whichever source above) by `perPage` to calculate individual slide heights. Reach for this only when the design specifically wants that even-division behavior; `autoHeight` is the safer default.

  **If the slides themselves need to be equal height, don't rely on `perPage`'s division math to force it — set the height explicitly on the slide content instead**, either:
  - a fixed `_height` (or `min-height`) on the slide's own `block`, or
  - `_aspectRatio` on the `image` element inside each slide (paired with `_objectFit: "cover"`, per the earlier image-gallery gotcha) — the better choice when slides are photos of varying source dimensions, since it stays consistent without a hardcoded pixel value.

  Most slider options come directly from Splide: see https://splidejs.com/guides/options/ for detailed option documentation.
  - Example vertical thumbnail slider (autoHeight + literal height): `{"direction": "ttb", "height": "600px", "autoHeight": true, "gap": "1rem"}`.
  - Example vertical thumbnail slider (autoHeight + aspect ratio, preferred for a responsive thumbnail strip): `{"direction": "ttb", "autoHeight": true, "_aspectRatio": ".25", "gap": "1rem"}` - no literal `height` at all; the slider's box height is derived from its own rendered width × the ratio, so it tracks a synced main slider's height across breakpoints automatically.
  - Example forcing equal-height slides under `autoHeight` (gallery mode): give each `xproslidergallery`-generated image an `aspectRatio: "1"` + `objectFit: "cover"` on the Gallery element's own settings, rather than switching to `perPage`.

---

## Splide's own constraints on `transform` — applies whenever a slide needs a transform effect (scale-on-active, hover-lift, tilt, etc.)

These come from Splide itself, not from this plugin.

1. **Never put a `transform` on the slider element itself, or on any ancestor of it.** Splide's own drag/swipe/position math depends on reading real, untransformed layout geometry (bounding rects, offsets). A `transform` anywhere upstream of the slider distorts what Splide measures and breaks that math. This rules out things like rotating an entire slider to fake a diagonal layout.
2. **Never put a `transform` directly on the slide element** (the `block` carrying `x-slider_slide splide__slide`). Splide's own track uses `transform` for positioning/movement, and a transform on the slide element conflicts with it.

**The fix for both: nest an inner `block` inside the slide, and put the transform — plus whatever styling needs to visually represent the "card" (background, border, box-shadow, etc.) — on that inner block instead.** The outer slide `block` keeps only its required Splide classes and no transform of its own; Splide owns it fully. Since the inner block now carries the visual weight, **explicitly set the outer slide's own padding to `0`** — slides have default padding in the CSS that needs overriding once an inner wrapper takes over, or an unwanted gap shows up around the inner card.

Example: a "scale up the active slide" effect. `.is-active` (Splide's own class marking the current slide) lands on the *outer* slide element, not the inner wrapper — so the CSS has to bridge that with a descendant/child combinator, not a compound class on one element:

```css
/* WRONG — .is-active and the card styling aren't on the same element */
.query-slide-card.is-active { transform: scale(1.08); }

/* RIGHT — .is-active is on the outer .splide__slide, the class to transform is on its inner child */
.splide__slide.is-active > .query-slide-card { transform: scale(1.08); }
```

**Default (unless told otherwise) to matching the transform's transition duration to the slider's own `speed` setting** (converted ms → s) — e.g. `speed: 500` on the slider pairs with `transition: transform .5s ease` on the inner block. This is a starting-point preference for a synchronized feel, not a rule: plenty of designs intentionally want the effect faster, slower, or otherwise decoupled from the slide transition. Follow an explicit request for different timing rather than defaulting to matching `speed`.

### A related, layout-dependent issue: enlarged slide content can get clipped by the track

If a transform effect (or anything else) makes a slide's content larger than its track allocation — the scale-on-active example above included, since `scale(1.08)` makes the active card briefly bigger than its box — the slider track's own `overflow: hidden` (needed for it to function as a slider at all) can clip the excess.

This is **a known constraint to watch for, not a single fix to apply by default** — the right response depends on the specific design/layout, and forcing one particular solution can itself change the layout in ways that aren't always wanted. One available option: `trackOverflow: "visible"` on the slider (targets `.splide__track`'s own `overflow`; options `unset`/`hidden`/`visible`) removes the clipping. The trade-off: this also makes slides sitting outside the visible viewport visible, so it typically needs an `_overflow: "hidden"` on some wrapping element to contain that — but that wrapping element has to be sized to just the slider's own visible area, not the full page width. **This means the slider can no longer be a true full-bleed/full-width layout** (direct child of the section, no container) the way the "fade slider + synced thumbnail strip" pattern below is — it needs a purpose-sized wrapping container/div around it instead. Don't reach for this reflexively; confirm it's actually compatible with the design before applying it, and consider whether reducing the transform's scale/size, or accepting a slight clip, might fit the layout's constraints better.

---

## Pattern: grid-ratio vertical thumbnail strip synced to a square main slider (default go-to)

The default pattern for a thumbnail slider synced to a main image/gallery slider, absent other spec — the grid ratio and matching `_aspectRatio` keep both sliders the same height at any viewport width, with no hardcoded pixel heights on either.

Wrapping grid (one `block`/`container`, ratio baked into the columns):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block",
  "settings": {
    "_display": "grid",
    "_gridTemplateColumns": "minmax(0, 4fr) minmax(0, 1fr)",
    "_gridGap": "1rem",
    "_gridTemplateColumns:mobile_portrait": "minmax(0, 1fr)"
  },
  "children": [
    { "name": "block", "children": ["/* main xproslider goes here */"] },
    { "name": "block", "children": ["/* thumbnail xproslider goes here */"] }
  ]
}
```

Main slider (square, drives the whole layout's height via its own column width):

```json
{
  "name": "xproslider",
  "settings": {
    "_cssGlobalClasses": ["<id of a global class named e.g. gallery-main-slider>"],
    "type": "fade",
    "perPage": "1",
    "arrows": "true",
    "slidePadding": {"top": "0", "right": "0", "bottom": "0", "left": "0"}
  },
  "children": [
    {
      "name": "block",
      "settings": {
        "_cssClasses": "x-slider_slide splide__slide",
        "hasLoop": true,
        "query": {"objectType": "queryLoopExtras"},
        "extrasQuery": "gallery",
        "x_gallery_data": "{acf_gallery}"
      },
      "children": [
        { "name": "image", "settings": {"image": {"useDynamicData": "{post_id}"}, "caption": "none", "_objectFit": "cover", "_aspectRatio": "1"} }
      ]
    }
  ]
}
```

Thumbnail slider (vertical, `_aspectRatio` chosen algebraically — `thumbColumnFraction ÷ (mainColumnFraction × mainAspectRatio)` = `1 ÷ (4 × 1)` = `.25` — so its height cancels out to match the main slider's height at any viewport width):

```json
{
  "name": "xproslider",
  "settings": {
    "direction": "ttb",
    "autoHeight": true,
    "_aspectRatio": ".25",
    "gap": "1rem",
    "arrows": "false",
    "isNavigation": true,
    "syncSelector": ".gallery-main-slider",
    "componentScope": "false",
    "slidePadding": {"top": "0", "right": "0", "bottom": "0", "left": "0"},
    "direction:mobile_portrait": "ltr",
    "_aspectRatio:mobile_portrait": "unset",
    "perPage:mobile_portrait": "4"
  },
  "children": [
    {
      "name": "block",
      "settings": {
        "_cssClasses": "x-slider_slide splide__slide",
        "hasLoop": true,
        "query": {"objectType": "queryLoopExtras"},
        "extrasQuery": "gallery",
        "x_gallery_data": "{acf_gallery}"
      },
      "children": [
        { "name": "image", "settings": {"image": {"useDynamicData": "{post_id}"}, "caption": "none", "_objectFit": "cover", "_aspectRatio": "1"} }
      ]
    }
  ]
}
```

Note the `syncSelector` targets `.gallery-main-slider` — a real global class attached to the main slider via `_cssGlobalClasses`, not a plain local `_cssClasses` string, per `bricksextras-start-here`'s "classes added for styling/targeting: use a real global class" guidance. At `mobile_portrait`, the grid collapses to `minmax(0, 1fr)` (single column, stacked) and the same thumbnail slider instance flips to a horizontal row (`direction: ltr`, `perPage: 4`) instead of needing a second parallel element tree — see the sizing section above.

---

## Pattern: fade slider + synced thumbnail strip (horizontal, fixed-pixel alternative)

Prefer the grid-ratio pattern above for a new build; reach for this one when the thumbnail strip needs fixed-pixel sizing instead of a viewport-proportional ratio (e.g. a horizontal-only design with no vertical variant).

Main slider (full-bleed section, direct child, no container wrapper):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": {
    "_cssId": "mainSlider",
    "type": "fade",
    "height": "70vh",
    "arrows": "true",
    "pagination": false
  }
}
```

Thumbnail slider (separate section/container below, its own manual slide blocks sized as thumbnails):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslider.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xproslider",
  "settings": {
    "type": "slide",
    "perPage": 4,
    "fixedWidth": "160px",
    "fixedHeight": "90px",
    "gap": "10px",
    "arrows": "false",
    "pagination": false,
    "isNavigation": true,
    "syncSelector": "#mainSlider",
    "componentScope": "false"
  }
}
```

On render, Splide drives the sync itself — the thumbnail slider's wrapper picks up an extra `splide--nav` class it wasn't given explicitly, and each thumbnail slide automatically gets `role="button"` plus `aria-controls` pointing at the matching main slide ID, with `aria-current="true"` on the active one.

**The same `isNavigation`/`syncSelector` sync also works combined with gallery mode on both sliders** (rather than manual slide blocks) — two `xproslider` instances, both `galleryMode: true` with their own `xproslidergallery` child (same image set on both), main slider `type: fade` with `_cssId`, thumbnail slider `type: slide` + `isNavigation`/`syncSelector` targeting it.

---

## Workflow for a new slider on any site

1. **Confirm the plugin is active and get the current schema.** Check the bundled schema first per `bricksextras-element-schemas`; only if the bundle is missing or stale, call whatever live schema ability the current MCP connection exposes (e.g. `ability_name: "bricks/get-element-schema"`, `parameters: {"elementName": "xproslider"}` on the native Bricks MCP). A live-call error or empty result means either the element name is wrong or BricksExtras isn't active — cross-check against that connection's element-listing ability (e.g. `bricks/list-element-types` on the native Bricks MCP) before assuming from memory.
2. **If a specific page is already in scope**, read its element tree and copy the nearest existing `xproslider` pattern rather than building from scratch.
3. **Decide the slide-population method.** Manual `block` slides (full design freedom), a query loop on the slide block (dynamic from posts/terms/users), or gallery mode (load the Gallery skill for the verified `items` shape) for straightforward image sliders. Fall back to a `code` element (see method 3 above) only when none of those can reach the needed content.
4. **Build the settings JSON** using the live schema — pay attention to `type` (slide/loop/fade) since it gates several other fields (see gotchas above). The slider and all its slides can be written in a single nested-JSON call.
5. **Set behavior settings (perPage, gap, height, direction, type, arrows, pagination, sync, autoplay, galleryMode, etc.) directly on the element** — plain options, no CSS/class involved.
6. **If wiring a sync or a control to this slider, target it by class, not `_cssId`** — set a class via `_cssClasses` and point `syncSelector`/`sliderSelector` at `.thatClass`. See `bricksextras-start-here`'s "Targeting another element" section for why: `_cssId` collides across every instance of a component. Add `componentScope: "true"` (the string, not the boolean) on the targeting element if both it and this slider live inside the same component.
7. **If adding external nav/progress/counter/dots, load the Control skill** (`bricksextras-xproslidercontrol`) — those elements are placed outside this slider as siblings, never nested inside it.
8. **If syncing two full sliders** (e.g. thumbnail-drives-main), use this element's own `isNavigation`/`syncSelector` on the secondary (navigation) slider.
9. **After building, verify against rendered output**, not just the settings re-read back — inspect the `data-x-slider` JSON attribute and the actual DOM (slide count, classes, ids).

---

## Never do

- Do not omit the required CSS classes `x-slider_slide splide__slide` on manual slide blocks — without both, slides won't function.
- Do not use `#{elementId}` as a sync selector without setting `_cssId` — the rendered DOM id is `brxe-{elementId}`, not `{elementId}`. But prefer a class selector over `_cssId` in the first place — see `bricksextras-start-here`.
- Do not combine `type: loop` with `rewind` — they're mutually exclusive wrap-around strategies.
- Do not set `perPage`, `gap`, or `fixedWidth` when `type: fade` — fade mode ignores slide-geometry controls.
- Do not assume `arrows` is a checkbox — it's a select with string values `"true"`/`"false"`.
- Do not manually build `.splide__track`/`.splide__list` wrappers — BricksExtras injects them automatically.
- Do not put `hasLoop` on the parent `xproslider` when building a query-loop slider — it belongs on the slide `block` itself, alongside `query`. `hasLoop` on the slider is silently ignored.
- Do not default a synced thumbnail+main slider pair to a percentage-based flex width split when the user hasn't specified sizing — default to a small fixed-size thumbnail strip (fixed width/height, or `autoHeight` for vertical) matched to the main slider's height/width via `_aspectRatio`, not viewport-relative percentages.
- Do not build a `direction: ttb` (vertical) slider without giving the box a real height (literal `height` or `_aspectRatio`) *and* pairing it with `autoHeight` (preferred) or `perPage` — leaving both unset gives slides no defined height and fails silently (settings read-back looks fine; frontend doesn't). Default to `autoHeight: true`, since thumbnail images are rarely uniform aspect ratio and `perPage`'s equal-division math (`height / perPage`) squashes them — and prefer `_aspectRatio` over a literal `height` when the strip is synced to a main slider, so it stays responsive across breakpoints instead of needing a hardcoded pixel value per breakpoint.
- Do not reach for `perPage` just to force equal-height slides under `autoHeight` — set the height explicitly on the slide content instead (a fixed `_height`/`min-height` on the slide's `block`, or `_aspectRatio` + `_objectFit: "cover"` on the image inside).
- Do not leave `preloadPages` unset "because it has a default" — the placeholder default is `1` *page*, and in a `fixedWidth` thumbnail strip (no explicit `perPage`) a page is just 1 slide, so only 2 slides load eagerly no matter how many are visibly sitting in the strip. Work out the real number of slides visible at once at the fixed-width/gap you built and set `preloadPages` to that number explicitly.
- Do not leave `slidePadding` unset when the design needs images/content to fill the slide edge-to-edge — the slide's real default CSS padding (not just a schema placeholder) will leave visible gutters. Set `"slidePadding": {"top": "0", "right": "0", "bottom": "0", "left": "0"}` explicitly for any full-bleed slide.
- Do not add an `image` element inside a slide without deciding `caption` explicitly — omitting the key does not mean "no caption" the way it does for most other controls; it renders the WordPress attachment caption by default. Set `"caption": "none"` unless a caption overlay is actually wanted.

## MCP write notes

- Check the bundled schema first per `bricksextras-element-schemas`; only if the bundle is missing or stale, call whatever live schema ability the current MCP connection exposes — don't rely on memory either way.
- Manual slide blocks require the required Splide classes on the slide `block`'s own `_cssClasses` setting — send `"_cssClasses": "x-slider_slide splide__slide"` directly, not wrapped in `_hidden: { _cssClasses: "..." }`. Some MCP write paths reject the `_hidden` wrapper with `bricks_invalid_param`; the plain top-level `_cssClasses` string is the safer default. These are functional, not styling classes either way.
- Class-based sync/targeting is preferred over `_cssId` — see `bricksextras-start-here`'s "Targeting another element" section. Internal element IDs are not exposed as DOM `id` by default (id-based selectors need `_cssId` set to work at all), but `_cssId` collides across component instances, so a class is the safer default even outside components.
- When verifying, check the rendered `data-x-slider` attribute and DOM structure, not just settings read-back.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real Splide.js instance from `window.xSlider.Instances[dataXId]` (keyed by the `data-x-id` on the slider's `.x-slider` root) in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), and drive it directly via Splide's own API. The instance is registered ~150ms after mount, so wait past that before reading it rather than reading immediately on `DOMContentLoaded`.
