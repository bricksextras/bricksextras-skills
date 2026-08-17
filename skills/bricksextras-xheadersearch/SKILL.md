---
name: xheadersearch
description: "Use when building or debugging the Header Search element (xheadersearch) from BricksExtras: a toggle button that reveals a search form (header-overlay, below-header, fullscreen, or inline-expand layouts). Covers the four layout modes, the rendered DOM for custom styling, and the icon-fields-have-no-runtime-default gotcha that produces an invisible toggle button if skipped."
---

**Requires:** BricksExtras 1.7.3+ with xheadersearch element enabled

# BricksExtras: Header Search (xheadersearch)

Shipped by the **BricksExtras** plugin. Nestable, and typically placed inside a header template row (e.g. alongside `xheaderrow`'s logo/nav). Renders a toggle button that reveals a search form in one of four layouts.

**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xheadersearch.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xheadersearch` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## `layout` — four genuinely different reveal behaviors

| `layout` | Behavior |
|---|---|
| `header_overlay` (default) | Form reveals as an overlay across/near the header |
| `below_header` | Form slides or fades open in a panel below the header (`belowAnimation`: `slide`/`fade`, `belowHeaderHeight` controls the panel height) |
| `full_screen` | Form opens as a full-viewport overlay, centered |
| `expand` | Form expands inline in place (`expandWidth` controls the expanded width) — no full-screen/overlay chrome |

Only `header_overlay` and `below_header` expose the **Live search** control group (see below) — `full_screen` and `expand` don't support nesting a live-search filter.

## Icon fields have no runtime default — set them explicitly

`search_icon` and `close_icon` both show a schema `default` (`{"library":"themify","icon":"ti-search"}` / `ti-close`), but **that default is a builder-UI pre-fill only, not applied at render time when settings are written directly.** Omitting them produces a toggle button that renders with no icon inside — an invisible, unlabeled clickable area. Same pattern already known from `xbacktotop`/`xbeforeafterimage` — always write icon-type settings explicitly, never rely on the schema `default` alone.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xheadersearch.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xheadersearch",
  "settings": {
    "layout": "full_screen",
    "search_icon": { "library": "themify", "icon": "ti-search" },
    "close_icon": { "library": "themify", "icon": "ti-close" }
  }
}
```

This is a general rule for the whole element, not just these two fields — any icon-object setting here (or elsewhere in BricksExtras) needs an explicit value if you want it to render at all.

## Rendered DOM (for custom CSS/targeting)

Captured live, `full_screen` layout, in its open state:

```html
<div id="brxe-{id}" class="brxe-xheadersearch" data-type="full_screen" data-close-label="Close Search">
  <button class="x-header-search_toggle-open" data-type="full_screen" data-reveal="slide"
          aria-label="Open search" aria-controls="x-header-search_form-{id}" aria-expanded="true" tabindex="-1">
    <i class="ti-search"></i>
  </button>
  <form role="search" autocomplete="off" method="get" class="x-search-form" id="x-header-search_form-{id}" action="https://example.com/">
    <div data-search-width="contentWidth" class="brxe-container">
      <label>
        <span class="screen-reader-text">Search</span>
        <input type="search" placeholder="Search..." value="" name="s">
      </label>
      <input type="submit" class="search-submit x-search-submit x-submit-hidden" value="Search" tabindex="-1" aria-hidden="true">
      <button class="x-header-search_toggle-close" aria-label="Close Search" aria-controls="x-header-search_form-{id}" aria-expanded="true">
        <i class="ti-close"></i>
      </button>
    </div>
  </form>
</div>
```

Notes on this structure:

- `data-type` on both the root and the open-toggle button mirrors the `layout` setting — use `[data-type=full_screen]` etc. as a CSS hook if styling needs to differ per layout.
- `data-reveal` on the open-toggle button carries `belowAnimation`'s value (`slide`/`fade`) regardless of the active `layout` — it's only functionally used in `below_header` layout, but the attribute is present either way.
- `aria-expanded`/`aria-controls` link both toggle buttons to the form's `id` (`x-header-search_form-{id}`) — the open/close buttons are siblings of the form, not nested inside a shared wrapper with it, except the close button which lives *inside* `.x-search-form > .brxe-container` alongside the input.
- The submit `<input>` always renders (even with `showSubmitButton` left unset/false) but gets `x-submit-hidden` — a CSS-hide class, not a conditional-render omission. If you need to guarantee it's gone (not just visually hidden), don't rely on the setting alone; check the rendered class.
- Unlike the icon fields, `showSubmitButton` (a checkbox) degrades safely when omitted — it just renders hidden via the `x-submit-hidden` class rather than breaking. The no-default problem is specific to icon-object controls, not every setting on this element.

## Live search (nestable mode) — this element is just the open/close chrome

`maybeLiveSearch` ("Live search (Nestable)") switches the element into a fundamentally different role. With it on, `xheadersearch` **stops being a self-contained search form** — several standard fields (`screenReaderLabel`, `placeholder`, `showSubmitButton`, `autoComplete`, `actionURL`, `additionalParams`) hide, and `maybeEnterKeyRedirect` becomes available instead. Only relevant for `header_overlay`/`below_header` layouts (the `liveSearch` control group requires one of those two — `full_screen`/`expand` don't support it).

In this mode, `xheadersearch` only manages the open/close *interaction* of the toggle button. The actual search input and results are Bricks' own **native Query Filter (`filter-search`) + query-loop live-search feature**, nested as direct children — not anything specific to this element's own settings:

```
xheadersearch (maybeLiveSearch: true, layout: "below_header")
├── filter-search                          ← Bricks core Query Filter element
│     settings.filterQueryId: "{loopId}"   ← points at the loop block's own id, same as any Query Filter binding
└── block                                  ← results wrapper — this block's own #brxe-{id} is the live-search AJAX swap target
      └── container
            └── block                      ← the actual query loop
                  settings.hasLoop: true
                  settings.query.is_live_search: true
                  settings.query.is_live_search_wrapper_selector: "#brxe-{wrapperBlockId}"  ← CSS selector of the results-wrapper block above
                  └── (loop content, e.g. post-title)
```

Two things make this work, both **outside `xheadersearch`'s own schema** — on the query-loop block itself:

- `hasLoop: true` + `query.is_live_search: true` — turns on live (AJAX, as-you-type) search for that loop.
- `query.is_live_search_wrapper_selector` — a CSS selector (`#brxe-{id}`) identifying the ancestor block whose contents get swapped when live-search results come back. It does **not** have to be the loop block itself — in the structure above it's the loop's own *grandparent* block, one level up from the intermediate `container`.

And on the `filter-search` child: just `filterQueryId` set to the loop block's id — the same binding mechanism as any other Bricks Query Filter element.

**Practical implication:** if live search isn't working, don't debug it as an `xheadersearch` settings problem — check the loop block's `query.is_live_search`/`is_live_search_wrapper_selector` and the `filter-search` child's `filterQueryId`, the same way you'd debug any other Bricks Query Filter setup.

## Never do

- Don't omit `search_icon`/`close_icon` expecting the schema `default` to apply — it won't, and the toggle button silently renders with no visible icon/clickable affordance.
- Don't expect `full_screen` or `expand` layouts to expose live-search nesting — that's gated to `header_overlay`/`below_header` only.
- Don't assume the submit button is absent just because `showSubmitButton` is unset — it still renders in the DOM with a hide class; style/target accordingly if that distinction matters.
- Don't treat "element not found" for `xheadersearch` as an MCP problem before checking whether the element is disabled at the plugin level.
