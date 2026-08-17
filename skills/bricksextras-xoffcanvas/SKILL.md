---
name: xoffcanvas
description: "Use only when editing a page that already has the OffCanvas (Template) element (xoffcanvas) from BricksExtras — a deprecated legacy element, superseded by xoffcanvasnestable. For any new build, use xoffcanvasnestable instead. Covers the one real difference: content comes from a selected Bricks Template (offcanvas_template), not from nested children."
---

**Requires:** BricksExtras 1.7.3+ with xoffcanvas enabled

# BricksExtras: OffCanvas (Template) (xoffcanvas) — deprecated

Shipped by the **BricksExtras** plugin. **Deprecated — superseded by `xoffcanvasnestable`.** Still fully registered and functional (not hidden or non-rendering), so an existing page can genuinely use it, but never choose this element for new work. Load `bricksextras-xoffcanvasnestable` first — every control, gotcha, and rendered-DOM detail documented there applies here identically, since both elements share the same underlying settings and JS behavior.

**The only real difference is how content gets into the panel.** `xoffcanvasnestable` builds content directly from nested children. `xoffcanvas` instead has an `offcanvas_template` field — a select of existing Bricks Templates — and renders that template's content inside `.x-offcanvas_inner` instead of any children of its own. `xoffcanvas` is not itself nestable.

**Before writing settings, read this element's schema JSON now** — open `references/elements/xoffcanvas.json` inside `bricksextras-element-schemas`, or fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xoffcanvas` if the bundle is missing or stale. Confirm `offcanvas_template` still exists and check what templates are actually available before assuming a name/id.

## When you'd actually touch this element

- An existing page already has an `xoffcanvas` instance and the task is to edit its content/styling/triggers — work with what's there rather than swapping it for the nestable version unprompted, since that would mean rebuilding the content as children and changing the underlying markup structure.
- Never add a new `xoffcanvas` instance from scratch — use `xoffcanvasnestable` instead, per its own skill.

## Build workflow

1. Confirm this is genuinely the deprecated `xoffcanvas`, not `xoffcanvasnestable` — check the element `name` in the page tree before assuming which one is present.
2. For content changes, `offcanvas_template` must point at a real, published Bricks Template id — look up available templates rather than guessing an id.
3. For every other setting (`clickTrigger`, `direction`/`offcanvas_width`/`offcanvas_height`, `disable_backdrop`, focus/accessibility options, etc.), follow `bricksextras-xoffcanvasnestable` exactly — the controls and their gotchas are identical between the two elements.

## Never do

- Don't add a new `xoffcanvas` instance for new work — always use `xoffcanvasnestable`.
- Don't assume content can be nested as children — this element has no `children` mechanism; content only comes from the selected template.
