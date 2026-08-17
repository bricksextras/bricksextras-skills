---
name: xpromodal
description: "Use only when editing a page that already has the Modal (Template) element (xpromodal) from BricksExtras — a deprecated legacy element, superseded by xpromodalnestable. For any new build, use xpromodalnestable instead. Covers the one real difference: content comes from a selected Bricks Template or a wysiwyg field (modal_content), not from nested children."
---

**Requires:** BricksExtras 1.7.3+ with xpromodal enabled

# BricksExtras: Modal (Template) (xpromodal) — deprecated

Shipped by the **BricksExtras** plugin, built on the same third-party **MicroModal.js** library as its successor. **Deprecated — superseded by `xpromodalnestable`.** Still fully registered and functional, so an existing page can genuinely use it, but never choose this element for new work. Load `bricksextras-xpromodalnestable` first — the trigger repeater, `disableFocus`'s inverted options, the `data-x-modal-close` mechanism, the two separate aria-label settings, and every other control/gotcha documented there applies here identically, since both elements share the same underlying settings and JS behavior.

**The only real difference is how content gets into the modal.** `xpromodalnestable` builds content directly from nested children. `xpromodal` instead has `modal_content`, a select with two modes: `template` (an existing Bricks Template, chosen via `modal_template`) or `wysiwyg` (an inline rich-text field, `modal_wysiwyg`, with a real default placeholder value). `xpromodal` is not itself nestable — its content always comes from one of those two fields, never from children.

**Before writing settings, read this element's schema JSON now** — open `references/elements/xpromodal.json` inside `bricksextras-element-schemas`, or fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xpromodal` if the bundle is missing or stale. Confirm `modal_content`'s options and check what templates are actually available before assuming a name/id.

## When you'd actually touch this element

- An existing page already has an `xpromodal` instance and the task is to edit its content/styling/triggers — work with what's there rather than swapping it for the nestable version unprompted, since that would mean rebuilding the content as children and changing the underlying markup structure.
- Never add a new `xpromodal` instance from scratch — use `xpromodalnestable` instead, per its own skill.

## Build workflow

1. Confirm this is genuinely the deprecated `xpromodal`, not `xpromodalnestable` — check the element `name` in the page tree before assuming which one is present.
2. For content changes: if `modal_content: "template"`, `modal_template` must point at a real, published Bricks Template id — look up available templates rather than guessing. If `modal_content: "wysiwyg"`, edit `modal_wysiwyg` directly (it holds real HTML, with a default placeholder value if unset).
3. For every other setting (`triggers` repeater, `disableFocus`, `maybe_remove_close`, positioning, animation, `clickBackdropClose`, etc.), follow `bricksextras-xpromodalnestable` exactly — the controls and their gotchas are identical between the two elements.

## Never do

- Don't add a new `xpromodal` instance for new work — always use `xpromodalnestable`.
- Don't assume content can be nested as children — this element has no `children` mechanism; content only comes from `modal_template` or `modal_wysiwyg`, gated by `modal_content`.
