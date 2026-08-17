---
name: xoffcanvasnestable
description: "Use when building or debugging the OffCanvas element (xoffcanvasnestable) from BricksExtras: a slide-in/fade-in panel with backdrop, built directly from nested children. Covers the clickTrigger targeting pattern, direction-dependent width/height controls, the disable_backdrop structural (not just visual) omission, and query-loop instance uniqueness."
---

**Requires:** BricksExtras 1.7.3+ with xoffcanvasnestable enabled

# BricksExtras: OffCanvas (xoffcanvasnestable)

Shipped by the **BricksExtras** plugin. A nestable element — its own children render directly inside `.x-offcanvas_inner`, no separate template needed (unlike the legacy `xoffcanvas`, which pulls content from a selected Bricks Template). Always use this version for new work; only touch `xoffcanvas` when editing a page that already uses it.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xoffcanvasnestable.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xoffcanvasnestable` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Targeting: `clickTrigger` has no safe default, same rule as everywhere else

`clickTrigger`'s schema default is `.brxe-xburgertrigger`, but that's a builder-UI placeholder, not a render-time fallback — per `bricksextras-start-here`'s targeting rule, always give the actual trigger element (e.g. `xburgertrigger`) an explicit `_cssClasses` value and point `clickTrigger` at that class. Add `componentScope: "true"` (the string, not boolean) if both live inside the same component.

## `direction` gates which size control applies

`direction`: `x-offcanvas_left`/`x-offcanvas_right` (horizontal slide-in) or `x-offcanvas_top`/`x-offcanvas_bottom` (vertical). `offcanvas_width` only applies (schema-gated) when `direction` is left/right; `offcanvas_height` only applies when it's top/bottom — setting the wrong one for the current direction has no effect.

## `disable_backdrop` removes the backdrop element entirely, not just visually

When `disable_backdrop` is `true`, the `.x-offcanvas_backdrop` div is never output at all — this is a structural omission, not a CSS hide. `backdrop_to_close`/`backdrop_color`/`backdrop_zindex` are schema-gated to `disable_backdrop != true` accordingly, since there's nothing for them to style once the backdrop doesn't exist.

## Other behavior worth knowing

- **`inert` is always present on `.x-offcanvas_inner`** — a real HTML attribute disabling interactivity while closed, alongside `aria-hidden`/`tabindex`/`role`, removed by the JS when the panel opens.
- **`move_focus`**: `content` (default — JS's own default focus behavior, no extra config written) / `selector` (writes `focus_selector` into the JS config) / `disable` (writes `focus: "false"` explicitly).
- **`maybe_hash_close`** defaults to enabled; the JS config only gets `disableHashlink: "true"` written when explicitly set to `"false"` — omitting the setting relies on the JS's own default rather than an explicit config key.
- **Query-loop instance uniqueness**: inside a running query loop, the config gets `isLooping`/`isLoopingComponent` (loop id) written in automatically — same category of mechanism used elsewhere in BricksExtras (e.g. `xwsforms`) to keep repeated instances distinguishable.
- **`tag`** (root wrapper HTML tag, placeholder `div`) and **`role`** (placeholder `dialog`) both need explicit values if anything other than the placeholder is wanted — standard schema-defaults-are-UI-only rule.

## Rendered DOM (for custom CSS/targeting)

Captured live, `clickTrigger` wired to a burger trigger, closed state — unlike the modal, this is present in the raw server HTML from the start, no browser-JS timing needed to see it:

```html
<div class="brxe-xoffcanvasnestable x-offcanvas" data-x-id="{id}" data-x-offcanvas="{...}">
  <div class="x-offcanvas_inner x-offcanvas_left" id="x-offcanvas_inner-{id}"
       aria-hidden="true" aria-label="Offcanvas" role="dialog" inert tabindex="0" data-type="slide">
    <h3 class="brxe-heading">Offcanvas title</h3>
    <div class="brxe-text"><p>Offcanvas body content.</p></div>
  </div>
  <div class="x-offcanvas_backdrop"></div>
</div>
```

Notes on this structure:

- **No separate container wrapper** — unlike the modal (`.x-modal_container` + `.x-modal_content` as two levels), `.x-offcanvas_inner` is a single element that's both the `role="dialog"` holder *and* where your children render directly. Target it directly, not a nonexistent inner content div.
- **The backdrop is a sibling of `.x-offcanvas_inner`, not a wrapper around it** — opposite structure from the modal, where the backdrop wraps the container. `.x-offcanvas_backdrop` renders as an empty div here (unless `disable_backdrop` removes it entirely — see above).
- `aria-label="Offcanvas"` is a real PHP-side fallback when `aria_label` is unset — genuinely server-rendered, unlike the modal's client-JS-computed aria-label. A raw HTML fetch will show this correctly.
- `id="x-offcanvas_inner-{id}"` on the inner panel is deterministic and server-rendered (not the rewrite-on-init pattern seen on the modal's root) — safe to rely on for targeting.
- `inert`, `aria-hidden="true"`, and `tabindex="0"` are all present in this closed-state capture, confirming the "always present, removed by JS on open" behavior already noted above — the open state drops `inert`/flips `aria-hidden="false"`.

1. Give the trigger element an explicit class; point `clickTrigger` at it. Never `_cssId`.
2. Set `direction` first, then only the matching size control (`offcanvas_width` for left/right, `offcanvas_height` for top/bottom).
3. Nest real content directly as children — no template selection step needed.
4. Leave `disable_backdrop` unset unless the backdrop should be structurally absent, not just invisible.