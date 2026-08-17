---
name: xfluentform
description: "Use when embedding a Fluent Forms form (xfluentform) from BricksExtras: a thin wrapper around the [fluentform] shortcode, where every non-selection setting is CSS styling targeting Fluent Forms' own DOM output. Covers the form-selection modes, the required Fluent Forms plugin dependency, and that there's nothing to build beyond picking a form and styling it."
---

**Requires:** BricksExtras 1.7.3+ with xfluentform enabled, and the **Fluent Forms** plugin active

# BricksExtras: Fluent Form (xfluentform)

Shipped by the **BricksExtras** plugin. This element outputs the selected form via Fluent Forms' own `[fluentform id="..."]` shortcode, inside a wrapper div. Every one of its ~60 non-selection settings (typography/color/border/box-shadow/padding groups for inputs, submit button, checkboxes, GDPR text, checkable grids, file upload, progress bar, steps, net promoter score, success/error messages) is pure CSS targeting Fluent Forms' own generated markup (`.fluentform`, `.ff-el-group`, `.ff-btn-submit`, etc.) — there is no BricksExtras-side logic to any of them beyond writing that CSS rule. Don't treat this as a form-builder element; it's a re-skinning wrapper around someone else's shortcode.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xfluentform.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xfluentform` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Hard dependency: Fluent Forms must be installed, active, and have a real form

`render()` checks `function_exists('wpFluentForm')` and renders a "Not found (FluentForm)" placeholder if it's missing — confirm the plugin is active before building with this element. The `fluentFormSelect` dropdown's `options` are populated live from `wpFluent()->table('fluentform_forms')` — i.e. actual forms that exist in Fluent Forms' own database on this site, not a fixed list. If no forms exist yet, the only option is the literal placeholder `noforms`. Create the form in Fluent Forms first (or via its own admin) before trying to select it here.

## Two mutually exclusive ways to pick a form

`formSource` (checkbox) toggles which of two fields is live:
- **Off (default):** `fluentFormSelect` — a dropdown of real form IDs/titles from the database, schema `default: "1"`.
- **On:** `fluentFormID` — a manual text field for the form ID, which also accepts a dynamic-data tag (the render code checks for `{` and resolves it via `render_dynamic_data_tag` if present) — useful for CPT-per-form setups.

Only set one; the other is schema-gated inert (`required: ["formSource", ...]`) and ignored by `render()` regardless of what's in it.

## Other settings

- **`smartUI`** (checkbox): enables BricksExtras' own custom checkbox/radio visual replacement (`data-x-fluent-form` attribute carries this flag to the frontend JS/CSS) — without it, the `radioCheckbox*` styling controls that target `[data-x-fluent-form*=smartUI]`-scoped selectors have no effect, since that attribute value won't be present.
- Every other control is a direct CSS mapping to a specific Fluent Forms selector — style what's visually present after picking a real form, rather than guessing which control affects which part sight-unseen.

## Build workflow

1. Confirm Fluent Forms is active and has at least one real form.
2. Pick the form via `fluentFormSelect` (or `fluentFormID` + `formSource: true` for a dynamic/manual ID).
3. Enable `smartUI` first if custom checkbox/radio styling is wanted — the relevant color/border controls are inert without it.
4. Style the rest by rendering the page and matching visible Fluent Forms elements to their corresponding control — there's no shortcut around this given the sheer number of theming controls.
