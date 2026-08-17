---
name: xpopover
description: "Use when building or debugging the Popover / Tooltip element (xpopover) from BricksExtras: a nestable Tippy.js-based popover that can either show its own button, or (via maybeTooltip) act as a tooltip attached to unrelated external elements matched by a selector, with its own button hidden on the frontend. Covers the two operating modes, followCursor's mutual exclusivity with multipleTriggers, the aria-label/button-text relationship, and the appendBody portal behavior."
---

**Requires:** BricksExtras 1.7.3+ with xpopover enabled

# BricksExtras: Popover / Tooltip (xpopover)

Shipped by the **BricksExtras** plugin, built on **Tippy.js** (the enqueued script handle is literally `x-popper`, a naming holdover — the actual rendered markup is Tippy's own `tippy-box`/`tippy-content`/`tippy-arrow`/`data-tippy-root`, see Rendered DOM below). Nestable — nested children become the popover's content, rendered inside `.x-popover_content-inner`.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xpopover.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xpopover` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Two distinct operating modes, controlled by `maybeTooltip`

- **Default (unset)** — the element renders its own visible trigger `<button>` (`.x-popover_button`, with icon + optional `buttonText`) plus its content, and the popover attaches to that button.
- **`maybeTooltip` (checkbox, enabled)** — the element instead acts as a tooltip source for **other, unrelated elements** matched by `elementSelector` (a plain CSS selector, same class-not-`_cssId` targeting rule as elsewhere — see `bricksextras-start-here`). On the real frontend, the element's own button is **not rendered at all** in this mode (`render()`'s condition: only output the button if in builder preview OR `maybeTooltip` is unset) — only the content/tooltip markup renders, positioned relative to whichever external element the JS attaches it to. `hidebutton` (only relevant when `maybeTooltip` is set) additionally controls builder-canvas visibility of that same (already frontend-hidden) button.

When using tooltip mode, `maybeDynamicContent` chooses the actual content source per-target-element: enabled pulls each matched element's own tooltip-text setting dynamically; disabled uses the popover's own nested children as static content for every matched element instead.

**Dynamic content is read from a `data-x-tooltip` attribute on each target element, written by BricksExtras' own generic `x_tooltips`/`x_tooltip_content` controls — not something to hand-write via `_attributes`.** BricksExtras adds a `Content` checkbox (`x_tooltips`) and a `x_tooltip_content` textarea (both under a "Tooltip" style-tab group) to elements broadly, alongside its other generic "Interactive" controls (parallax/floating/tilt) — this is a plugin-wide mechanism, not specific to `xpopover`. Enabling `x_tooltips` and filling in `x_tooltip_content` on any element writes `data-x-tooltip="{"content":"..."}"` onto it automatically; the target element's own schema `description` on `x_tooltip_content` literally says "For use with the popover/tooltip element (enable Dynamic tooltip text)". With `maybeDynamicContent` enabled on the `xpopover`, its JS filters `elementSelector` matches down to only elements carrying a non-empty `data-x-tooltip` attribute and reads each one's own text from it at hover/click time — so a single `xpopover` in dynamic mode can serve multiple different elements sharing the same `elementSelector` class, each showing its own distinct content. A target matching `elementSelector` but with `x_tooltips` left off is silently excluded — it won't get a tooltip at all.

`multipleTriggers` (tooltip mode only) should be enabled whenever more than one element on the page matches `elementSelector` — it's the setting that makes the popover attach independently to every matched element (each showing its own content when using `data-x-tooltip`) instead of assuming a single target.

## `followCursor` only applies when `multipleTriggers` is off

Schema-gated (`required: maybeTooltip = true AND multipleTriggers = false`) and the control's own `info` text states this directly: `followCursor` (`false`/`true`/`horizontal`/`vertical`/`initial`) has no effect once `multipleTriggers` is enabled.

## `ariaLabel` only applies when there's no visible button text

`render()`: the button only gets `aria-label` set when `ariaLabel` is present **and** `buttonText` is empty — if `buttonText` is set, the visible text itself is the accessible name and `ariaLabel` is silently dropped, not layered on top.

## Positioning/behavior config, all frontend JS settings

`placement` (default `auto`), `offsetSkidding`/`offsetDistance` (px, along/away from the button), `moveTransition` (ms), `interaction` (`mouseenter focus` / `click` (default) / `mouseenter click`), `delay` (ms), `appendBody` (checkbox — moves the popover content to `document.body` in the DOM rather than staying inline, useful for escaping `overflow: hidden` ancestors). All of these apply in both modes.

## Styling

- `popoverBackgroundColor`/`popoverTransitionIn`/`popoverTransitionOut`/`popoverTranslateX`/`popoverTranslateY`/`popoverScale` are all written as CSS custom properties on the root (`--x-popover-*`), not directly on the content selector — the actual `.x-popover_content .tippy-content` styling reads those variables.
- `popoverWidth`/`popoverTypography`/`popoverBorder`/`popoverBoxShadow`/`popoverPadding` apply directly to `.x-popover_content .tippy-content` via normal `css` mappings.
- Button styling (`buttonTypography`/`buttonBackgroundColor`/`buttonBorder`/`buttonBoxShadow`/`buttonPadding` plus flex layout controls `buttonDirection`/`buttonJustifyContent`/`buttonAlignItems`/`_columnGap`/`buttonRowGap`) targets `.x-popover_button` and only matters when the button actually renders (i.e. not in tooltip mode on the frontend).

## Other

- Query-loop instance uniqueness: `isLooping`/`isLoopingComponent` written into the `data-x-popover` config when inside a running query loop, same mechanism as other BricksExtras elements.
- Default nested child is a single `text` element ("Popover content") — real content should replace it.

## Rendered DOM (for custom CSS/targeting)

Closed state — real, present in raw server HTML, nested children sit in place inside `.x-popover_content-inner` and Tippy hasn't touched anything yet:

```html
<div class="brxe-xpopover x-popover" data-x-id="{id}" data-x-popover="{...}">
  <button class="x-popover_button" aria-describedby="x-popover_content_{id}">
    <span class="x-popover_button-icon">...</span>
    <span>Open popover</span>
  </button>
  <div class="x-popover_content" role="tooltip" id="x-popover_content_{id}">
    <div class="x-popover_content-inner">
      <div class="brxe-text"><p>Popover content here.</p></div>
    </div>
  </div>
</div>
```

Open state depends on `appendBody`:

**`appendBody: false` (default) — Tippy injects its wrapper *inside* `.x-popover_content`, nesting your content further:**

```html
<div class="x-popover_content" role="tooltip" id="x-popover_content_{id}">
  <div data-tippy-root style="...">
    <div class="tippy-box" data-state="visible" data-theme="extras" data-animation="extras" role="tooltip" data-placement="bottom">
      <div class="tippy-content" data-state="visible">
        <div class="x-popover_content-inner">...</div>
      </div>
      <div class="tippy-arrow"></div>
    </div>
  </div>
</div>
```

**`appendBody: true` — `.x-popover_content` stays in place but goes empty; `[data-tippy-root]` is appended directly to `<body>` instead, carrying a copy of the popover's own classes so normal Bricks style-panel CSS (anything compiling to `.brxe-xpopover { ... }`) still reaches it:**

```html
<!-- stays where you built it, now empty: -->
<div class="x-popover_content" role="tooltip" id="x-popover_content_{id}"></div>

<!-- appended to <body>: -->
<div data-tippy-root class="brxe-xpopover x-popover" style="...">
  <div class="tippy-box" data-state="visible" data-theme="extras" data-animation="extras" role="tooltip">
    <div class="tippy-content" data-state="visible">
      <div class="x-popover_content-inner">...</div>
    </div>
    <div class="tippy-arrow"></div>
  </div>
</div>
```

Notes on this structure:

- **`button[aria-expanded]` is only added once opened** — absent in the closed/server-rendered state, `"true"` once open. Useful as a `[aria-expanded=true]` state hook without needing JS-added classes.
- **This is the same Tippy.js library used by `xmediacontrol`'s tooltips, but configured completely differently** — its own theme (`data-theme="extras"`, `data-animation="extras"`) rather than the media control's `data-theme="light"`/`data-animation="fade"`, and — critically — `xpopover`'s Tippy instance is **not portalled to `<body>` by default** the way the media control's always is. `appendBody` is the setting that decides which behavior this element gets; there's no equivalent choice on `xmediacontrol`.
- Regardless of `appendBody`, the actual content you nested as children always ends up inside `.tippy-content`, wrapped in `.x-popover_content-inner` — target styling there, not `.x-popover_content` itself (which either holds the whole Tippy tree, in inline mode, or sits empty, in `appendBody` mode).

---

## Build workflow

1. Decide the mode first: own visible trigger button (leave `maybeTooltip` unset) vs. tooltip attached to other elements (`maybeTooltip` enabled + `elementSelector` pointed at a real class on the target elements).
2. In tooltip mode, choose `maybeDynamicContent` deliberately — enabled reads each target's own tooltip text, disabled uses this element's nested children as shared static content.
3. Set `buttonText` or `ariaLabel`, not both expecting both to apply — only one becomes the accessible name.
4. Only rely on `followCursor` when `multipleTriggers` is left disabled.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real Tippy.js instance from `window.xTippy.Instances[dataXId]` (keyed by the `data-x-id` on the `.brxe-xpopover` root) in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), and drive it directly via Tippy's own API (`instance.show()`, `instance.setContent()`, etc.). It's registered synchronously on init, no artificial delay to wait out. Note this registry is shared: the Copy to Clipboard element's own built-in tooltip registers into this exact same `window.xTippy.Instances` map, keyed by its own `data-x-id` — not a popover-exclusive registry.
