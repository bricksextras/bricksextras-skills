---
name: xtableofcontents
description: "Use when building or debugging the Table of Contents element (xtableofcontents) from BricksExtras: a tocbot-powered auto-generated heading list. Covers that contentSelector has no safe empty fallback (unlike the similarly-named field on xreadingprogressbar), the conditionalDisplay removal mechanism, and the behavioral fork when 2+ instances exist on one page."
---

**Requires:** BricksExtras 1.7.3+ with xtableofcontents enabled

# BricksExtras: Table of Contents (xtableofcontents)

Shipped by the **BricksExtras** plugin. A non-nestable element built on Bricks core's own bundled **tocbot** library. The list body (`.x-toc_body`) always renders empty server-side — the entire heading list is built client-side by tocbot scanning the page against the JSON config in `data-x-toc`. Page source never shows real TOC items; only the live DOM does.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xtableofcontents.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xtableofcontents` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `contentSelector` has no safe empty fallback

Unlike `xreadingprogressbar`'s similarly-named `containerSelector` (safe to leave unset, falls back to `document.body`), `xtableofcontents`'s `contentSelector` is always written into the config — as an empty string if unset — and passed directly into tocbot with no page-wide fallback. Always set `contentSelector` to a real class on the container that holds the headings.

## `conditionalDisplay` removes the element from the DOM entirely

With `conditionalDisplay: "enable"`, if the number of real headings found on the page is below `conditionalDisplayValue` (default `1`, including zero), the TOC element is removed from the DOM after page load — not hidden via CSS.

## Behavior forks when 2+ TOC instances exist on the same page

With a single TOC instance, tocbot's own built-in scrollspy and click-to-scroll behavior drives active-link highlighting and navigation. With two or more TOC instances on the same page, a separate multi-instance tracker takes over instead — its own `IntersectionObserver` per instance, and its own click-to-scroll override when `smoothScroll` is enabled. Scroll-offset or active-state behavior can differ between a single-TOC page and a multi-TOC page because of this fork.

## Auto ID generation — two modes, one needs an extra script

`autoID` (checkbox) turns on automatic `id` generation for headings that don't already have one. `autoIDText`: `prefix` (default — `idPrefix` + index) or `text` (slugified heading text, via a conditionally-enqueued `slugify` script that only loads when `autoIDText: "text"`).

## Programmatic control

`window.bricksextras.tableOfContents.open({target})` / `.close({target})` / `.toggle({target})` — simulates a click on the header toggle for a given TOC element, for wiring up open/close from elsewhere on the page (e.g. a Bricks interaction).

## Other settings

- **`closePageLoad`** (checkbox) — starts the body collapsed. Mutually exclusive with **`closeBreakpoint`** (gated to `closePageLoad != true`), which force-closes only at specific breakpoints via a CSS custom property (`--x-toc-close`) the JS reads back on resize — same "CSS carries the config" pattern as `xreadmoreless`'s `collapsedHeight`.
- **`smoothScrollOffset`** — the entered number is negated before being sent to tocbot (entering `100` results in a `-100` scroll offset), to account for a sticky header pushing the scroll target down.
- **`listType`**: `counter` / `border` / `text` — also embedded in `data-x-toc` for CSS to match via substring attribute selectors (`[data-x-toc*=counter]`, `[data-x-toc*=border]`), the same mechanism `xreadingprogressbar` uses for `progressPosition`.
- **`nestCounters`** (checkbox) — only relevant with `listType: counter`; prevents nested sub-items from compounding their parent's counter numbering.
- **`maybe_remove_header`** (checkbox) — removes the clickable header/toggle row entirely, not just its content.
- Extensive typography/color/spacing controls per heading level (h2–h6), active-link state, and the header row — direct CSS mappings, no hidden behavior.

## Rendered DOM (hydrated, for custom CSS/targeting)

Real headings, `autoID: true`, `listType: "counter"`:

```html
<nav class="brxe-xtableofcontents" data-x-toc="{...}">
  <div class="x-toc_header" role="button" aria-expanded="true" aria-controls="x-toc_{id}">
    <div class="x-toc_header-text">Table of contents</div>
  </div>
  <div class="x-toc_body" aria-hidden="false" id="x-toc_{id}">
    <ol class="x-toc_list">
      <li class="x-toc_list-item x-toc_active-li">
        <a href="#heading0" class="x-toc_link node-name--H2 x-toc_active-link">Introduction</a>
      </li>
      <li class="x-toc_list-item">
        <a href="#heading1" class="x-toc_link node-name--H2">Getting Started</a>
        <ol class="x-toc_list">
          <li class="x-toc_list-item">
            <a href="#heading2" class="x-toc_link node-name--H3">Installation</a>
            <ol class="x-toc_list"><!-- one more nested level per H4, etc. --></ol>
          </li>
        </ol>
      </li>
    </ol>
  </div>
</nav>
```

Notes:

- **The root is a real `<nav>` element**, not a `<div>`.
- **Sub-headings produce genuinely nested `<ol>` elements** (one nested list per depth level, e.g. an `h3` under an `h2` gets its own child `<ol>`) — not a flat list with indentation classes. Depth-specific styling should target actual nesting depth (`.x-toc_body > ol > li > ol`, etc.) or the per-level class below, not assume a flat structure.
- **Every link carries a `node-name--H{level}` class** (`node-name--H2`, `node-name--H3`, etc., matching the heading's real tag) — a real, stable per-heading-level styling hook not tied to nesting depth, useful when depth and heading level don't map 1:1 (e.g. `headingSelectors` skipping a level).
- **The active heading gets two classes simultaneously**: `.x-toc_active-li` on the `<li>` and `.x-toc_active-link` on the `<a>` inside it — style via whichever is more convenient, both are present together.
- `.x-toc_header`'s `aria-expanded`/`.x-toc_body`'s `aria-hidden` are real, live-updated ARIA state (toggled by clicking the header, or via the `bricksextras.tableOfContents` API) — not static.

## Build workflow

1. Always set `contentSelector` explicitly to a real class on the content wrapper — there is no working "whole page" fallback.
2. Set `headingSelectors`, `listType`, and `idPrefix` as needed for the design.
3. If auto-generating IDs from heading text (`autoIDText: "text"`), confirm the `slugify` script actually loaded.
4. Use `conditionalDisplay`/`conditionalDisplayValue` to avoid showing a near-empty TOC on short pages — this removes the element from the DOM entirely, not just visually.
5. If a page has more than one TOC instance, verify scroll/active-link behavior specifically on that page rather than assuming parity with a single-instance page.
