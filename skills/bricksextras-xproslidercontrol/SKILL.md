---
name: xproslidercontrol
description: "Use when building or debugging Pro Slider Control (xproslidercontrol) from BricksExtras: external prev/next arrows, progress bars, counters, pagination dots, or play/pause toggles for xproslider. Covers targeting mechanics and a controlType cheatsheet."
---

**Requires:** BricksExtras 1.7.3+ with xproslidercontrol element enabled

# BricksExtras: Pro Slider Control (xproslidercontrol)

Shipped by the **BricksExtras** plugin, alongside `xproslider` (the slider itself — see skill `xproslider`) and `xproslidergallery` (dynamic image population — see skill `xproslidergallery`). Load the Pro Slider skill alongside this one for any build that needs both — which is most of them.

This is a **sibling utility element**, placed **outside** the slider it targets (e.g. below the slider's section, or in a page header). It renders exactly one piece of external UI per instance and has no slide content of its own — it's a remote control, not a slider.

**Ignore this element's own schema placement text if it conflicts with real testing** — in general, trust verified behavior over a generic `intro`/`description` string in the schema.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xproslidercontrol.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xproslidercontrol` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Targeting a slider (this element's own mechanism)

Settings: `slider` (select: `section` — within the same section / `selector` — specific CSS selector / `component` — within the same component) + `sliderSelector` (text, required when `slider: selector`) + `componentScope` (select; defaults to false).

**`componentScope` is a select control, not a checkbox — send the string `"true"`, not the boolean `true`.** Sending a boolean is silently ignored (the control falls back to its unset/false state with no error), which looks identical to a genuine scoping failure. This is the same value-shape rule as `arrows` on `xproslider`: select-type true/false fields take string values.

Default (`slider: section`) works for the common case of the control sitting in the same Bricks section as the slider it targets — **no selector needed**. The control finds the slider automatically with zero extra wiring, as long as both share an actual enclosing `section` element (not just adjacent root-level elements — the targeting requires a real ancestor `section`).

The runtime config attribute (`data-x-slider-control`) always shows `"slider":false` regardless of whether targeting actually succeeded — that field is a static server-rendered placeholder, not a live binding-status indicator. **Once initialized, the control's root gets a separate `data-x-control-for="{sliderId}"` attribute holding the actual matched slider's id** — this is the real, live way to confirm targeting succeeded (and which slider it bound to), not the static `data-x-slider-control` JSON. It's only present after JS runs, so check a real browser render, not raw HTML.

Use `selector` only when the control lives somewhere else in the layout (e.g. a sticky header). **Target by class, not `_cssId`** — see `bricksextras-start-here`'s "Targeting another element" section for why: `_cssId` is baked into a component's definition, so every instance of that component collides on the same literal DOM id, and any control relying on it silently grabs the wrong slider the moment the pairing is duplicated via a component. Give the slider a class via `_cssClasses` and point `sliderSelector` at `.thatClass` instead.

**If both the control and the slider it targets live inside the same component, set `componentScope: "true"`.** Without it, `.thatClass`-based targeting matches the *first* element with that class anywhere on the page once the component is duplicated — `componentScope: "true"` confines the lookup to the current component instance. With a two-slider component (`sliderA`/`sliderB` classes, `slider: "selector"` + matching `sliderSelector` + `componentScope: "true"` on each control) duplicated to two instances on one page, each instance's "next" button correctly advances only its own slider. The same setup with `componentScope: true` (boolean) fails — the second instance's button silently drives the first instance's slider instead of its own.

**This targeting mechanism is different from `xproslider`'s own `isNavigation`/`syncSelector`** (slider-to-slider sync, for when the "other side" is itself a full slider with real slides, e.g. a thumbnail strip). Use `xproslidercontrol`'s targeting (this skill) for a non-slider utility widget — a button, bar, counter, or dots — with no slide content of its own.

---

## Control-type cheatsheet (`controlType`)

Each Control element renders exactly one of these — add multiple Control elements (as siblings) for multiple pieces of UI (e.g. one for progress bar, one for counter, two for prev/next).

| `controlType` | Renders | Key companion settings |
|---|---|---|
| `progressBar` | Interactive progress track | `progressBarClickable` (default true), `progressSegmented` (one segment per slide, meant for single-slide-per-move sliders), `progressDirection` |
| `counter` | "X of Y" text | `countType`: pages vs slides to move; prefix/suffix text and separator are all independently styleable |
| `navArrow` | A single prev **or** next button | `navType` (`prev`/`next` — **add two Control elements, one per direction**, not one element for both); `buttonType`: `icon` (default chevron renders even if no icon is explicitly set) or `nest` (add your own elements as the button content) |
| `playPause` | Autoplay toggle button | Only meaningful when the slider's `autoplayscroll` is enabled; separate play/pause icons and aria-labels |
| `autoplayProgress` | Visual countdown of the autoplay interval | `autoplayProgressType`: `progressBar` or `circle`; requires slider autoplay enabled to animate |
| `slideContent` | Reads content from the active/prev/next slide | `slideContentType`: `caption` (pulls the nested `xproslidergallery` element's per-image WP caption automatically — only works if that slider is in gallery mode, see the Gallery skill), `attribute` (reads a custom `data-*` attribute you set per manual slide block), or `custom` (targets an arbitrary selector inside the slide) |
| `paginationDots` | Clickable dot pagination (separate from the slider's own built-in `pagination` toggle) | Full independent dot styling: size, color, active state, gap, direction |

**Note:** the slider element (`xproslider`) also has its own built-in `pagination` (dots) and `arrows` toggles, rendered *inside* the slider's own markup, independently styled from this element's `paginationDots`/`navArrow` types. Use the slider's own toggles for the simple default case; reach for this element when the nav/dots need to be positioned or styled independently of the slider's box (e.g. dots below a full-bleed hero slider, in a different container, or via one of the richer control types like a progress bar or counter that the slider itself doesn't offer natively).

---

## Gotchas

- **Disable the slider's own `arrows`/`pagination` when replacing them with Control elements.** Adding a `paginationDots` control without turning off the slider's own `pagination` renders both at once — same for `navArrow` and the slider's own `arrows`. The slider defaults both to visible (see the Pro Slider skill's gotchas), so this has to be an explicit step, not an afterthought: set `pagination: false` on the slider whenever a `paginationDots` control is added, and `arrows: "false"` whenever both `navArrow` (prev + next) controls are added.
- **Prev/next needs two separate elements.** One Control element renders one arrow (`navType: prev` or `navType: next`) — there's no single-element "both arrows" option.
- **`slideContent` with `caption` only works when the target slider is in gallery mode.** It reads the WordPress attachment caption field from the paired `xproslidergallery` element's images — no caption source exists for manual slide blocks. Use `attribute` or `custom` content type instead for manual-slide sliders.
- **Icon fields have working defaults.** `prevIcon`/`nextIcon` (and similar) render a sensible default chevron SVG even when left completely unset. No need to explicitly source an icon for a basic prev/next pair.

---

## Reference pattern: progress bar + counter + prev/next, targeting by section

A slider with its own `arrows`/`pagination` turned off, replaced by four external Control elements sitting in the same Bricks section, using the default `slider: section` targeting (no `_cssId` needed since everything shares one section):

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xproslidercontrol.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
[
  { "name": "xproslidercontrol", "settings": { "controlType": "progressBar", "progressBarThickness": "3px", "progressDirection": "ltr" } },
  { "name": "xproslidercontrol", "settings": { "controlType": "counter", "countType": "slides", "slideSeperatorText": "/" } },
  { "name": "xproslidercontrol", "settings": { "controlType": "navArrow", "navType": "prev" } },
  { "name": "xproslidercontrol", "settings": { "controlType": "navArrow", "navType": "next" } }
]
```

`data-x-slider-control` on each carries `"control":"section"`, the progress bar renders real `role="progressbar"` markup, the counter renders index/separator/total sub-elements, and both nav buttons render working default chevron icons with correct default `aria-label`s ("Previous Slide"/"Next Slide") — all without setting the icons or labels explicitly.

**`progressBar`'s exact markup** — server-rendered shell, hydrated with live ARIA values once Splide initializes:

```html
<div class="x-slider_progress" role="progressbar" aria-label="Slider progress" aria-valuemin="1" aria-valuemax="2" aria-valuenow="1" aria-valuetext="1 of 2 slides">
  <div class="x-slider_progress-bar" style="width: 50%;"></div>
</div>
```

**`paginationDots`'s exact markup** — the `<ul>` shell is server-rendered empty, `<li>` dots are added by Splide's JS at init (each dot's `aria-controls` points at that specific slide's own id, `{sliderId}-slideNN`, not the slider root):

```html
<ul class="x-slider-control_pagination" role="tablist" aria-label="Select a slide to show">
  <li role="presentation"><button class="splide__pagination__page is-active" role="tab" aria-label="Go to page 1" aria-selected="true" aria-controls="{sliderId}-slide01"></button></li>
  <li role="presentation"><button class="splide__pagination__page" role="tab" aria-label="Go to page 2" aria-selected="false" tabindex="-1" aria-controls="{sliderId}-slide02"></button></li>
</ul>
```

**`counter`'s exact markup** (note the misspelled `x-slider_counter-seperator` class — real, not a typo to fix when styling):

```html
<div class="x-slider_counter">
  <div class="x-slider_counter-index"><div class="x-slider_counter-index-number">2</div></div>
  <div class="x-slider_counter-seperator"> of </div>
  <div class="x-slider_counter-total"><div class="x-slider_counter-total-number">2</div></div>
</div>
```

**`navArrow`'s exact markup** (prev shown; next is identical with `--next`/`Next Slide`) — `aria-controls` points at `{sliderId}-track`, not the slider root or a specific slide, and Splide adds a real `disabled` attribute at the non-looping end of the track, not just a visual/CSS-only disabled state:

```html
<button class="x-slider-control_nav x-slider-control_nav--prev" type="button" aria-label="Previous Slide" aria-controls="{sliderId}-track">
  <span class="x-slider-control_nav-arrow"><svg class="x-slider-control_nav-arrow-default" ...></svg></span>
</button>
```

**`playPause`'s exact markup** — same `-track` `aria-controls` target as `navArrow`, both play and pause icon spans always present (toggled via CSS against the button's own state, not swapped in/out of the DOM):

```html
<button class="x-splide__toggle splide__toggle is-active x-splide__toggle_ready" type="button" aria-label="Pause" aria-pressed="true" aria-controls="{sliderId}-track">
  <span class="x-splide__toggle__play splide__toggle__play"><svg>...</svg></span>
  <span class="x-splide__toggle__pause splide__toggle__pause"><svg>...</svg></span>
</button>
```

**`autoplayProgress`'s exact markup** — also sets a `--x-slider-autoplay` CSS custom property on its own root (not documented elsewhere), presumably what drives the bar's fill animation:

```html
<div class="x-slider-control_autoplay_progress" role="presentation" style="--x-slider-autoplay: 1;">
  <div class="x-slider-control_autoplay_progress-bar"></div>
</div>
```

**`slideContent`'s exact markup** (`attribute` mode) — the content itself is naturally whatever the matched slide's attribute value is, so don't expect fixed text; the structural part is the single wrapping span and the opacity/visibility fade between slide changes:

```html
<div class="brxe-xproslidercontrol x-slider-control" style="opacity: 1; visibility: visible;">
  <span class="x-slider-control_content">{whatever the matched attribute's value is}</span>
</div>
```

For `attribute` mode specifically, two settings are both required together: `slideContentSelector` (a selector for the specific element *within* the slide to read from — not the whole slide) and `slideAttribute` (the attribute name on that matched element). Getting either wrong (or leaving `slideContentSelector` on its schema placeholder, `.some-element`) produces a real element with `opacity: 0; visibility: hidden` and empty content — a silent, structurally-valid-looking failure, not an error.

**The remaining three control types**, against a manual-slide slider with `autoplayscroll: autoplay`, all inside a real `section` wrapping both the slider and the controls:
- `playPause` renders a real `<button class="x-splide__toggle splide__toggle">` with working default play/pause SVG icons and default `aria-label`s (`Play`/`Pause`). Clicking toggles the `aria-label` between `Play`/`Pause` and the `is-active`/`x-splide__toggle_ready` classes, and correctly pauses/resumes the slider's autoplay.
- `autoplayProgress` renders `<div class="x-slider-control_autoplay_progress" role="presentation">` wrapping a `.x-slider-control_autoplay_progress-bar` inner div — a different ARIA role than `progressBar` (`presentation`, not `progressbar`), since this is a passive countdown, not an interactive track. The bar animates over the autoplay interval while autoplay is running.

**`slider: "component"` also works:** a slider + four controls (progress bar, prev, dots, next) all built inside one component, no `_cssId` anywhere, every control set to `slider: "component"`, duplicated to two instances on one page — each instance's controls correctly drive only their own slider, with no cross-instance interference. This is the preferred targeting mode whenever the control and its slider live inside the same component and there's only one slider in that component — simpler than wiring up a class + `componentScope`.

---

## Workflow for adding controls to a slider

1. **Identify the target slider first.** If a specific page is already in scope, read its element tree and locate the `xproslider` instance this control set should target — there's no site-wide "find every page using this element" ability in this environment, so this step only applies once a page is already in scope.
2. **Get the current schema** for `xproslidercontrol` — check the bundled schema first per `bricksextras-element-schemas`; only if the bundle is missing or stale, call whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` with `parameters: {"elementName": "xproslidercontrol"}` on the native Bricks MCP). Don't assume control keys from memory, especially for less-common types like `autoplayProgress` or `slideContent`.
3. **Decide targeting.** Same-section placement + default `slider: "section"` covers most cases with zero extra config. Inside a component with a single slider, prefer `slider: "component"`. Only reach for `slider: "selector"` + a class on the slider (+ `componentScope: "true"`, the string, if both live in the same component) when the control lives elsewhere in the layout. Never use `_cssId` for this — see `bricksextras-start-here`.
4. **Turn off the slider's own `arrows`/`pagination`** if replacing them with external Control elements for those same functions, to avoid duplicate UI.
5. **Add one Control element per piece of UI**, remembering prev/next needs two.
6. **Insert and verify** — for any control type not covered in the "reference pattern" above, check the rendered HTML/config rather than trusting the settings alone.
