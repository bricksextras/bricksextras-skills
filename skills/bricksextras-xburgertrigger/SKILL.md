---
name: xburgertrigger
description: "Use when adding a hamburger/menu-toggle button (xburgertrigger) from BricksExtras: a small non-nestable button element with animated icon styles, meant to trigger an offcanvas, nav-nested, or dropdown open/close. Covers its specific settings and that it has no target field of its own — wiring is set from the target element's side, per the general rule in bricksextras-start-here."
---

**Requires:** BricksExtras 1.7.3+ with xburgertrigger enabled

# BricksExtras: Burger Trigger (xburgertrigger)

Shipped by the **BricksExtras** plugin. A small, non-nestable button element: an animated hamburger icon (optionally paired with text), meant to be clicked to open something else (offcanvas, nav-nested, dropdown). It has no children and no field of its own that says what it opens.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xburgertrigger.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xburgertrigger` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

**This element has no `clickTrigger`/target field at all.** The general targeting rule already covered in `bricksextras-start-here` ("Targeting another element: use a class, never `_cssId`") applies directly — give this element an explicit `_cssClasses` value, then point the *target* element's own trigger-selector field at that class. Don't restate that mechanism here; just remember it applies, since it's easy to look at this element's schema and assume a target setting is missing/broken rather than realize the wiring lives elsewhere.

`burger_animation` and `aria_label` both show schema `default`s that are UI-only pre-fills, per `bricksextras-start-here`'s "Schema defaults are UI-only" rule — write them explicitly rather than omitting the keys. An `xburgertrigger` built without `burger_animation` renders a static icon with no animation class at all.

---

## Settings specific to this element

- **`burger_animation`** (select, default `x-hamburger--slider`): `x-hamburger--3dx`, `--3dy`, `--arrow`, `--arrowalt`, `--arrowturn`, `--boring`, `--collapse`, `--elastic`, `--minus`, `--slider`, `--squeeze`, `--vortex`, `--none`.
- **Icon styling:** `burger_scale` (0–1), `burger_line_height`/`burger_line_radius` (unit values), `burger_line_color` — all map to CSS custom properties on `.x-hamburger-box`, not direct properties, if referencing them via `_cssCustom` elsewhere is ever needed.
- **Button chrome:** `button_bg` (applies to the root element itself), `burger_padding`.
- **Active-state colors:** `button_bg_active` (root, active when `aria-expanded="true"`), `burger_line_color_active` (`.x-hamburger-box.is-active`) — both reflect the burger's own click-toggle state, which the element's own click handler sets independently of whatever it's wired to open.
- **Optional button text:** `buttonText` (empty by default — icon-only unless set). Setting it unlocks `text_padding`, `flexDirection`, `buttonTextDisplay` (`block`/`none`) — all schema-gated behind `required: ["buttonText", "!=", ""]`, inert otherwise.
- **`aria_label`** (placeholder `"Open menu"`).

---

## `aria_label` is only written to the DOM when `buttonText` is empty — by design

`render()` only calls `set_attribute( '_root', 'aria-label', ... )` inside an `if ( '' === $button_text )` check — **setting `buttonText` (visible label text) omits the `aria-label` attribute entirely, even if `aria_label` itself is also set.** This is intentional, not a gap: an `aria-label` overrides an element's visible text for screen readers, so if both were written, `aria_label` could silently diverge from `buttonText` over time (someone updates the visible text, forgets the separate aria field) and a sighted user and a screen-reader user would end up hearing/seeing different labels for the same button. Omitting `aria-label` whenever real visible text exists means the accessible name always matches what's actually on the button, with no separate field to keep in sync. Don't set `aria_label` expecting it to reach the DOM when `buttonText` is also set — it only applies in icon-only mode.

## Rendered DOM (for custom CSS/targeting)

Icon-only (no `buttonText`):

```html
<button class="brxe-xburgertrigger" aria-label="Open menu">
  <span class="x-hamburger-box x-hamburger--arrow">
    <span class="x-hamburger-inner"></span>
  </span>
</button>
```

With `buttonText` set — note `aria-label` is gone (see above):

```html
<button class="brxe-xburgertrigger">
  <span class="x-hamburger-box x-hamburger--slider">
    <span class="x-hamburger-inner"></span>
  </span>
  <span class="x-hamburger-text">Menu</span>
</button>
```

The animation itself is pure CSS driven off the `x-hamburger--{variant}` class on `.x-hamburger-box` plus `.x-hamburger-box.is-active` once the click handler toggles that class (and `aria-expanded="true"` on the root) — `.x-hamburger-inner` is the single element the animation CSS actually transforms/pseudo-elements into the bars; there's no separate markup per bar.

## Build workflow

1. Write `burger_animation` and `aria_label` explicitly.
2. Add `_cssClasses` on the trigger; set the target element's own click-trigger field to that class (see `bricksextras-start-here`).
3. Add `buttonText` + its dependent fields only if text is wanted alongside the icon.
4. Verify in the browser: click the rendered button and confirm both the animation class is visibly applied (not a static icon) and the target actually opens.
