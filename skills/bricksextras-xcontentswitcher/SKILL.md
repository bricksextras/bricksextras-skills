---
name: xcontentswitcher
description: "Use when building or debugging the Content Switcher element (xcontentswitcher) from BricksExtras: a container that shows one child block at a time, switched by an external xtoggleswitch. Covers the required hidden class on each direct child and why childrenBlocks isn't the real mechanism."
---

**Requires:** BricksExtras 1.7.3+ with xcontentswitcher element enabled

# BricksExtras: Content Switcher (xcontentswitcher)

Shipped by the **BricksExtras** plugin. Nestable — each direct child is one "panel" (e.g. Monthly pricing / Yearly pricing), shown/hidden by index. Typically paired with an **`xtoggleswitch`** element elsewhere on the page/section, which drives which panel is active.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcontentswitcher.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xcontentswitcher` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: every direct child needs the `x-content-switcher_block` hidden class

The switching JS (`toggleswitch.js`) selects direct children purely by this class and indexes them in DOM order — it does not read any structural/settings data from the element tree itself. A child block missing this class is invisible to the switcher (never shown, never hidden — it just doesn't participate):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcontentswitcher.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xcontentswitcher",
  "children": [
    {
      "name": "block",
      "label": "Monthly",
      "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } },
      "children": [ /* pricing content */ ]
    },
    {
      "name": "block",
      "label": "Yearly",
      "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } },
      "children": [ /* pricing content */ ]
    }
  ]
}
```

Panel order in the tree is the panel order the switcher cycles through — the first `x-content-switcher_block` child is index 0, and so on.

**Note:** the builder's `childrenBlocks` repeater setting (visible in the element's Content tab) is UI bookkeeping only — the server-side render never reads it; children render normally and the client-side JS scans for the class. Don't try to set `childrenBlocks` via the API expecting it to create or configure panels — it does nothing at render time. The class on each child is the entire mechanism.

---

## Pairing with xtoggleswitch

`xtoggleswitch` is a separate element (a visual toggle/switch control) that needs to target this content switcher the same way other cross-element pairings do in BricksExtras — via a class + selector field, not `_cssId`. See `bricksextras-start-here`'s "Targeting another element" section for the general pattern, and check `xtoggleswitch`'s own live schema for its specific selector-field name before wiring the pairing up.

---

## Granular pattern: switch individual elements inside a static structure, not just whole tab-style panels

`xcontentswitcher` doesn't have to wrap an entire "page" of content per option (the classic tabs-style layout where each panel is a full alternate block). It's just as valid — and often the better fit — to use several small `xcontentswitcher` instances, each wrapping only the one or two elements that actually change, inside an otherwise completely static layout.

Example: a pricing block with a shared heading, a shared "Pricing" label, and a shared Terms & Conditions line, where only the feature list and the price/CTA genuinely differ between Monthly/Yearly/Lifetime:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcontentswitcher.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block",
  "children": [
    { "name": "heading", "settings": { "text": "Everything you need" } },
    {
      "name": "xcontentswitcher",
      "settings": { "_cssClasses": "pricing-switcher" },
      "children": [
        { "name": "block", "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } }, "children": [ /* monthly feature list only */ ] },
        { "name": "block", "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } }, "children": [ /* yearly feature list only */ ] },
        { "name": "block", "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } }, "children": [ /* lifetime feature list only */ ] }
      ]
    },
    { "name": "heading", "settings": { "text": "Pricing" } },
    {
      "name": "xcontentswitcher",
      "settings": { "_cssClasses": "pricing-switcher" },
      "children": [
        { "name": "block", "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } }, "children": [ /* monthly price + CTA only */ ] },
        { "name": "block", "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } }, "children": [ /* yearly price + CTA only */ ] },
        { "name": "block", "settings": { "_hidden": { "_cssClasses": "x-content-switcher_block" } }, "children": [ /* lifetime price + CTA only */ ] }
      ]
    },
    { "name": "text-basic", "settings": { "text": "Cancel anytime. See our Terms & Conditions." } }
  ]
}
```

The heading, "Pricing" label, and T&C line sit **outside** both switchers entirely and never re-render on toggle. Each switcher's own panels can be as small as a single wrapped element (a price `heading`, a `text-basic` feature list) — there's no requirement that a panel represent a whole visual "page."

Both `xcontentswitcher` instances above share the identical class (`pricing-switcher`) and are driven by **one single `xtoggleswitch`** pointed at that shared selector — see `bricksextras-xtoggleswitch`'s "One toggle can drive multiple switchers at once" section. Clicking one toggle option updates every `xcontentswitcher` sharing the class simultaneously, in sync.

Use this granular pattern whenever a design brief describes shared/common elements (a heading, legal text, a static label) alongside only a few genuinely-variable values — don't default to duplicating the entire block per option just because that's the more obvious "tabs" shape.

---

## Rendered DOM and the real visibility mechanism (CSS attribute selectors, not JS display toggling)

```html
<div class="brxe-xcontentswitcher your-class x-content-switcher" data-x-id="{id}">
  <div class="x-content-switcher_content" data-x-switcher="1">
    <div class="brxe-block x-content-switcher_block" id="{id}_0">...</div>
    <div class="brxe-block x-content-switcher_block" id="{id}_1">...</div>
  </div>
</div>
```

**JS never toggles panel visibility directly — it only sets/removes an attribute or class, and `contentswitcher.css` does the actual showing/hiding via attribute selectors.** This is worth understanding precisely, since it explains a real hard limit:

- **`double` mode**: no `data-x-switcher` attribute in the default state → CSS shows child 1, hides child 2 (`:not([data-x-switcher]) > *:nth-child(2)`). Clicking toggles a `.x-content-switcher_toggled` class on `.x-content-switcher_content` (JS), which flips a second pair of CSS rules to show child 2 / hide child 1 instead. Binary only — there's no third state.
- **`multiple` mode**: JS sets `data-x-switcher="{labelIndex}_{toggleId}"` (1-based index) on click, and CSS has one hardcoded attribute-prefix selector per index, `[data-x-switcher^="1_"]` through `[data-x-switcher^="20_"]`, each hiding every panel except the matching `nth-child`. **This is a real, hardcoded 20-panel cap** — a 21st `x-content-switcher_block` has no matching CSS rule to hide it, so it stays visible alongside whichever panel is actually selected. For a query-loop-driven `multiple` switcher, keep the loop's result count to 20 or fewer.
- **The literal `data-x-switcher="1"` seen in server-rendered HTML (as shown above) is a separate, exact-match CSS rule** (`[data-x-switcher="1"]`, not the `^="1_"` prefix rule) — this is the PHP-written default state (panel 1 visible) that exists before any real `xtoggleswitch` interaction has run. Don't confuse it with the JS-computed `"N_id"` runtime value multiple mode sets after wiring up to a real toggle.

## Never do

- Do not omit `x-content-switcher_block` on any direct child — it's the only thing that makes a child a switchable panel.
- Do not set `childrenBlocks` expecting it to define panel content or count — it's not read at render time.

## MCP write notes

- A child missing the required class produces a "valid" write with no visible error, and only surfaces as a panel that never appears or never switches.