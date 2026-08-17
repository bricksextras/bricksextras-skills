---
name: xlottie
description: "Use when building or debugging the Lottie element (xlottie) from BricksExtras: animated vector graphics with interactive triggers (click, hover, scroll, cursor). Covers trigger types, selector targeting, frame ranges, and performance options."
---

**Requires:** BricksExtras 1.7.3+ with xlottie element enabled

# BricksExtras: Lottie (xlottie)

Shipped by the **BricksExtras** plugin. Renders Lottie animations (JSON-based vector animations) with interactive triggers. Built on **Lottie Web** (light or full version).

**Not nestable:** This element has no children.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xlottie.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xlottie` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Core settings

### Animation source

**`lottieURL`** (text) - URL to the Lottie JSON file. Supports dynamic data.

- **LottieFiles:** https://lottiefiles.com/ - free and premium animations
- **Example URL:** `https://assets8.lottiefiles.com/packages/lf20_8qDRX7nBln.json`
- Can use dynamic data to pull from ACF fields, post meta, etc.

### Sizing

**`width`** (number with units) - Width of the animation container. CSS-mapped to `width` property.

- Default placeholder: `300px`
- Lottie animations are typically vector-based and scale cleanly
- Height is usually auto-calculated from the animation's aspect ratio

---

## Frame range

**`frameStart`** (number) - Starting frame of the animation.
**`frameEnd`** (number) - Ending frame of the animation.

- **Defaults:** `0` to `60` (placeholders shown in schema)
- **Stick to defaults unless specified** - actual frame count depends on the specific Lottie animation being used
- Different animations have different total frame counts
- Only override if the user specifies a custom range or you know the animation's frame count

---

## Trigger types

**`trigger`** (select) - How the animation is triggered. **Only one trigger per element.**

### Available triggers:

1. **`viewport`** - Animation plays when element enters viewport
   - Requires: `autoPlay` and/or `loop` checkboxes
   - Use `offsetTop`/`offsetBottom` for scroll offset

2. **`scroll`** - Animation scrubs based on scroll position relative to this element
   - Animation progress tied to scroll position
   - Use `offsetTop`/`offsetBottom` for viewport offset percentages
   - **`speed` is disabled** for scroll triggers (animation is scroll-driven, not time-based)

3. **`scrollSelector`** - Animation scrubs based on scroll position relative to another element
   - Requires: `scrollElementSelector` (CSS selector)
   - Use `offsetTop`/`offsetBottom` for viewport offset percentages
   - **`speed` is disabled** for scroll triggers

4. **`click`** - Animation plays when this element is clicked
   - Optional: `reverseClick` checkbox - reverses animation on second click

5. **`clickSelector`** - Animation plays when another element is clicked
   - Requires: `clickElementSelector` (CSS selector)
   - Optional: `reverseClick` checkbox

6. **`hover`** - Animation plays when hovering over this element
   - Optional: `hoverReverse` checkbox - reverses animation on mouseout

7. **`hoverSelector`** - Animation plays when hovering over another element
   - Requires: `hoverSelector` (CSS selector)
   - Optional: `hoverReverse` checkbox

8. **`cursor`** - Animation follows cursor position
   - **`speed` is disabled** for cursor trigger
   - Animation frame changes based on cursor movement

9. **`none`** (or omit trigger) - No automatic trigger
   - Requires: `autoPlay` and/or `loop` checkboxes to play

**Default:** `click` (schema default)

---

## Selector targeting

When using `clickSelector`, `hoverSelector`, or `scrollSelector` triggers, you must provide a CSS selector to target another element.

**Always use a class selector, not an ID.** See `bricksextras-start-here` skill, "Targeting another element" section for why: `_cssId` collides across component instances.

**Pattern:**
1. Add a class to the target element via `_cssClasses`: `"hover-trigger"`
2. Set the selector field to: `".hover-trigger"`

**Example - hover over a section to trigger Lottie:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xlottie.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "section",
  "settings": {
    "_cssClasses": "lottie-hover-zone"
  },
  "children": [
    {
      "name": "xlottie",
      "settings": {
        "lottieURL": "https://assets8.lottiefiles.com/packages/lf20_8qDRX7nBln.json",
        "trigger": "hoverSelector",
        "hoverSelector": ".lottie-hover-zone",
        "hoverReverse": true
      }
    }
  ]
}
```

**Component scope:** This element does **not** have a `componentScope` setting in the current schema. If the Lottie element and its target both live inside a component that gets duplicated, class-based targeting will match the first instance on the page. Keep selector-targeted Lotties outside components or ensure only one instance per page.

---

## Speed

**`speed`** (number) - Playback speed multiplier.

- Default: `1` (normal speed)
- `2` = double speed, `0.5` = half speed
- **Not available when `trigger` is:** `scroll`, `scrollSelector`, or `cursor` (these are position-driven, not time-based)

---

## Scroll offset (scroll triggers only)

**`offsetTop`** (number, percentage) - Distance from top of viewport where animation starts.
**`offsetBottom`** (number, percentage) - Distance from bottom of viewport where animation starts.

- Only applies to `scroll` and `scrollSelector` triggers
- Values are percentages (0-100)
- Default: `0` for both

---

## Autoplay & Loop (viewport/none triggers only)

**`autoPlay`** (checkbox) - Animation plays automatically.
**`loop`** (checkbox) - Animation loops continuously.

- Only available when `trigger` is `viewport` or `none`

---

## Performance

**`lottieVersion`** (select) - Which Lottie library version to load.

- **`light`** (default) - ~39% smaller, doesn't support expressions or effects
- **`full`** - Full Lottie library with all features

**Use `light` unless the animation requires expressions or effects.** Most animations from LottieFiles work fine with the light version.

---

## Verified pattern: Hover over section to trigger animation

Tested end-to-end on a live site:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xlottie.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "section",
  "settings": {
    "_cssClasses": "hero-section",
    "_padding": {"top": "4rem", "bottom": "4rem"}
  },
  "children": [
    {
      "name": "container",
      "settings": {
        "_display": "flex",
        "_justifyContent": "center",
        "_alignItems": "center"
      },
      "children": [
        {
          "name": "xlottie",
          "label": "Hover Animation",
          "settings": {
            "lottieURL": "https://assets8.lottiefiles.com/packages/lf20_8qDRX7nBln.json",
            "width": {"number": 400, "unit": "px"},
            "trigger": "hoverSelector",
            "hoverSelector": ".hero-section",
            "hoverReverse": true,
            "lottieVersion": "light"
          }
        }
      ]
    }
  ]
}
```

**Behavior:** When hovering over the section, the Lottie animation plays forward. On mouseout, it reverses back to the start.

---

## No part of the animation itself can be styled through this element

Lottie Web renders real SVG (shapes, groups, colors all baked into `<path fill="...">` etc. from the source `.json` file), but none of it is a usable styling surface — the schema has no color/style override controls at all, and the shape/layer structure is entirely arbitrary per animation file. Any visual change to the animation (colors, shapes) has to happen at the Lottie creation/export stage, before the file is used here — not via this element's settings or custom CSS.

## Gotchas

- **Only one trigger per element** - You cannot combine multiple triggers. If you need different trigger behaviors, use multiple Lottie elements.
- **Selector fields have no runtime fallback** - If `hoverSelector`/`clickElementSelector`/`scrollElementSelector` is empty or doesn't match anything, the trigger silently fails. Always provide a valid selector.
- **Frame range depends on the animation** - Default `0-60` is just a placeholder. Different Lottie animations have different total frame counts. Stick to the defaults unless you know the specific animation's frame count.
- **Speed is disabled for scroll/cursor triggers** - These triggers are position-driven, not time-based, so `speed` doesn't apply.
- **Light version doesn't support expressions/effects** - If an animation uses these features and you use `lottieVersion: light`, it may not render correctly. Switch to `full` if needed.

---

## Workflow for a new Lottie element

1. **Choose a Lottie animation** - Use LottieFiles.com or provide a custom JSON URL
2. **Decide trigger type** - Click, hover, scroll, viewport, cursor, or none
3. **If using selector-based trigger** (clickSelector, hoverSelector, scrollSelector):
   - Add a class to the target element via `_cssClasses`
   - Set the selector field to `.that-class`
4. **Set frame range** - Stick to defaults (0-60) unless specified
5. **Set performance** - Use `lottieVersion: "light"` unless the animation needs expressions/effects
6. **Build and verify** - Check that the trigger works as expected on the frontend

---

## Never do

- Do not combine multiple triggers on one element - only one trigger is supported
- Do not use `_cssId` for selector targeting - use a class instead (see `bricksextras-start-here`)
- Do not set `speed` when `trigger` is `scroll`, `scrollSelector`, or `cursor` - it's disabled for these triggers
- Do not assume frame ranges - stick to defaults (0-60) unless you know the animation's frame count
- Do not leave selector fields empty when using selector-based triggers - provide a valid class selector

---

## MCP write notes

- Selector fields (`hoverSelector`, `clickElementSelector`, `scrollElementSelector`) require valid CSS selectors - use class selectors, not IDs
- `speed` is conditionally disabled based on trigger type - check schema `required` conditions
- Default `lottieVersion` is `"light"` - only use `"full"` if needed
