---
name: xwsforms
description: "Use when embedding a WS Form (xwsforms) from BricksExtras: a thin wrapper around the [ws_form] shortcode, where every non-selection setting is CSS styling targeting WS Form's own DOM output. Covers the form-selection modes, the required WS Form plugin dependency, and WS Form's own Styler feature changing which selectors matter."
---

**Requires:** BricksExtras 1.7.3+ with xwsforms enabled, and the **WS Form** plugin active

# BricksExtras: WS Forms (xwsforms)

Shipped by the **BricksExtras** plugin. This element does nothing but `do_shortcode('[ws_form id="..." element_id="..."]')` inside a wrapper div. The large majority of its settings (form-wide color roles, input/label/help styling, prefix/suffix, buttons, checkbox/radio, phone input incl. the international dropdown, repeatable sections, range slider, meter/progress, tabs, file upload, password, legal text, star rating) are pure CSS targeting WS Form's own generated markup (`.wsf-form`, `.wsf-field`, `button.wsf-button`, etc.) — no BricksExtras-side logic beyond writing that CSS. Same category of element as `xfluentform` — a re-skinning wrapper around someone else's shortcode, not a form builder.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xwsforms.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xwsforms` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Hard dependency: WS Form must be installed and active

`render()` checks `defined('WS_FORM_NAME')` and renders a "Not found (WS Form)" placeholder if missing. `formSelect`'s options are populated live via `WS_Form_Common::get_forms_array()` — real forms from WS Form's own data, not a fixed list. If `$form_id` resolves to `0` or less (nothing selected), it renders a "No form selected" placeholder instead of the shortcode.

## Two mutually exclusive ways to pick a form — same pattern as xfluentform

`formSource` (checkbox) toggles:
- **Off (default):** `formSelect` — dropdown of real forms.
- **On:** `formID` — manual text field, resolves dynamic-data tags (`{...}`) via `render_dynamic_data_tag` if present, same as `xfluentform`'s `fluentFormID`.

## WS Form's own "Form Styler" changes which selectors actually apply

Several color controls (`generalPrimaryColor`, and the enqueued stylesheet itself) branch on `WS_Form_Common::styler_enabled()`. If WS Form's own Styler feature is active on the site, extra selectors get merged into some controls' `css` arrays (e.g. direct checkbox/radio background-color rules), and a different base stylesheet is enqueued entirely (`wsforms-styler-enabled.css` vs `wsforms.css`). **If a color control doesn't seem to visually apply, check whether WS Form's Styler is enabled on this site** — that's a real variable in what actually renders, not a guess.

## Other settings

- **General color roles** (`generaldefaultColor`, `generaldefaultInvertedColor`, `generalLightColor`, `generalLighterColor`, `generalLightestColor`, `generalPrimaryColor`, `generalSecondaryColor`, `generalSuccessColor`, `generalInformationBg`, `generalWarningColor`, `generalDangerColor`) each map to a *large, specific* set of WS Form selectors (visible directly in the PHP, dozens of selectors per color) — these are deliberately broad "theme role" colors, described in their own `description` text (e.g. "Labels and field values", "Checkboxes, radios, range sliders, progress bars, and submit buttons"). Set these first for overall theming before reaching for the more granular per-component controls.
- Everything else is a direct CSS mapping to a specific WS Form selector, same as `xfluentform` — style what's visually present after picking a real form.

## Build workflow

1. Confirm WS Form is active and has at least one real form.
2. Pick the form via `formSelect` (or `formID` + `formSource: true`).
3. Check whether WS Form's own Styler is enabled if colors don't seem to be applying as expected.
4. Set the broad `general*Color` roles first, then style specific components by rendering the page and matching visible elements to controls.
