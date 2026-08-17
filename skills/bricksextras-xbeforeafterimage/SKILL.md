---
name: xbeforeafterimage
description: "Use when building or debugging the Before/After Image element (xbeforeafterimage) from BricksExtras: a draggable comparison slider between two images. Covers the required block>image child structure and the iconLeft/iconRight settings having no runtime fallback."
---

**Requires:** BricksExtras 1.7.3+ with xbeforeafterimage element enabled

# BricksExtras: Before/After Image (xbeforeafterimage)

Shipped by the **BricksExtras** plugin. Nestable — requires exactly two children in a specific wrapped structure, not two bare `image` elements.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xbeforeafterimage.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xbeforeafterimage` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: each image needs a `block` wrapper, not a bare `image` child

The element's children must be exactly two `block`s (one "before", one "after"), each containing a single `image` child — not two `image` elements added directly. Both the outer block and the inner image carry required hidden classes the comparison JS depends on.

**This is the complete baseline — including `iconLeft`/`iconRight` in `settings`, which have no runtime fallback (see below). Don't build from a version that drops them, even for a "simple" comparison slider:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xbeforeafterimage.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xbeforeafterimage",
  "settings": {
    "iconLeft": { "library": "themify", "icon": "ti-angle-left" },
    "iconRight": { "library": "themify", "icon": "ti-angle-right" }
  },
  "children": [
    {
      "name": "block",
      "label": "Before Block",
      "settings": {
        "_hidden": { "_cssClasses": "x-before-after-image_block" }
      },
      "children": [
        {
          "name": "image",
          "settings": {
            "image": { "id": 123 },
            "caption": "none",
            "_hidden": { "_cssClasses": "x-before-after-image_image" }
          }
        }
      ]
    },
    {
      "name": "block",
      "label": "After Block",
      "settings": {
        "_hidden": { "_cssClasses": "x-before-after-image_block" }
      },
      "children": [
        {
          "name": "image",
          "settings": {
            "image": { "id": 456 },
            "caption": "none",
            "_hidden": { "_cssClasses": "x-before-after-image_image" }
          }
        }
      ]
    }
  ]
}
```

Bare `image` children (skipping the `block` wrapper) render two static images side by side with no working drag comparison, no error shown.

---

## Critical: `iconLeft`/`iconRight` must be set explicitly, or the drag handle renders with no icon

The drag handle button itself always renders; the icon inside it does not fall back to its schema-declared default when omitted (`empty()` check with no PHP-side default). **Already included in the baseline example above** — set both explicitly, matching the schema's declared defaults unless told otherwise.

`direction` (`horizontal`/`vertical`) and `maybeLabels`/`controlAriaLabel` are all safe to omit — they have working PHP-side fallbacks.

---

## Labels: `maybeLabels` + `beforeText`/`afterText`

`maybeLabels` is a checkbox — set `true` to show the "before"/"after" text overlays at all; `false`/omit to hide them entirely regardless of what `beforeText`/`afterText` are set to. `beforeText`/`afterText` have schema defaults ("Before"/"After") but per the general defaults rule, set them explicitly to whatever the labels should read.

---

## Rendered structure — schema alone doesn't show this

```html
<div class="brxe-xbeforeafterimage x-before-after"
     data-x-before-after='{"direction":"horizontal","maybeMouseMove":false}'
     style="--x-before-after-position: 46.915%;">
  <div class="x-before-after_container">
    <div class="brxe-block x-before-after-image_block">
      <img class="brxe-image x-before-after-image_image ..." src="..." alt="...">
    </div>
    <div class="brxe-block x-before-after-image_block">
      <img class="brxe-image x-before-after-image_image ..." src="..." alt="...">
    </div>
  </div>
  <div class="x-before-after_slider-container">
    <input class="x-before-after_slider" type="range" min="00" max="100" step="0.001" value="50"
           aria-label="Percentage of image visible" orient="horizontal">
  </div>
  <div class="x-before-after_slider-line x-before-after_slider-line-before" aria-hidden="true"></div>
  <div class="x-before-after_slider-button" aria-hidden="true">
    <div class="x-before-after_slider-button-icon">
      <i class="ti-angle-left"></i> <i class="ti-angle-right"></i>
    </div>
  </div>
  <div class="x-before-after_slider-line x-before-after_slider-line-after" aria-hidden="true"></div>
  <div class="x-before-after_before-label x-before-after_label">one</div>
  <div class="x-before-after_after-label x-before-after_label">two</div>
</div>
```

- **The actual comparison control is a native `<input type="range">`**, not a custom drag-only div — genuinely keyboard-operable and accessible out of the box (arrow keys move it, `aria-label` describes it). The visible `.x-before-after_slider-button`/`-line` elements are decorative overlays (`aria-hidden="true"`) that visually track the real input; don't add your own ARIA to them, and don't assume you need to build keyboard support — it's already there via the native input.
- **Two distinct CSS custom properties control position, and they are not interchangeable.** The `start` setting (schema `css` selector `""`, property `--x-start-position`) seeds the *initial* value baked into the compiled CSS/config. The **root element's inline style at render time uses a different variable, `--x-before-after-position`** — this is the *live* position, updated by JS as the range input moves (46.915% here isn't a rounding artifact of "50", it's whatever the slider's actual current position was). If writing `_cssCustom` that reacts to slider position, target `--x-before-after-position`, not `--x-start-position` — the latter only ever reflects the configured starting point, never subsequent movement.
- **`iconLeft` and `iconRight` render together, simultaneously, inside the same `.x-before-after_slider-button-icon` wrapper** — not one-icon-that-swaps-by-direction and not one-on-each-side of the button. Both `<i>` tags sit side by side as a single combined glyph (e.g. "‹ ›"). Don't design around an assumption that only one shows at a time.
- **Label divs (`.x-before-after_before-label`, `.x-before-after_after-label`) are absolutely-positioned siblings at the root level**, not nested inside their respective image blocks — matches the schema's `labelTop`/`labelLeft`/`labelRight`/`labelBottom` (and `labelAfter*`) controls targeting these exact selectors independently for each side.

---

## Never do

- Do not build children as bare `image` elements — always wrap each in a `block` carrying `x-before-after-image_block`, with the image carrying `x-before-after-image_image`.
- Do not omit `iconLeft`/`iconRight` — the drag handle renders without any icon inside it.
- Do not target `--x-start-position` in custom CSS expecting it to reflect live slider movement — it only ever holds the configured starting value. The live position is a different variable, `--x-before-after-position`, set inline on the root element.

## MCP write notes

- Missing wrapper classes or missing icons both produce a "valid" write with no visible error.