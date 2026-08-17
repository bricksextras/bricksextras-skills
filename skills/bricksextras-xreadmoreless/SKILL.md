---
name: xreadmoreless
description: "Use when building or debugging the Read More / Less element (xreadmoreless) from BricksExtras: a nestable content-collapse element built on the Readmore.js library. Covers the wysiwyg-vs-nestable content mode switch, and that the collapse threshold is height + margin (not just collapsedHeight) so the button silently doesn't appear for short content."
---

**Requires:** BricksExtras 1.7.3+ with xreadmoreless enabled

# BricksExtras: Read More / Less (xreadmoreless)

Shipped by the **BricksExtras** plugin. A nestable element built on the third-party **Readmore.js** library, collapsing content to a fixed height with a toggle button. Several behaviors here aren't derivable from the schema alone.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xreadmoreless.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xreadmoreless` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Whether the button appears at all is conditional on actual rendered height

**This is the single most important thing to understand about this element, and it's easy to mistake for a bug during testing:** setting `collapsedHeight` does not guarantee a Read More button appears. The underlying Readmore.js check is:

```
content height <= collapsedHeight + heightMargin
```

If the real content's rendered height is *already* within `heightMargin` of `collapsedHeight`, the library decides nothing needs collapsing: no button gets inserted, `.x-read-more_content` gets the `x-read-more_not-collapsable` class, and any fade gradient is explicitly removed (`readMore.classList.remove('x-read-more_fade')` in the `blockProcessed` callback). This is intentional, correct behavior — not broken. `heightMargin` defaults to `20` in the JS if the setting is omitted (matching its schema placeholder), so in practice the real threshold is always `collapsedHeight + 20` unless `heightMargin` is set otherwise. **When testing or debugging "why isn't the button showing," first check whether the actual content height clears that combined threshold** before assuming anything is misconfigured.

## `collapsedHeight` is CSS-only — never sent to the JS config

The `$config` array built for the `data-x-readmore` JSON attribute only ever includes `heightMargin`, `speed`, `moreText`, `lessText`, `moreAria`, `lessAria`. `collapsedHeight` has no JS-config counterpart at all — it only ever reaches the page via its `css` mapping to `.x-read-more_content`'s `max-height`. The JS reads the *actual applied* CSS max-height off the element at runtime (via `readMore.style.maxHeight`) rather than being told the value directly. Practical effect: if the generated stylesheet for this element ever fails to apply (caching, a bad selector override elsewhere, etc.), the collapse mechanism has nothing to measure against — check the rendered CSS directly if collapse behavior looks wrong, not just the saved element settings.

## Two mutually exclusive content sources — `readMoreContent`

- **`wysiwyg`** (schema default) — content comes from `readMoreWysiwyg`, a rich text editor field.
- **`nestable`** — content comes from the element's own **children**, rendered normally like any other nestable element's children.

Only one is live at a time; `readMoreWysiwyg` is schema-gated to `!= nestable` and simply unused in nestable mode. Always set real content in whichever mode is used — for `wysiwyg`, that means writing actual content into `readMoreWysiwyg` explicitly.

## `maybeIcon` gates whether the icon shows

Set `maybeIcon` when an icon is wanted alongside the more/less button text, alongside setting `icon` itself.

## Other settings

- **`speed`** (ms, JS default `300` if omitted) — collapse/expand transition duration.
- **`heightMargin`** (px, JS default `20` if omitted) — see the threshold explanation above; this is a real functional value, not just cosmetic.
- **`maybeGradient`** (`true`/`false` select, no schema default — set explicitly) — enables the fade-out overlay while collapsed; automatically suppressed if the content turns out not-collapsable regardless of this setting.
- **`moreText`/`lessText`/`moreAria`/`lessAria`** all have real schema defaults ("Read more"/"Read Less") — per the general rule in `bricksextras-start-here`, write them explicitly if not customizing.
- **`buttonPosition`**: `static`/`absolute` — `absolute` needs `buttonBottom` (gated to it); `static` needs `buttonMargin` instead (gated to `!= absolute`) — mutually exclusive, matching the control's own `info` text about reducing layout shift above the fold.

## Rendered DOM (for custom CSS/targeting)

**The toggle button doesn't exist in the server-rendered HTML at all — Readmore.js inserts it client-side, only once it determines the collapse threshold is actually cleared.** The initial PHP output is just the two wrapper divs and the content, no button, no `data-readmore` attribute yet. Once it decides to collapse, it rewrites the content div in place and inserts `<button class="x-read-more_link">` as a sibling after it:

```html
<div id="brxe-{id}" class="brxe-xreadmoreless x-read-more_fade" data-x-fade data-x-id="{id}" data-x-readmore="{...}">
  <div class="x-read-more_content rmjs-1" data-readmore id="rmjs-2" style="height: 60px; max-height: none;">
    <p>...</p>
  </div>
  <button aria-expanded="false" aria-label="" class="x-read-more_link" data-readmore-toggle="rmjs-2" aria-controls="rmjs-2">
    <span class="x-read-more_link-text">Read more</span>
    <span class="x-read-more_link-icon"><i class="fas fa-chevron-down"></i></span>
  </button>
</div>
```

Expanding (click) just adds `x-read-more_expand` to the content div, recalculates its inline `height`, and flips the button's text/`aria-expanded` from `false`→`true` — the icon itself doesn't change between states. `aria-label` stays empty unless `moreAria`/`lessAria` are set explicitly.

## Build workflow

1. Choose `readMoreContent` mode and populate the matching field — `readMoreWysiwyg` (always write real content explicitly; there's no safe fallback) or real nested children.
2. Set `collapsedHeight` meaningfully shorter than the expected real content height, accounting for `heightMargin`'s default 20px buffer (or set `heightMargin` explicitly if a different threshold is wanted).
3. If an icon is wanted, set both `maybeIcon` and `icon`.
4. Verify in the browser with content that's actually taller than the combined threshold — a shorter test paragraph will legitimately produce no button and can look like a bug when it isn't.
