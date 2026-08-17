---
name: xslidemenu
description: "Use when building or debugging the Slide Menu element (xslidemenu) from BricksExtras: a nestable WordPress nav-menu renderer with slide/collapse behavior, sub-menu toggle icons, and mega-menu template support. Covers the two menu-source modes, the dropdown options list being empty outside the actual Bricks builder UI, the nested-vs-sibling icon positioning choice, and where nestable children get inserted relative to the menu."
---

**Requires:** BricksExtras 1.7.3+ with xslidemenu enabled

# BricksExtras: Slide Menu (xslidemenu)

Shipped by the **BricksExtras** plugin. A nestable element that renders a real WordPress nav menu via `wp_nav_menu()`, wrapped for slide/collapse JS behavior and BricksExtras' own sub-menu toggle icon and mega-menu template system. Several behaviors here aren't derivable from the control list alone.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xslidemenu.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xslidemenu` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Picking the menu: two source modes

`menuSource`: `dropdown` (default) or `dynamic`.

- **`dropdown` mode** — `menu` holds a WP nav menu's `term_id` as a string. Setting this to a real menu's term_id (e.g. `"67"`) renders correctly even though the schema's `options` list for this field is empty outside the actual Bricks builder UI — `set_controls()` only populates `options` from `wp_get_nav_menus()` when `bricks_is_builder()` is true, so a live schema pull via MCP will always show `options: []` for `menu` regardless of how many real menus exist on the site. Don't take an empty options list as "no menus exist" — look up real menus independently (e.g. `wp_get_nav_menus()` via a PHP execution ability) and pass the raw term_id.
- **`dynamic` mode** — `menu_id` (text field: menu name, slug, *or* ID — WordPress's own `wp_nav_menu()` `menu` arg accepts any of the three) — accepts a dynamic-data tag (`{...}`) resolved via `render_dynamic_data_tag`. Unlike `dropdown` mode, this value is passed straight to `wp_nav_menu()` with no `is_nav_menu()` validation first.

**If `menu` is empty or invalid in `dropdown` mode**, the element falls back to the first registered nav menu — a real build should always set `menu` explicitly; this fallback only exists so something renders if it's ever left unset. If literally no nav menus exist at all, it renders an element placeholder ("No nav menu found").

## Sub-menu toggle icon: set `icon`, but don't expect it in a static HTML read

`icon` doesn't render inline per menu item — it's only added once the frontend JS runs (`slidemenu.js`, for every item that has children, mega-menu items included), so a raw `get-page-elements`/HTML read won't show it. What the JS actually inserts, wrapping the item's original `<a>` together with a new toggle `<button>`:

```html
<div class="x-slide-menu_icon-wrapper">
  <a href="...">My Favorites</a>
  <button aria-expanded="false" class="x-slide-menu_dropdown-icon" aria-label="Toggle sub menu">
    <i class="fas fa-chevron-down"></i>
  </button>
</div>
```

`aria-label` on the button comes from `subMenuAriaLabel` (default "Toggle sub menu"); `aria-expanded` flips as the sub-menu opens/closes. Style the toggle via `.x-slide-menu_dropdown-icon`, not by assuming the icon sits directly inside the `<li>` or `<a>`.

## `menuStructure`: use `sibling`, not `nested`

`nested` (the schema placeholder/legacy default) exists only for backward compatibility with older builds. **`sibling` is the current, recommended option** — set it explicitly rather than leaving this on the legacy default.

## Nestable children: `maybeNestable` controls before/after placement, not visibility

`maybeNestable`: `disable` (default) / `above` / `below`. Controls exactly where the element's nested children render relative to the WordPress nav menu's own output, both inside the same `<nav>` wrapper:
- `above` → children render **before** the menu `<ul>`.
- `below` → children render **after** the menu `<ul>`.
- `disable` → nested children in the tree are simply never rendered at all, regardless of what's nested under the element.

## Mega menus: configured on the WP menu items, not on this element

`megaMenu` (checkbox) doesn't define mega-menu content itself — it turns on consumption of Bricks core's own native mega-menu template assignment (a Bricks Template ID stored in menu-item post meta, `_bricks_mega_menu_template_id`, normally set via Bricks' standard nav-menu-item mega-menu picker in wp-admin). When enabled, any menu item with a template assigned there gets that template's content rendered in place of its normal sub-menu, and the item's real WP sub-menu children are filtered out so they don't also appear. Assign mega-menu templates the normal Bricks way first; this checkbox just switches `xslidemenu` to honor them.

## Other settings

- **`defaultState`**: `open` (default) / `hidden`. Only when `hidden` does `clickSelector` (a plain CSS selector field, same targeting principle as `bricksextras-start-here`'s "use a class, never `_cssId`" rule — point it at an explicit class on whatever trigger element should reveal the menu) get included in the JS config at all; in the `open` state it's omitted from the config entirely, not just empty.
- **`builderPreview`**: gated to `defaultState: hidden`, and per the general `builderPreview`/`*Preview` rule in `bricksextras-start-here`, only affects the builder canvas — never the live frontend.
- **`maybeExpandActive`**: `enable`/`disable` (default `disable`) — sets `data-x-expand-current="true"` on the root when enabled, for JS to auto-expand whichever sub-menu contains the current page's active item.
- **`menu_width`**, **`slideDuration`**, **`menuDirection`** (ltr/rtl) — straightforward, self-explanatory from their labels.
- Extensive typography/color/border/spacing controls for menu links, active-state links, sub-menu links, and the toggle icon (default + expanded state) — all direct CSS mappings, no hidden behavior.

## Rendered DOM (for custom CSS/targeting)

Root, a regular item, and a mega-menu item, icon/template content elided:

```html
<nav class="brxe-xslidemenu" data-x-id="{id}" data-x-slide-menu="{...}">
  <ul id="menu-{slug}" class="x-slide-menu_list">
    <!-- regular item with a normal sub-menu — real wp_nav_menu() output, standard WP classes untouched: -->
    <li id="menu-item-{id}" class="menu-item menu-item-type-post_type menu-item-object-page menu-item-has-children menu-item-{id} bricks-menu-item">
      <a href="...">My Favorites</a>
      <ul class="sub-menu">
        <li id="menu-item-{id}" class="menu-item ... bricks-menu-item"><a href="...">Gotcha Verify Test</a></li>
      </ul>
    </li>
    <!-- item with a Bricks mega-menu template assigned (megaMenu: true) — no <ul class="sub-menu"> at all: -->
    <li id="menu-item-{id}" class="menu-item menu-item-type-post_type menu-item-object-page menu-item-{id} bricks-menu-item menu-item-has-children">
      <a href="...">Simple Elements Test</a>
      <div class="brxe-xslidemenu_mega-menu sub-menu" data-menu-id="{item-id}">
        <!-- the assigned Bricks template's full element tree, rendered verbatim - sections/containers/whatever it contains: -->
        <section class="brxe-section ...">...</section>
      </div>
    </li>
  </ul>
</nav>
```

Notes on this structure:

- **The root renders as a real `<nav>` element, not a `<div>`.**
- **A mega-menu item's content wrapper is a `<div class="brxe-xslidemenu_mega-menu sub-menu" data-menu-id="{item-id}">` — not a `<ul>`.** It keeps the `sub-menu` class alongside its own `brxe-xslidemenu_mega-menu` class (so any styling already targeting `.sub-menu` broadly still partially applies), but the tag itself changes from `<ul>` to `<div>` and there's no `<li>` children the way a real sub-menu has — instead, the *entire assigned Bricks template's own rendered markup* (sections, containers, whatever that template contains) is injected wholesale inside it. `data-menu-id` ties it back to the specific WP menu-item id, distinguishing it from the element's own `data-x-id`.
- **Every other menu item — including ones with a normal (non-mega) sub-menu — keeps completely standard WordPress `wp_nav_menu()` output**: `menu-item`, `menu-item-type-*`, `menu-item-object-*`, `menu-item-has-children`, `menu-item-{id}` classes, plus BricksExtras' own added `bricks-menu-item` class. Nothing about the regular item/sub-menu structure is BricksExtras-specific — style it the same way you'd style any core WordPress nav menu.

---

## Build workflow

1. Look up real WP nav menus independently (don't trust an empty `options` list on the `menu` control — that's expected outside the actual builder).
2. Choose `menuSource`; set `menu` (dropdown mode) or `menu_id` (dynamic mode) explicitly.
3. Set `menuStructure: "sibling"` (current option — don't leave it on the legacy `nested` default).
4. If nesting custom content, pick `maybeNestable: above`/`below` deliberately based on desired position, not just "on."
5. For mega menus, confirm the target menu items already have a Bricks mega-menu template assigned before enabling `megaMenu` here.
6. Verify in the browser — the sub-menu toggle icon and slide/collapse behavior are JS-driven and won't show up in a structural read of the element tree.
