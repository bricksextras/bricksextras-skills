---
name: xbacktotop
description: "Use when building or debugging the Back to Top element (xbacktotop) from BricksExtras: a scroll-to-top button/progress-circle. Covers the icon setting having no runtime fallback, and the type/content options."
---

**Requires:** BricksExtras 1.7.3+ with xbacktotop element enabled

# BricksExtras: Back to Top (xbacktotop)

Shipped by the **BricksExtras** plugin. Not nestable when `content: "icon"` (the default); becomes nestable when `content: "nest"` (add your own elements as the button's content instead of the built-in icon/text).


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xbacktotop.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xbacktotop` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: `icon` must be set explicitly, or the button renders with no icon

`type` and `content` both fall back correctly at render time when omitted (`type` → `"progress"`, `content` → `"icon"` — both read via `isset()` with a matching PHP-side default, so leaving them unset is safe). **`icon` is different**: it's read with `! empty( $settings['icon'] )`, and there is no PHP-side fallback to its schema-declared default. Omit it and the button renders with `content: "icon"` selected but nothing inside — no chevron, no error.

Set it explicitly, matching the schema's own declared default unless told otherwise:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xbacktotop.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xbacktotop",
  "settings": {
    "icon": { "library": "fontawesomeSolid", "icon": "fas fa-chevron-up" }
  }
}
```

## Other settings

| Setting | Values | Notes |
|---|---|---|
| `type` | `progress` (default) / `regular` | `progress` renders a scroll-progress circle behind the button; `regular` is a plain button |
| `content` | `icon` (default) / `nest` | `nest` makes the element nestable — add child elements as the button's content instead of `icon`/`buttonText` |
| `buttonText` | Any text | Only shown/relevant when `content: "icon"`; text sits alongside the icon |
| `ariaLabel` | Text, no runtime fallback | **Depends on `content` mode — see below, don't treat as universally required or universally safe to omit** |
| `scrollDistance` | Number (px scrolled before the button appears) | Safe to omit — defaults to `100` |
| `scrollUp` | Checkbox | `true` to enable, `false`/omit to disable |

---

## `ariaLabel`: required for `content: "icon"`, but do NOT add it for `content: "nest"`

This looks like the same "no PHP fallback" problem as `icon`, but it isn't — it's mode-dependent, and getting it wrong in either direction is a real accessibility issue, not just a cosmetic gap:

- **`content: "icon"` (default):** the button's only visible content is an icon (and optionally `buttonText`, which sits *alongside* the icon, not as the icon's accessible name). `$aria_label` is `false` unless `ariaLabel` is explicitly set, and the `aria-label` attribute is only written when truthy — there is no fallback string. **Omit it here and the button has no accessible name at all.** Set it explicitly.
- **`content: "nest"`:** the button's content is whatever elements the user nests inside it. If that nested content includes visible text (a heading, a text element, anything readable), **adding `ariaLabel` on top of it is wrong** — it produces a redundant/conflicting accessible name that accessibility testers flag (the visible text and the `aria-label` competing as the button's name). The right call in this mode is usually to leave `ariaLabel` unset and let the nested visible content provide the accessible name naturally — same as any native `<button>` with visible text.

Don't apply a blanket rule here ("always set it" or "safe to omit") — check what `content` mode is in play and what's actually inside the button before deciding.

The builder's own editing preview (`render_builder()`) hardcodes a visible `aria-label="back to top"` in its Vue template regardless of mode or settings — that's editor-only convenience markup, not what ships to the frontend, and it will make an icon-mode button missing `ariaLabel` look fine in the builder while shipping with no accessible name in production. Don't trust the builder preview to tell you whether `ariaLabel` is actually needed.

---

## Rendered structure — schema alone doesn't show this

### `content: "icon"`, `type: "progress"` (defaults)

```html
<button class="brxe-xbacktotop x-back-to-top"
        data-x-backtotop='{"type":"progress","scrollDistance":100,"scrollUp":false}'
        aria-hidden="false" style="opacity: 1; transform: none;">
  <svg class="x-back-to-top_progress" height="100%" viewBox="0 0 100 100" width="100%">
    <path class="x-back-to-top_progress-background" d="M50,1 a50,50 0 0,1 0,100 a50,50 0 0,1 0,-100"></path>
    <path class="x-back-to-top_progress-line" d="M50,1 a50,50 0 0,1 0,100 a50,50 0 0,1 0,-100"
          style="stroke-dasharray: 314.204; stroke-dashoffset: 47.544;"></path>
  </svg>
  <span class="x-back-to-top_content">
    <span class="x-back-to-top_icon"><i class="fas fa-chevron-up"></i></span>
  </span>
</button>
```

- **Root element is the `<button>` itself** — no wrapping div. `_cssClasses`/`_cssCustom` targeting this element lands directly on the button.
- **`type: "progress"`'s ring is two overlaid SVG `<path>` circles**, not a CSS-only effect: `.x-back-to-top_progress-background` is the static full ring, `.x-back-to-top_progress-line` is the actual progress indicator, animated via inline `stroke-dasharray`/`stroke-dashoffset` recalculated by JS as the page scrolls. `type: "regular"` omits this whole `<svg>` block. Custom styling of the ring itself (color/thickness) works through the schema's normal border/color controls on these classes; the *position* of the dash-offset is JS-owned, same caution as the toggle switch's positioned highlight — don't expect CSS alone to move where the progress indicator starts/ends.
- **`aria-hidden` and the inline `opacity`/`transform` are the actual show/hide mechanism** as the visitor crosses `scrollDistance` — not a class toggle. If wiring custom `_interactions` or CSS transitions around this element's visibility, target these, not a class.
- **The icon is a plain `<i class="...">`** (Font Awesome-style class icon in this example) — confirms `icon: {library, icon}` maps directly to an `<i>` tag when using a class-based icon library. Rendered `data-x-backtotop` JSON always includes `scrollUp` as an explicit `true`/`false`, reflecting whatever was actually set (or the default `false` if omitted).

### `type: "regular"` (no progress ring)

```html
<button class="brxe-xbacktotop x-back-to-top"
        data-x-backtotop='{"type":"regular","scrollDistance":100,"scrollUp":false}'
        aria-hidden="false" style="opacity: 1; transform: none;">
  <span class="x-back-to-top_content">
    <div class="brxe-text-basic">custom element</div>
  </span>
</button>
```

- Confirms the `<svg class="x-back-to-top_progress">` block is **entirely absent** for `type: "regular"`, not just hidden via CSS — there's nothing to select/style for a progress ring in this mode, and no way to fake one back in via `_cssCustom` without the underlying markup existing. If a design needs the ring, `type` has to be `"progress"`.
- `content` mode (icon vs nest) is independent of `type` — this example combines `regular` + `nest`, but `regular` + `icon` renders the same way minus the nested content, just the plain `.x-back-to-top_content > .x-back-to-top_icon` from the first example with no `<svg>` sibling.

### `content: "nest"`

```html
<button class="brxe-xbacktotop x-back-to-top"
        data-x-backtotop='{"type":"progress","scrollDistance":100,"scrollUp":false}'
        aria-hidden="false" style="opacity: 1; transform: none;">
  <svg class="x-back-to-top_progress">...</svg>
  <span class="x-back-to-top_content">
    <div class="brxe-text-basic">custom element</div>
  </span>
</button>
```

- **Nested children render inside `.x-back-to-top_content`**, replacing the built-in `.x-back-to-top_icon`/`buttonText` markup entirely — same wrapper span, different contents. Whatever you nest (heading, text-basic, an icon element, multiple elements) lands there verbatim.
- The progress-ring `<svg>` (when `type: "progress"`) is unaffected by `content` mode — it's a sibling of `.x-back-to-top_content`, not something `content: "nest"` replaces.

---

## Never do

- Do not omit `icon` when `content: "icon"` — the button renders with nothing visible inside it. This is specific to `icon`; `type` and `content` are safe to leave unset.
- Do not omit `ariaLabel` when `content: "icon"` — the button ships with no accessible name at all; there is no fallback.
- Do not add `ariaLabel` when `content: "nest"` and the nested content already has visible text — it creates a redundant/conflicting accessible name, which accessibility testers flag as an error, not a nicety.
- Do not trust the builder's editing preview to validate `ariaLabel` — it hardcodes a visible label in its own template regardless of your actual settings or mode.
- Do not assume every setting on this element behaves the same way when omitted — check each one, since PHP fallback behavior differs per field even within the same element (`type`/`content` have working fallbacks, `icon` and `ariaLabel`-in-icon-mode don't).

## MCP write notes

- An omitted `icon` produces a "valid" settings write with no visible error — only a missing icon in the browser.
- An omitted `ariaLabel` in `content: "icon"` mode produces a "valid" settings write with no visible error — only a missing accessible name, which won't show up in a browser at all and needs an accessibility check (screen reader or automated a11y test) to catch.
