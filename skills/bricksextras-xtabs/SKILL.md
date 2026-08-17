---
name: xtabs
description: "Use when building, debugging, or styling Pro Tabs (xtabs) from BricksExtras: tabbed content panels, FAQ tabs, pricing/feature tab groups, tabs that collapse into an accordion on mobile. Covers required hidden per-item structure, CSS classes, attributes, and the built-in accordion breakpoint."
---

**Requires:** BricksExtras 1.7.3+ with xtabs element enabled

# BricksExtras: Pro Tabs (xtabs)

Shipped by the **BricksExtras** plugin. A nestable container element — each tab is two paired subtrees nested inside it (a tab-list item plus a matching content item), not a repeater field on the parent. There is no auto-inserted default item when created empty via MCP — every tab has to be constructed to match the exact shape below or the element renders as static, non-interactive content (or nothing at all).


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xtabs.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xtabs` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Built-in mobile accordion — no custom logic needed

`xtabs` natively collapses into an accordion below a chosen breakpoint via the `accordionBreakpoint` setting (`478` / `767` / `991` / `1279` / `none`, default `767`). Below that width the horizontal tab list disappears entirely and each tab's title becomes a clickable accordion toggle (`.x-tabs_toggle`) that expands its own panel in place. This is a single setting, not something to hand-build — if a person asks for "tabs that turn into an accordion on mobile," that is `xtabs`'s default behavior, not a feature to construct.

At a mobile viewport, the `.x-tabs_list` tab bar is not present in the accessibility tree at all — only the accordion toggle buttons are, one per tab, each independently expandable.

---

## Critical: the hidden structure is the mechanism, not decoration

Like the accordion, this plugin's JS binds behavior by **CSS class**, not by element type or position. Missing or misplaced classes render fine visually but respond to nothing:

| Piece | Element | Required hidden class / attrs | Notes |
|---|---|---|---|
| Tabs list wrapper | `block` | `x-tabs_list`, `tag: ul`, `_attributes: [{role: tablist}]` | One per `xtabs`, direct child. `cloneable: false`, `deletable: false` in the native structure |
| Tab item | `div` | `x-tabs_tab`, `tag: li`, `_attributes: [{role: tab}]` | One per tab, child of the list wrapper |
| Tab title | `text-basic` (tag `span`) | *(none)* | Child of the tab item — the visible label |
| Tabs content wrapper | `block` | `x-tabs_content`, `tag: div` | Sibling of the list wrapper, direct child of `xtabs`. `cloneable: false`, `deletable: false` |
| Tab content item | `block` | `x-tabs_content-item`, `tag: div` | One per tab, child of the content wrapper — paired 1:1 in order with a tab item |
| Accordion toggle | `block` | `x-tabs_toggle`, `_attributes: [{role: button}, {tabindex: 0}]` | The mobile-accordion clickable header for this item. `cloneable: false` |
| Toggle title | `text-basic` (tag `span`) | *(none)* | Child of the toggle — usually duplicates the tab title text |
| Toggle icon | `icon` | `x-tabs_toggle-icon` | Default icon `ion-ios-arrow-down` / `ionicons`, `iconSize: "1em"` — rotates via the element's own `iconTransform`/`iconTransformActive` settings, not a per-icon setting |
| Tab panel | `block` | `x-tabs_panel`, `_attributes: [{role: tabpanel}]` | Sibling of the toggle, inside the content item. `cloneable: false`, `deletable: false` |
| Panel content | `block` | `x-tabs_panel-content` | Wraps the actual content. `cloneable: false`, `deletable: false` |
| Content | any (`heading`, `text`, etc.) | *(none)* | Real content — this is the only piece with no required class |

**Tab items and tab content items must stay in the same order** — the Nth `.x-tabs_tab` controls the Nth `.x-tabs_content-item`, matched by DOM position, not by an id/ref pairing.

The classed pieces above are fixed; everything else is free to customize — swap the toggle icon, add extra content next to a tab title, or put whatever's needed inside `x-tabs_panel-content` (not just one element). Keep the class/element-type/position of the pieces in the table intact; the "exact shape" requirement below is about that skeleton, not about the actual content going in each slot.

---

## Canonical per-tab template (example — verify against schema)

**This JSON is an example, not the schema.** It shows the required skeleton (which pieces nest where, which classes go on which) but individual setting names/values (e.g. `tabActiveBackgroundColor`, `accordionBreakpoint`) may be incomplete or stale relative to the current element version. Before building from this, read `references/elements/xtabs.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

Build the whole `xtabs` — list wrapper, content wrapper, and every tab's paired subtrees — in one nested call:

```json
{
  "name": "xtabs",
  "settings": {
    "tabActiveBackgroundColor": { "hex": "#dddedf" },
    "accordionBreakpoint": 767,
    "iconTransform": { "rotateX": "0deg" },
    "iconTransformActive": { "rotateX": "180deg" }
  },
  "children": [
    {
      "name": "block",
      "label": "Tabs list",
      "settings": {
        "tag": "ul",
        "_display": "flex",
        "_direction": "row",
        "_hidden": { "_cssClasses": "x-tabs_list" },
        "_attributes": [{ "name": "role", "value": "tablist" }]
      },
      "children": [
        {
          "name": "div",
          "label": "Tab Item",
          "settings": {
            "tag": "li",
            "_hidden": { "_cssClasses": "x-tabs_tab" },
            "_attributes": [{ "name": "role", "value": "tab" }]
          },
          "children": [
            { "name": "text-basic", "label": "Title", "settings": { "tag": "span", "text": "Item 1" } }
          ]
        }
      ]
    },
    {
      "name": "block",
      "label": "Tabs Content",
      "settings": { "tag": "div", "_hidden": { "_cssClasses": "x-tabs_content" } },
      "children": [
        {
          "name": "block",
          "label": "Tab Content Item",
          "settings": { "tag": "div", "_hidden": { "_cssClasses": "x-tabs_content-item" } },
          "children": [
            {
              "name": "block",
              "label": "Accordion Toggle",
              "settings": {
                "_hidden": { "_cssClasses": "x-tabs_toggle" },
                "_attributes": [{ "name": "role", "value": "button" }, { "name": "tabindex", "value": "0" }]
              },
              "children": [
                { "name": "text-basic", "settings": { "tag": "span", "text": "Item 1" } },
                {
                  "name": "icon",
                  "label": "Toggle icon",
                  "settings": {
                    "_hidden": { "_cssClasses": "x-tabs_toggle-icon" },
                    "icon": { "icon": "ion-ios-arrow-down", "library": "ionicons" },
                    "iconSize": "1em"
                  }
                }
              ]
            },
            {
              "name": "block",
              "label": "Tab Panel",
              "settings": {
                "_hidden": { "_cssClasses": "x-tabs_panel" },
                "_attributes": [{ "name": "role", "value": "tabpanel" }]
              },
              "children": [
                {
                  "name": "block",
                  "label": "Tab Content",
                  "settings": { "_hidden": { "_cssClasses": "x-tabs_panel-content" } },
                  "children": [
                    { "name": "heading", "settings": { "tag": "h4", "text": "Item 1" } },
                    { "name": "text", "settings": { "text": "Tab content here" } }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

Repeat the tab-item block (inside the list) and its paired tab-content-item block (inside the content wrapper) once per tab, keeping order in sync between the two.

---

## Query loop for dynamic tabs

**The JSON below is an example, not the schema.** It illustrates the required pattern (two independent `hasLoop` sites, matching queries) but setting names/values may be incomplete or stale. Check `references/elements/xtabs.json` (or the live schema ability) before building from it.

Unlike most nestable elements, a tab has **two separate DOM subtrees that both need to loop independently** — the `x-tabs_tab` list item and its paired `x-tabs_content-item` panel live in different parent wrappers (`x-tabs_list` vs `x-tabs_content`), so one `hasLoop` can't drive both. Build a **single** tab item and a **single** matching content item (instead of hand-building N pairs), then put `hasLoop: true` on **both** of them independently — not on `xtabs` itself, and not on just one side:

```json
{
  "name": "div", "label": "Tab Item",
  "settings": {
    "tag": "li",
    "_hidden": { "_cssClasses": "x-tabs_tab" },
    "_attributes": [{ "name": "role", "value": "tab" }],
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": ["post"], "posts_per_page": 10 }
  },
  "children": [ { "name": "text-basic", "settings": { "tag": "span", "text": "{post_title}" } } ]
}
```

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xtabs.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block", "label": "Tab Content Item",
  "settings": {
    "tag": "div",
    "_hidden": { "_cssClasses": "x-tabs_content-item" },
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": ["post"], "posts_per_page": 10 }
  },
  "children": [
    {
      "name": "block", "label": "Accordion Toggle",
      "settings": {
        "_hidden": { "_cssClasses": "x-tabs_toggle" },
        "_attributes": [{ "name": "role", "value": "button" }, { "name": "tabindex", "value": "0" }]
      },
      "children": [
        { "name": "text-basic", "settings": { "tag": "span", "text": "{post_title}" } },
        { "name": "icon", "label": "Toggle icon", "settings": { "_hidden": { "_cssClasses": "x-tabs_toggle-icon" }, "icon": { "icon": "ion-ios-arrow-down", "library": "ionicons" }, "iconSize": "1em" } }
      ]
    },
    { "name": "block", "label": "Tab Panel", "settings": { "_hidden": { "_cssClasses": "x-tabs_panel" }, "_attributes": [{ "name": "role", "value": "tabpanel" }] }, "children": [ /* panel-content → heading/text, same as the static template */ ] }
  ]
}
```

**Both loop sites must run the same query, in the same order, to stay paired.** They aren't linked to each other by id or reference — the Nth generated tab only matches the Nth generated content item because both independently loop identically. Use the exact same `query` object (or the exact same omitted-query default, see below) on both, or the tab labels and their panels will drift out of sync.

**`hasLoop` only duplicates the block — it does not make child content dynamic on its own.** Fields left as plain strings repeat identically for every looped item; only fields explicitly set to a dynamic tag (`{post_title}`, `{post_excerpt}`, etc.) vary per iteration.

**The tab title and the toggle title are two separate `text-basic` elements — a dynamic tag on one does not apply to the other.** `x-tabs_tab`'s title (in the list wrapper) and `x-tabs_toggle`'s title (in the content item) are not the same element and don't share state; each needs the dynamic tag set independently, same as with the two `hasLoop`/`query` pairs above. **This one matters more than it looks:** below `accordionBreakpoint`, the `x-tabs_list` tab bar is not rendered at all — the toggle title is the *only* visible label on mobile. Setting `{post_title}` on the tab title but leaving the toggle title as static placeholder text (`"Item 1"`, etc.) renders with no error on desktop, then shows the wrong/generic label for every item once the page collapses to the mobile accordion.

---

## Active tab needs a visible state — don't ship it with none

Without some active-state styling, there's no way for a visitor to tell which tab is selected — the tab list and the panel content are visually silent on their own. `tabActiveBackgroundColor` (schema default `#dddedf`) is the most direct way to provide that, but it isn't the only valid one: `animatedTabs` + `animationSlideBackgroundColor` (sliding highlight), `tabActiveBorder`, or `tabActiveTypography` (bold/color change on the label) all satisfy the same requirement. Which one fits is a design decision — this isn't "always set `tabActiveBackgroundColor` to its default," it's "make sure *something* distinguishes the active tab," per the design actually being built.

The failure mode worth flagging: because `tabActiveBackgroundColor` has a schema `default`, it's easy to assume it's applied automatically and skip it entirely when writing JSON directly (see `bricksextras-start-here`'s "Schema defaults are UI-only" rule) — a human adding this element in the builder gets the default background pre-filled and never notices it's a setting at all. Skipping it when authoring JSON produces tabs that work (click, switch, accordion collapse all function) but look broken — no visual indication of which panel is open. Confirm before shipping that some active-state treatment is present, whether that's the default background, a custom color, or one of the alternatives above.

---

## Other settings worth knowing

- **`tabOrientation`**: `horizontal` (default) or `vertical` — only changes keyboard nav (Up/Down vs Left/Right), not layout. For an actual vertical tab layout, style `.x-tabs_list` (`_direction: column`) yourself.
- **`adaptiveHeight`**: animates the content area's height to match the active panel instead of a fixed/jumping height. Pair with `adaptiveHeightDuration` (default `300ms`).
- **`animatedTabs`**: adds a sliding highlight (`.x-tabs_slider`) behind the active tab instead of an instant background swap. Style it via `animationSlideBackgroundColor`/`animationSlideBorder`.
- **`animateTabContent`**: fade/slide transition for panel content on switch (`fadein_x`, `fadeinup_x`, `fadeindown_x`, `fadeinleft_x`, `fadeinright_x`, or none). Controlled separately from `animatedTabs`.
- **`hashLink` vs `maybeURLparam` are mutually exclusive** (each is `required` on the other being off) — `hashLink` opens/scrolls to a tab matching a URL hash (`#panel-id`); `maybeURLparam` does the same via a query param (`?tab=panel-id`, key configurable via `URLParamKey`). Pick one, not both.
- **`hoverSelect`**: switches tabs on hover instead of click — usually undesirable for FAQ-style content, more common on feature-showcase tabs.
- **`tabEqualWidth`**: checkbox, makes all tabs flex-grow equally instead of sizing to their label.

---

## Never do

- Do not omit any of the required hidden classes (`x-tabs_list`, `x-tabs_tab`, `x-tabs_content`, `x-tabs_content-item`, `x-tabs_toggle`, `x-tabs_toggle-icon`, `x-tabs_panel`, `x-tabs_panel-content`) — each one is a functional JS/CSS hook, not styling.
- Do not build a custom mobile-accordion fallback (a separate `xproaccordion`, conditional visibility, breakpoint-specific element swaps, etc.) — `accordionBreakpoint` already does this natively.
- Do not let tab items and tab content items drift out of matching order — pairing is positional (Nth tab controls Nth content item), not by id.
- Do not use a `heading` element for tab titles or toggle titles — both are `text-basic` rendered as `span`, matching the accordion's title convention.
- Do not set both `hashLink` and `maybeURLparam` — they are mutually exclusive URL-open mechanisms.
- Do not put `hasLoop` on only one side of a query-loop tab (just the tab item, or just the content item) — both loop sites need it, with matching queries, or tabs and panels desync.
- Do not set a dynamic tag on the tab title without also setting it on the toggle title — they're separate elements, and the toggle title is the only one visible once the mobile accordion is active.

## MCP write notes

- Check the bundled schema first per `bricksextras-element-schemas`; only if the bundle is missing or stale, call whatever live schema ability the current MCP connection exposes — don't rely on memory either way.
- The whole `xtabs` — list wrapper, content wrapper, and every tab's paired subtrees — can be written in a single nested `add-element`/`set-page-elements` call.
- When verifying, confirm all required classes are present in the DOM, that clicking a tab switches its panel, and — if `accordionBreakpoint` is set — that resizing below it collapses the tab list into working accordion toggles.