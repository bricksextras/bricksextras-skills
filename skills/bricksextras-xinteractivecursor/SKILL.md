---
name: xinteractivecursor
description: "Use when adding a custom animated cursor (xinteractivecursor) from BricksExtras: a fixed, page-wide ball+trail cursor replacement with hover/text/click states. Covers that it belongs in a sitewide template (footer/global), not per-page content, that hoverSelectors needs an explicit default, and that the 'Text' state requires a data-x-hover attribute on the target element rather than any control."
---

**Requires:** BricksExtras 1.7.3+ with xinteractivecursor enabled

# BricksExtras: Interactive Cursor (xinteractivecursor)

Shipped by the **BricksExtras** plugin. A non-nestable element that replaces the OS cursor with a custom animated "ball + trail" pair, tracking real mouse movement across the whole page. It's `aria-hidden`, has no visible container of its own, and outputs `<div class="brxe-xinteractivecursor x-interactive-cursor" data-x-cursor="{...}">` wrapping two inner divs (`.x-cursor_ball`, `.x-cursor_trail`).

## Add it once, sitewide — not as page content

This is a page-wide overlay effect, not a piece of in-flow content. **Add exactly one instance in a sitewide template** (a footer template, or a dedicated global/embed template that renders on every page) rather than in individual page content areas — the same way you'd add a Back to Top button or a cookie banner. Never add more than one instance on the same page.

## `hoverSelectors` default is UI-only — write it explicitly

Same rule as documented in `bricksextras-start-here`: the schema shows `hoverSelectors` defaulting to `"a, button, input, textarea, .splide__slide, .vds-button"`, but that's a builder pre-fill, not a render-time fallback. Write it into the JSON explicitly, or the hover ("grow") state has nothing to match against and never triggers.

## The "Text" state needs a `data-x-hover` attribute on the target, not a control

The schema's `textVisibleSep` description says *"When hovering over elements with a data-x-hover attribute"* — this isn't a Bricks control on the target element, it's a raw HTML attribute you add yourself via that element's `_attributes` setting:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xinteractivecursor.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{ "_attributes": [{ "id": "attr01", "name": "data-x-hover", "value": "View more" }] }
```

Renders correctly: `<div class="brxe-text-basic" data-x-hover="View more">...</div>`.

## Testing caveat: hover-state class transitions weren't reliably reproducible headlessly

Attempting to verify `x-cursor_grow`/`x-cursor_text-visible`-style state classes via Playwright's `hover()` (a single cursor jump) and via synthetic `mousemove` dispatch gave inconsistent results between attempts — this looks like the plugin's tracking is built around a continuous requestAnimationFrame/mousemove stream rather than a single discrete hover event, which headless single-jump hovers don't reproduce reliably. Don't take an automated single-hover check as proof a state isn't working; verify visually with real mouse movement in an actual browser instead.


**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xinteractivecursor.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xinteractivecursor` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Other settings

- **Cursor trail smoothing:** `trailDelay` (0–1, placeholder `0.2`) — the only other value besides `hoverSelectors`/`wait` that's actually sent to the frontend JS config.
- **Loading delay:** `wait` (ms, placeholder `100`).
- **Shape:** `ballRadius`/`trailRadius` (placeholder `50%` — circular by default).
- **Colors:** `ballColor`/`trailColor` (CSS vars `--x-cursor-ball-color`/`--x-cursor-trail-color`).
- **`builderHidden`:** hides the element in the builder canvas only (editing convenience — the element still renders on the frontend regardless).

## Build workflow

1. Write `hoverSelectors` explicitly (schema default is UI-only).
2. Set whichever state's scale/color/typography values the design needs — all states can be written in one call; `builderPreview` doesn't gate what gets saved or rendered.
3. For the "Text" state, add `data-x-hover` via `_attributes` on each target element that should trigger it.
4. Place the element once in a sitewide template (footer/global), not in per-page content.
5. Verify visually in a real browser with actual mouse movement — headless single-hover checks are not a reliable way to confirm state transitions for this element.
