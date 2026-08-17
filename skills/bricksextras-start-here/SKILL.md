---
name: bricksextras-start-here
description: "Use at the start of every BricksExtras session. Core concepts: element naming conventions, MCP schema fetching, targeting other elements, and fundamental differences from Bricks core elements."
---

**Requires:** BricksExtras 1.7.3+.

# BricksExtras: Start Here

Load this skill at the start of any BricksExtras work. It covers fundamental concepts that apply across all BricksExtras elements and prevent common mistakes.

**Hard requirement: load the native `bricks-start-here` skill first, before this one, whenever it exists in the project.** BricksExtras is not a standalone theme or page-builder — it's a plugin that registers additional elements and controls into Bricks. The underlying page/element/settings model, the write-verification workflow, and most of what you're actually building with is core Bricks. This skill only adds the extra information needed for the `x*` elements BricksExtras registers on top of that — it is not a substitute for the core Bricks skills and never covers Bricks' own fundamentals. Check for `bricks-start-here` (and load it) before doing anything else in a Bricks/BricksExtras session; only skip it if it genuinely isn't present in this environment.

## An unrecognized query-loop object type is a signal to check for a skill, not a detail to skip past

If your MCP connection has a dedicated ability for listing query-loop object types (e.g. `bricks/list-query-loop-types` on the native Bricks MCP), it returns core types (`post`, `term`, `user`, `api`, `array`) alongside any provider- or plugin-registered ones. Not every connection exposes this as its own ability — if it isn't listed, the same information is discoverable from an element's own schema instead: fetch the `query` control's schema on `container` (or whatever element carries it) and check its `objectType` option list, which includes the same registered types. BricksExtras registers its own: `queryLoopExtras` (`source: "custom"`) covers adjacent posts, related posts, a nav-menu loop, a favorites loop, and — despite the generic-sounding name — a dynamic-data image gallery loop, all documented in `bricksextras-query-loop-extras`. None of this is covered by core Bricks' query-loop skill, since it's plugin-registered.

**Treat any `objectType` outside the five core types as a reason to check for a matching skill before falling back to a generic pattern** (array loop, manual repeater, etc.) — don't note it as "some custom type" and move on. This applies whether the type turns up in a live call or schema check, or the user's own request names a data source (a gallery field, related posts, a favorites list) that doesn't map cleanly onto post/term/user/array.

---

## Always pull current schema before building with an element — bundle first, live as fallback

**Before building or editing any BricksExtras element, get its current schema first — bundle first, live as fallback.** Load `bricksextras-element-schemas` and follow its workflow: check the bundled, version-checked schema first, and only go live when the bundle is missing, stale, or doesn't cover that element. If falling back to live, use `bricks/get-element-schema` if it's available in your current MCP connection's ability/tool listing; if it isn't, find and use whatever that connection's listing actually offers instead — don't guess at a substitute name. Pass that specific element's name (e.g. `xproslider`, `xaddtocalendar`, `xpanoramascene`) as the target either way. This applies everywhere in the BricksExtras skill set, not just here, so individual element skills don't repeat this caveat.

**"The schema" means one specific document: the raw JSON file the bundle stores for that element (`references/elements/<name>.json` inside `bricksextras-element-schemas`, found via that element's `schemaPath` in `references/index.json`) — or the live ability's response, if falling back.** It does **not** mean that element's own `SKILL.md` (e.g. `bricksextras-xtabs`). The element's own skill is curated documentation — gotchas, worked examples, rendered-DOM notes — written by a person reading the schema at some point in the past; it is not the schema itself, and reading it is not "checking the schema" no matter how complete or authoritative it looks. **Concretely, checking the schema means: open `references/index.json`, resolve that element's `schemaPath`, and actually read that JSON file this session** — not recalling that a version check happened earlier, and not treating a worked example already sitting in the element's own skill file as sufficient.

**Never assume** control names, defaults, or required fields from memory or previous sessions, and never trust a schema (bundled or previously fetched) without checking it's current for this site. BricksExtras elements evolve, and schema is the source of truth — but only the *current* schema, not whichever copy happens to be lying around, and only once you've confirmed which ability actually delivers it in this environment.

**The version check and the file read are two separate, both-required steps — doing one is not doing the other.** Confirming the bundle's `version` matches the live BricksExtras plugin version (`bricksextras-element-schemas`' step 1) only establishes that the bundled files are trustworthy; it does not mean you've opened the one that matters for this build. Both steps are required before writing settings JSON for an element: (1) confirm the bundle version is current, (2) actually read that element's `references/elements/<name>.json`.

**This rule applies to every BricksExtras element, with no exceptions** — individual element skills don't repeat it. When one says "get the current schema," it means: follow this section, substituting that element's own name for the schema ability's element-name parameter and its `schemaPath` for the file to read.

---

## Element naming conventions

BricksExtras elements use an `x` prefix:
- `xproslider` - Pro Slider
- `xproaccordion` - Pro Accordion
- `xmediaplayer` - Media Player
- `xproslidercontrol` - Pro Slider Control
- etc.

When searching for an element or checking if it exists, always include the `x` prefix.

Most BricksExtras elements also follow a consistent runtime-attribute pattern on their rendered root: `data-x-id="{id}"` (an internal instance identifier, separate from the DOM `id`) and some `data-x-*` attribute (name varies per element, not literally the element name) holding a JSON blob of the element's own runtime config — that's what the element's JS actually reads, not individual `data-*` attributes per setting. Not universal — check the actual render for the specific element rather than assuming — but common enough that it doesn't need re-explaining in every element's own skill.

---

## Fixed-position elements: place once, near the end of the page

Some BricksExtras elements render as fixed-position overlays relative to the viewport: `xpromodalnestable`, `xoffcanvasnestable`, `xinteractivecursor`, and `xbacktotop`. Their position in the page's content flow has no effect on where they visually appear, since CSS takes them out of flow entirely. Add them once, typically at the very end of the page (or in a sitewide footer/global template) — not nested inside a specific content section. Nesting one inside an unrelated section adds no visual benefit and just clutters the structure.

**Exception for `xpromodalnestable`/`xoffcanvasnestable` specifically:** when one is intentionally placed inside a query loop to pick up per-item dynamic content (e.g. a "view details" modal showing the current loop post's own data), it needs to live inside that loop to access the loop context — that's a deliberate, valid pattern, not something to "fix" by moving it to the end of the page.

---

## Nestable elements work with nested children

BricksExtras nestable elements (sliders, accordions, tabs, media players) support nested children in a single `add-element` call.

You can build the entire structure in one call:

```json
{
  "name": "xproslider",
  "settings": { "perPage": 1 },
  "children": [
    {
      "name": "block",
      "settings": { "_hidden": { "_cssClasses": "x-slider_slide splide__slide" } },
      "children": [
        { "name": "heading", "settings": { "text": "Slide 1" } }
      ]
    }
  ]
}
```

---

## Required CSS classes for functionality

Many BricksExtras elements require specific CSS classes to function. These are **not styling** - they're the mechanism that makes the element work (due to JS looking for the selector). Check that specific element's skill or schema for which classes it needs — always set them via `_hidden: { _cssClasses: "..." }`, they must be present or the element won't function.

**The classed wrapper elements themselves are fixed; what's inside/around them is not.** These elements are nestable specifically so their actual content can be customized — a documented per-item/per-slide template is a starting shape, not a rigid tree to reproduce verbatim. Adding, removing, or swapping content (no icon, an image instead of an icon, extra text, several elements in a content area instead of one) is a completely normal request and doesn't break anything. What must never change is the required-class element itself — its class, and generally its element type/position in the tree. Removing or reclassing *that* is what breaks the JS binding; customizing what's nested inside or alongside it is always safe.

---

## Style with the element's own controls first — custom CSS is for what those controls don't expose

Reach for an element's built-in style controls before writing custom CSS. They're the intended, maintained styling path — they track the plugin's own class names/structure across updates, and they're what a person editing the same page in the builder UI will find and use.

**Every style control's schema entry already states exactly what it targets** — each carries its own `css: [{ property, selector }]` mapping (e.g. `header_bg` → `background-color` on `th.gridjs-th`). Check the live/bundled schema's `css` mappings before assuming something needs custom CSS — a control that looks unrelated by its label may already target the exact selector a design needs.

**That schema selector is only the unscoped part — Bricks automatically prefixes it with the element's own scope when the control is actually used.** `header_bg`'s `th.gridjs-th` doesn't mean the generated rule is bare `th.gridjs-th { ... }` (which would leak to every table on the page) — Bricks compiles it scoped under whatever selects this specific element. Writing the same selector by hand in `_cssCustom` does not get that automatic scoping — prefix it yourself with the exact selector form (`#brxe-{id}` standalone, `.brxe-{id}` inside a component, `.{class-name}` for a global class) — a bare unscoped selector applies globally instead of to just this element.

Drop to custom CSS only when the design genuinely needs something no control exposes (a hover state with no dedicated setting, a sub-element with no styling group at all, a very specific layout tweak). Where an element auto-injects markup you didn't write yourself (wrapper divs, JS-inserted buttons, state attributes that only appear at runtime, portalled content, etc.), that element's skill may include a "Rendered DOM" section showing the actual output — this exists to make *that* custom-CSS case easier: knowing the real class names/structure in advance means it can be targeted correctly the first time, without needing to fetch a live render to find out. Not every element has one; it's only added where the real DOM has structure that isn't derivable from the settings JSON alone.

---

## Classes added for styling: use a real global class, not a local string

Any class you add purely for styling purposes — whether it carries built-in style-control settings or a `_cssCustom` rule — should be a real global class (`_cssGlobalClasses`, created via the design-system abilities), not a plain string dropped into an element's own `_cssClasses`. A global class is a registered design-system record: it shows up in whatever global-class/design-context listing ability the current MCP connection exposes, is reusable across other elements, and gives anyone auditing the site one place to find and edit it. A local string on one element's `_cssClasses` is invisible to that tooling and isn't reusable, even though the resulting CSS selector behaves identically either way.

**Never use the `x-` prefix for a class you create.** That prefix is reserved for BricksExtras' own internal element-structure classes — the required functional classes documented above, and the ones you'll see in element schemas/rendered DOM (`x-hotspot-item`, `x-panorama-viewer_inner`, etc.). Using it for a site-authored styling class collides with that convention and misleads anyone reading the class list into thinking it's plugin-internal. Check the site's existing global classes first for its actual naming pattern and match it.

**Mechanically, a global class is attached via `_cssGlobalClasses` — an array of class IDs on the element's own `settings` object, separate from `_cssClasses`/`_hidden._cssClasses`.** This matters here because a required functional class (via `_hidden: { _cssClasses: "..." }`) and a design-system styling class routinely land on the same element — e.g. a slide `block` that needs both the slider's functional classes and a card-styling global class:

```json
{
  "name": "block",
  "settings": {
    "_hidden": { "_cssClasses": "x-slider_slide splide__slide" },
    "_cssGlobalClasses": ["someGlobalClassId"]
  }
}
```

Don't fold a global class into the `_cssClasses` string — that field takes plain local-class names, not global class IDs.

---

## Schema defaults are UI-only unless you write them into the JSON yourself

Every control in a schema shows a `default` (e.g. `xburgertrigger`'s `burger_animation` defaults to `"x-hamburger--slider"`). That default is what a human gets in the Bricks builder because the UI pre-fills it into the element's saved settings the moment the element is added — it is **not** applied by the plugin at render time just because the key is missing. Writing JSON directly bypasses that UI pre-fill step, so omitting a key is not the same as "use the default."

An `xburgertrigger` built without `burger_animation` renders a static hamburger icon with no animation class at all — not the slider animation the default implies. Same silent-gap failure as the `clickTrigger` behavior documented above.

**The rule: for any setting the user hasn't specified one way or the other (no ask for a particular icon, style, content, or behavior), write the schema's `default` value into the settings JSON explicitly, rather than omitting the key.** This reproduces exactly what a person would get by adding the element in the builder and leaving that field untouched — which is the right assumption whenever the user hasn't said otherwise. Only deviate from the default when the request actually calls for something else.

**Exception: selector-targeting fields keep their own rule.** `clickTrigger`/`elementSelector`/`linkSelector`/`syncSelector` also show a `default` (e.g. `.brxe-xburgertrigger`), but don't just copy it verbatim — follow "Targeting another element" above instead: create an explicit class on the target element and point the selector field at that class. That rule exists for a different reason (multi-instance safety, not just "the default isn't auto-applied"), so it overrides this general one for that specific category of field.

---

## `builderPreview`/`*Preview` controls are builder-only — never confused with frontend behavior

Some elements have a control literally named `builderPreview`, or another control with "preview" in its name/label (e.g. `xinteractivecursor`'s `builderPreview` select with options like `default`/`trail-grow`/`text-visible`/`mousedown`). These exist purely to let a person toggle what's shown/highlighted **while editing in the Bricks builder canvas** — hiding elements that would otherwise clutter the canvas, or letting the editor preview a hover/active/click state without physically triggering it. They are never sent to frontend JS and never affect what a real visitor sees or does.

On `xinteractivecursor`, the element's rendered `data-x-cursor` JSON config attribute on the frontend contains only its genuinely runtime fields (`hoverSelectors`, `trailDelay`, `wait`) — `builderPreview` is absent entirely. Whatever `builderPreview` (or an equivalent preview-only control) is set to has no bearing on which state actually shows on the live site; real states are driven by real user interaction (hover, click, scroll, etc.), not by this setting.

**Practical effect:** don't try to set a `*Preview`-style control to "activate" a particular visual state for real visitors, and don't treat its value as meaningful when auditing why something looks a certain way on the frontend — it only affects the builder's own editing view.

---

## Targeting another element: use a class, never `_cssId`

Many BricksExtras elements find "the other element" they work with (a slider control finding its slider, a popover finding an arbitrary target element to attach a tooltip to, etc.) via a text-based selector field on their own settings. **Never wire this up with `_cssId`.** `_cssId` exists for CSS targeting and anchor links, not for this — and it actively breaks the moment the pairing sits inside a Bricks component: `_cssId` is baked into the component's *definition*, so every instance of that component renders the identical literal `id` in the DOM. Two instances on the same page means two elements with the same `id`, and any selector lookup (`document.querySelector('#id')`, which is how these selector fields resolve under the hood) always resolves to the *first* match in the page — so the second instance's control silently grabs the first instance's element instead of its own. No error, no warning, just wrong behavior only visible once something is duplicated.

**Use a class instead.** Give the target element a class via `_cssClasses` (a plain non-global class is fine — it doesn't need to be a design-system class), and point the controlling element's selector field at `.thatClass`. Classes don't have the uniqueness problem `_cssId` has.

**Always create and wire a new class explicitly — never rely on a selector field's `default`/`placeholder` value.** Selector-targeting fields (`clickTrigger`, `elementSelector`, `syncSelector`, etc.) often show a schema `default` like `.brxe-xburgertrigger` — that's a placeholder shown to a *human* in the builder UI to illustrate the expected format, not a value that gets applied automatically when the key is omitted from JSON. Omitting the key leaves the setting empty at runtime, silently breaking the connection with no error. For an AI writing JSON directly, that placeholder is irrelevant either way: always add an explicit class to the target element via `_cssClasses` and set the controlling element's selector field to `.thatClass`, even for the single-instance case the default was hinting at.

Example: to wire a burger trigger to an offcanvas, don't lean on `.brxe-xburgertrigger`. Add `_cssClasses: "offcanvas-toggle"` to the `xburgertrigger` element, then set `clickTrigger: ".offcanvas-toggle"` on the `xoffcanvas`/`xoffcanvasnestable` element.

### Component scope: the general fix for selector targeting inside components

Once that class-based pairing sits inside a component, a *class* has the opposite problem from an id: if the component gets duplicated, `.thatClass` now matches multiple elements across multiple instances, and `.thatClass` alone doesn't say which instance's copy to use.

This is what the **`componentScope`** setting is for, on the elements that have it: set it to `"true"` and the selector search is confined to the current component *instance's* own subtree, not the whole page. Present on `xproslidercontrol` and on `xproslider`'s own `isNavigation`/`syncSelector` sync settings. **Not every element with a selector-targeting field has this option** — `xpopover`'s `elementSelector` field, for example, has no `componentScope` counterpart in its schema at all. Always check the live schema for the specific element before assuming `componentScope` is available; when it isn't, duplicating that element's targeting pairing inside a component is unsafe regardless of class vs. id, and the targeting element likely needs to live outside the component, or the component needs to stay to a single instance per page.

**`componentScope` is a select control, not a checkbox — the value must be the string `"true"`, not the boolean `true`.** A boolean is silently ignored (falls back to the unset/false state with no error), which looks identical to a genuine scoping failure and is easy to misdiagnose as "componentScope doesn't work." A two-slider component with class-based targeting and `componentScope: true` (boolean) fails — one instance's control drives the other instance's slider. Changing only the value to `componentScope: "true"` (string) fixes it immediately, with everything else unchanged.

### "Within the same component" — a narrower convenience, not the general rule

A few elements (Pro Slider Control's `slider: "component"` option is the concrete example seen so far — a handful of others, like some toggle/content-switcher elements, offer an equivalent) provide a dedicated targeting mode that skips the class/selector pairing entirely: it just automatically finds the nearest matching target within the current component instance. This is the same idea as their `"section"` option (search within the nearest enclosing section), just scoped to "within this component" instead. It only exists on elements whose author specifically built it in — check the live schema for an option literally named `component` before assuming it's available. When it *is* available and there's only one of the target element inside the component, prefer it — it's simpler than wiring up a class and remembering `componentScope`. When it isn't available, or there's more than one plausible target inside the component, fall back to the general class + `componentScope: "true"` pattern above.

---

## Never do

- Do not assume where `hasLoop`/`query` belongs from one element to the next — placement varies (looping child, the element itself, or a wrapping `block`, depending on the element's own architecture). Check that specific element's skill or schema rather than applying a general rule.
- Do not skip getting the current schema (bundle-first per `bricksextras-element-schemas`, live as fallback) - control names and requirements change between versions
- Do not treat reading an element's own `SKILL.md` as equivalent to checking its schema — they are different documents; only the bundled JSON file at that element's `schemaPath` (or the live ability response) is the schema.
- Do not treat "I did the version check earlier this session" as covering a specific element's file read — the version check and the per-element schema file read are separate steps, both required.
- Do not omit required CSS classes - they're functional, not decorative
- Do not assume BricksExtras elements behave identically to Bricks core elements in all ways
- Do not use `_cssId` to wire two elements together (slider sync, control targeting, popover element selectors, etc.) — it's baked into component definitions and collides across every instance. Use a class, and set `componentScope: "true"` (the string, not the boolean) if both elements live inside the same component *and* the targeting element's schema actually has a `componentScope` field — not all of them do.
- Do not leave a selector-targeting field (`clickTrigger`, `elementSelector`, `syncSelector`, etc.) unset because the schema shows a `default` — that default is a UI placeholder only, never applied at runtime. Always add an explicit `_cssClasses` class to the target element and set the field to that class selector.
- Do not omit a non-selector setting just because the schema lists a `default` — that default only lands in a human's saved settings because the builder UI pre-fills it; writing JSON directly skips that step. Write the default value into the JSON explicitly unless the user asked for something else.

---

## Workflow for any BricksExtras element

1. **Get the current schema** — bundle-first per `bricksextras-element-schemas`, live only as fallback via whatever schema ability is actually available
2. **Check if it's nestable** - look for `"nestable": true` in schema
3. **Check for required CSS classes** - review element-specific skill or schema
4. **Check for `hasLoop` requirement** - if building with query loops
5. **Build, then verify per `bricks-start-here`'s write-verification rule** (read back + check rendered output, not just the write response)
