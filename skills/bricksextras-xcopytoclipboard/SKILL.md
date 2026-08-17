---
name: xcopytoclipboard
description: "Use when building or debugging the Copy to Clipboard element (xcopytoclipboard) from BricksExtras: a button that copies text (dynamic data or another element's content) and shows a copied state. Covers copyFrom modes, and the icon settings needing explicit enabling."
---

**Requires:** BricksExtras 1.7.3+ with xcopytoclipboard element enabled

# BricksExtras: Copy to Clipboard (xcopytoclipboard)

Shipped by the **BricksExtras** plugin. Nestable (rarely nested in practice — most builds use it as a self-contained button with no children).


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcopytoclipboard.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xcopytoclipboard` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Two source modes: `copyFrom`

- **`"dynamic_data"`** — copies whatever's in `copytext`, which accepts dynamic tags (`{post_title}`, ACF tags, etc.) or plain text. This is what to use when copying a value from the current loop/post context. `stripTags` (enable/disable) controls whether HTML in the resolved value is stripped before copying.
- **`"selector"`** (default if `copyFrom` omitted) — copies the text content of another element on the page, targeted via `copySelector` (a CSS selector) — same class-based cross-element targeting pattern as the rest of BricksExtras (see `bricksextras-start-here`'s "Targeting another element" section). Has its own `componentScope` field for the same component-duplication-safety reason as `xproslidercontrol`.

**This is the complete baseline — every field here matters, don't drop any when copying this example:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcopytoclipboard.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xcopytoclipboard",
  "settings": {
    "copyFrom": "dynamic_data",
    "copytext": "{post_title}",
    "copyButtonText": "Copy",
    "copiedButtonText": "Copied",
    "maybeIcons": "enable",
    "copyIcon": { "library": "themify", "icon": "ti-files" },
    "afterPressed": "icon",
    "copiedIcon": { "library": "themify", "icon": "ti-check" }
  }
}
```

---

## Critical: icons need `maybeIcons: "enable"` set explicitly, plus their own values

The icon fields (`copyIcon`, `copiedIcon`) don't render at all unless `maybeIcons` is explicitly `"enable"` — despite the control's `placeholder` reading "Enable" in the builder (a placeholder is a visual hint, not an applied value; omitting the key leaves icons off). Once enabled, `copyIcon`/`copiedIcon` still need their own values set — same "declared default isn't auto-applied" behavior seen elsewhere in this plugin, not something `maybeIcons: "enable"` alone fixes. **These are already included in the baseline example above — don't build from a stripped-down version that drops them, even for a "simple" button.**

`afterPressed` decides what shows after a successful copy: `"check"` (an animated checkmark built into the plugin's own CSS/SVG, no icon field needed) or `"icon"` (swaps to `copiedIcon`, which must be set for anything to show).

If `afterPressed` is left as `"check"` instead, `copiedIcon` isn't needed — the built-in checkmark animation handles the copied state.

---

## Button text vs. icons — independent, not either/or

`copyButtonText` (default `"Copy"`, safe to omit) and `copiedButtonText` (no PHP-side default — set explicitly, e.g. `"Copied"`, or the button's text won't change on copy) both render **alongside** the icon, not instead of it. For a text-only button with no icon, just leave `maybeIcons` unset (or set it to `"disable"` explicitly).

---

## Rendered structure — schema alone doesn't show this

```html
<div class="brxe-xcopytoclipboard"
     data-x-copy-text="Post A"
     data-x-copy-to-clipboard='{"componentScope":"false","isLooping":"{loop-parent-id}"}'>
  <button class="x-copy-to-clipboard" aria-pressed="false" data-x-animation="fade">
    <span class="x-copy-to-clipboard_text">Copy title</span>
    <span class="x-copy-to-clipboard_icons">
      <span class="x_copied-icon x-copy-to-clipboard_icon" aria-hidden="true"><i class="ti-check"></i></span>
      <span class="x_copy-icon x-copy-to-clipboard_icon" aria-hidden="true"><i class="ti-files"></i></span>
    </span>
  </button>
</div>
```

- **Root element is a wrapping `<div>`, not the button itself** — unlike `xbacktotop`, where the root *is* the `<button>`. The real button lives one level in at `.x-copy-to-clipboard`; that's what native style controls already target (matching the schema's own `css` selectors), and what any custom `_interactions`/`_cssCustom` aimed at hover/focus/click states should target too, not the outer wrapper.
- **`data-x-copy-text` is the already-resolved value, baked in at render time** — for `copyFrom: "dynamic_data"` this is your dynamic tag's rendered output (here, the literal string `"Post A"`, not `"{post_title}"`). It's set once when the page renders, not re-fetched on click — if the underlying data can change client-side after load without a page reload, the copy button won't pick that up.
- **Both `copyIcon` and `copiedIcon` render simultaneously in the markup at all times** — `.x_copy-icon` and `.x_copied-icon` are both always present (both `aria-hidden`), not one swapped in via JS when the copy succeeds. The toggle between them is a CSS state change keyed off the button's `aria-pressed` attribute (`false` → `true` after a successful copy), animated via `data-x-animation` (here `"fade"`, matching `iconAnimation`'s schema default — unlike `copyIcon`/`copiedIcon` themselves, `iconAnimation` does have a working runtime fallback, since it appears here despite not being set). This "both states always in the DOM, toggled by an ARIA/state attribute rather than swapped by JS" pattern shows up elsewhere in BricksExtras too (e.g. `xbeforeafterimage`'s two drag-handle icons) — don't assume a single icon element gets its icon swapped out; expect two, and target the state-selector variants (`[aria-pressed=true] .x_copied-icon` etc., matching the schema's own `buttonCopiedTypography`/`popoverCopiedBackgroundColor`-style `[aria-pressed=true]` selectors) when styling the "after copy" state specifically.
- Both icon spans and the text span are always accessibility-inert (`aria-hidden`) or plain text — the button's accessible name comes entirely from `.x-copy-to-clipboard_text`'s visible content, confirming icons are purely decorative here, same as everywhere else in the plugin.

---

## Never do

- Do not assume `maybeIcons` defaults to enabled because the builder placeholder says "Enable" — omitting the key means no icons render at all.
- Do not set `afterPressed: "icon"` without also setting `copiedIcon` — there's no fallback icon.
- Do not forget `copiedButtonText` if button text should change after copying — unlike `copyButtonText`, it has no working default.
- Do not target the outer `.brxe-xcopytoclipboard` div for button-state styling/interactions — the real `<button>` (and its `aria-pressed` state) is one level in, at `.x-copy-to-clipboard`.

## MCP write notes

- A missing `maybeIcons: "enable"` produces a "valid" write with a working copy button that simply never shows any icon.