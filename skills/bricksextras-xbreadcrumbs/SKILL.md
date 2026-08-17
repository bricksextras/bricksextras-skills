---
name: xbreadcrumbs
description: "Use when building or debugging the Site Breadcrumbs element (xbreadcrumbs) from BricksExtras: a breadcrumb trail with its own built-in generator plus optional passthrough to Rank Math, NavXT, All in One SEO, SEOPress, Yoast, or Slim SEO's own breadcrumb output. Covers which control groups apply per source, the WooCommerce-endpoint special case, and the always-shown front-page crumb."
---

**Requires:** BricksExtras 1.7.3+ with xbreadcrumbs enabled

# BricksExtras: Site Breadcrumbs (xbreadcrumbs)

Shipped by the **BricksExtras** plugin. Not nestable. `source` selects between BricksExtras' own breadcrumb generator (`extras`, default) or a passthrough wrapper around another SEO plugin's own breadcrumb output (`rankmath`, `navxt`, `allinone`, `seopress`, `yoast`, `slim`) — if the selected third-party plugin isn't actually installed/active, most sources render a builder-only placeholder (`seopress` instead renders nothing at all, silently).

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xbreadcrumbs.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xbreadcrumbs` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## Most control groups only apply to `source: "extras"` — two exceptions apply universally

`config`, `configSlim`, and `separator` groups are schema-gated to specific sources (`config`/`separator` require `extras`; `configSlim` requires `slim`) — when using a third-party source, that plugin renders its own markup and BricksExtras' equivalent settings for it (home link, category inclusion, separator character, etc.) have no effect, since they're not shown at all in that mode.

**`linkStyles` and `currentItem` (styling groups) are the exception — they apply regardless of `source`.** Their CSS selectors are written to target not just BricksExtras' own markup but the known DOM class patterns from every supported third-party plugin's breadcrumb output too (e.g. `currentItem`'s selector list includes `.rank-math-breadcrumb .last:last-child`, `.breadcrumb_last`, `.breadcrumb--last`, `.breadcrumb-item.active`, alongside BricksExtras' own `[aria-current=page]`). So typography/color/border styling for links and the current (active) crumb keeps working consistently even when switching `source` to a third-party plugin — only the structural/content settings (home link, prefixes, category inclusion) are source-locked.

`breadcrumbPrefix` (and its own styling sub-controls) is the one content-level setting that also applies to every source — every branch prepends it before calling into the selected source's output.

## `slim` gets bespoke treatment, unlike the other five third-party sources

The other third-party sources (`rankmath`, `navxt`, `allinone`, `seopress`, `yoast`) just call that plugin's own render function directly with no parameters. `slim` (Slim SEO) is different: BricksExtras builds an explicit `[slim_seo_breadcrumbs separator="..." display_current="..." label_home="..." label_search="..." label_404="..." taxonomy="..."]` shortcode call itself, using the dedicated `configSlim` group's controls (`slimhomeLabel`, `slimsearchLabel`, `slimerrorLabel`, `slimTaxonomy`, `slimseparator`, `slimdisplayCurrent`) — meaning Slim SEO's breadcrumb labels are actually configurable through this element, unlike the other passthrough sources.

## Home crumb: always shown on the front page itself, regardless of `maybeHome`

`render()`'s condition is `is_front_page() || $maybe_home` — so visiting the front page always includes a home crumb entry (rendered as static text/icon, not a link, since you're already there) even if `maybeHome` were somehow set to disable. Everywhere else, the home crumb only appears if `maybeHome` isn't disabled, and renders as a real link to `homeURL` (default dynamic tag `{site_url}`).

`removePrefix` (checkbox) only suppresses `breadcrumbPrefix` specifically **on the front page** — it has no effect on any other page, where the prefix (if set) always shows.

## `removeTitle` only removes the trail's last item on singular content

`removeTitle` (enable/disable) strips the final breadcrumb entry only when `is_singular()` is true — on archive/search/404/date pages, the last crumb (search term, error message, etc.) is never removed by this setting regardless of its value.

## WooCommerce account/endpoint pages replace the normal ancestor chain

For a `page`-type post that's also a WooCommerce endpoint URL (`is_wc_endpoint_url()` — account pages, order-pay, etc.), the page-crumb logic skips the usual parent/ancestor-page chain entirely and instead outputs just two crumbs: a link to the account page itself, then the endpoint's own title (e.g. "My Account" → "Orders"). Ordinary non-endpoint pages use the standard ancestor-chain logic (walking `post_parent`/`ancestors`).

## `cpt` repeater maps which taxonomy represents "the category" for a custom post type

When building the breadcrumb trail for a singular custom post type, BricksExtras can't guess which of that CPT's taxonomies should act as the hierarchical "category" ancestor — `cpt` (repeater: `postType` + `taxonomy` pairs) makes that mapping explicit per post type. Without an entry for a given CPT, no taxonomy-based ancestor crumb is added for it.

## Category/product-category crumb building

`priorityCategory` picks which specific category's term hierarchy to use when a post has multiple categories (otherwise the first/primary one is used). `excludeCategories`/`excludeProductCategories` (multi-select) remove specific categories from consideration entirely — both accept term IDs via the schema's own dropdown, populated from real site categories. `taxNesting` (enable, default) walks up each matched category's own parent terms as additional intermediate crumbs; disabling it flattens to just the single matched category.

## Rendered DOM (`source: "extras"`, for custom CSS/targeting)

Fully server-rendered, no JS dependency. A singular post with a nested category chain (`maybeHome: "enable"`):

```html
<nav class="brxe-xbreadcrumbs" data-source="extras" aria-label="breadcrumbs" data-separator-type="text" data-display-mode="flex">
  <ol itemscope itemtype="http://schema.org/BreadcrumbList" class="x-breadcrumbs_list">
    <li class="x-breadcrumbs_list-item" itemprop="itemListElement" itemscope itemtype="http://schema.org/ListItem">
      <a href="https://example.com" itemprop="item"><span itemprop="name">Home</span></a>
      <meta itemprop="position" content="1">
    </li>
    <li class="x-breadcrumbs_list-item" itemprop="itemListElement" itemscope itemtype="http://schema.org/ListItem">
      <a href="https://example.com/category/parent/" itemprop="item"><span itemprop="name">Parent Category</span></a>
      <meta itemprop="position" content="2">
    </li>
    <!-- one such <li> per taxNesting ancestor category, in order -->
    <li class="x-breadcrumbs_list-item" itemprop="itemListElement" itemscope itemtype="http://schema.org/ListItem" aria-current="page">
      <span itemprop="name">Post Title</span>
      <meta itemprop="position" content="5">
    </li>
  </ol>
</nav>
```

Notes:

- **`_root` has no `id`/`brxe-{id}` attribute at all** — unlike almost every other Bricks/BricksExtras element, `xbreadcrumbs` never sets an identifier attribute in `render()`. Target it via `.brxe-xbreadcrumbs` (or a custom `_cssId`/`_cssClasses` you set yourself), not an assumed `#brxe-{id}`.
- **The current-page crumb is a bare `<span>`, never a link, and carries `aria-current="page"` on its parent `<li>`** — this is the real hook `currentItem` styling targets (`[aria-current=page]`, per the skill's own note above), not a class on the span itself.
- Every non-current crumb's link and text are always split into a real `<a href itemprop="item">` wrapping a `<span itemprop="name">`. The schema.org `itemListElement`/`ListItem`/`position` microdata shown above is gated on `maybeSchema`, which defaults to enabled if left unset — disable it explicitly to drop the `itemprop`/`itemscope`/`itemtype`/`<meta>` markup entirely.
- `data-source`, `data-separator-type`, `data-display-mode` are always written on the root regardless of which crumbs render — useful as CSS/JS hooks independent of trail content.

## Build workflow

1. Leave `source` on `extras` unless the site already has one specific SEO plugin's breadcrumb output that must be reused verbatim — switching sources loses access to most of BricksExtras' own structural settings (though link/current-item styling still applies either way).
2. For CPTs with more than one taxonomy, add an explicit `cpt` repeater row mapping that post type to the taxonomy that should represent its "category" ancestor — without it, no taxonomy crumb is added.
3. Use `priorityCategory`/`excludeCategories` to control which category drives the breadcrumb trail on multi-category posts, rather than assuming the first-assigned category always wins.
4. Remember `removeTitle` only affects singular post/page trails — it won't remove the last crumb on archives, search, or 404.
