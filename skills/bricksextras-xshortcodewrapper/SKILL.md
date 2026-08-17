---
name: xshortcodewrapper
description: "Use when building or debugging the Shortcode Wrapper element (xshortcodewrapper) from BricksExtras: wraps a Bricks Template, wysiwyg text, or nested elements inside a shortcode's opening/closing tags, useful for feeding Bricks-built markup into a 3rd-party shortcode that expects to wrap its own content. Covers the auto-derived closing tag and the builder-only hide mechanism."
---

**Requires:** BricksExtras 1.7.3+ with xshortcodewrapper enabled

# BricksExtras: Shortcode Wrapper (xshortcodewrapper)

Shipped by the **BricksExtras** plugin. Nestable. Purpose: take Bricks-built content (a Template, wysiwyg text, or nested elements) and wrap it inside a shortcode's open/close tags at render time — for 3rd-party shortcodes designed to wrap arbitrary inner content (e.g. `[some_shortcode]...inner...[/some_shortcode]`) rather than ones that render everything themselves.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xshortcodewrapper.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xshortcodewrapper` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `fullShortcode` — enter the open/close shortcode tag pair, as the field expects

Set `fullShortcode` to the shortcode's opening and closing tags, e.g. `[some_shortcode attr="value"][/some_shortcode]` — this is what makes the element genuinely different from Bricks' own core `shortcode` element, which just renders a single, complete shortcode string. Here, the wrapped content (`shortcodeContent`) is inserted *between* the open and close tags at render time, so a 3rd-party shortcode that's designed to wrap arbitrary inner content can wrap real Bricks-built markup instead of plain text.

Internally, `render()` only regex-captures the opening `[tagname ...]` construct and rebuilds the closing tag itself as `[/tagname]` from that captured name — so the literal closing tag text typed in the field isn't used verbatim, but entering the full open/close pair (matching the control's own placeholder) is still the correct and expected way to fill it in.

## `shortcodeContent` sources

| `shortcodeContent` | Source |
|---|---|
| `wysiwyg` (default if unset) | `shortcodeWysiwyg` editor field |
| `template` | `templateId` — a selected Bricks Template, rendered via `[bricks_template id="..."]` |
| `nestable` | Real nested children |

`templateId` is only schema-visible when `shortcodeContent` is neither `nestable` nor `wysiwyg` — i.e. effectively only for `template` mode.

## `builderHidden` hides content only in the builder canvas, never the frontend

When set, `render()` wraps the inner content in a `.x_hide_in_builder` div, but only while in the builder preview (`maybe_preview()`) — on the real frontend this wrapper is never added and content always renders normally regardless of the setting. Use this to keep a shortcode's live output (e.g. a form, a widget with its own JS) from cluttering the builder canvas without affecting real visitors.

## Build workflow

1. Set `fullShortcode` to the real shortcode's opening and closing tag pair, e.g. `[some_shortcode][/some_shortcode]`.
2. Pick `shortcodeContent` explicitly to choose what gets wrapped inside those tags — a Template (`templateId`), wysiwyg text (`shortcodeWysiwyg`), or real nested elements.
