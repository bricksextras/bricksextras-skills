---
name: query-loop-extras
description: "Use when building a Bricks query loop that needs adjacent posts, a related-posts loop, a dynamic-data-sourced image gallery, a nav menu loop, or a favorites-list loop — BricksExtras registers all five as one custom query object type (queryLoopExtras) available on container/block/div/xdynamictable. Covers the settings shape (a sibling of query, not nested inside it), each sub-type's fields, and where the schema is and isn't fully discoverable via MCP."
---

**Requires:** BricksExtras 1.7.3+, Bricks 2.4+

# BricksExtras: Query Loop Extras (`queryLoopExtras`)

BricksExtras registers a custom Bricks query-loop object type, `queryLoopExtras` ("Extras"), via the `bricks/setup/control_options` filter — confirmed as a first-class, fully-discoverable object type: on connections with a dedicated ability for it (e.g. `bricks/list-query-loop-types` on the native Bricks MCP) it lists as `source: "custom"`, `engine: "bricks/query/run"`; where no such ability exists, it's discoverable the same way as any other custom type — from the `query` control's schema on `container`/`block`/`div`/`xdynamictable`. Whatever live schema ability the connection exposes for those elements (e.g. `bricks/get-element-schema` on the native Bricks MCP) returns every one of `queryLoopExtras`'s own controls directly — no PHP reading required to find the shape.

It's one query object type covering **five unrelated loop sources**, switched via a single `extrasQuery` select: `adjacent` (adjacent posts), `gallery` (dynamic-data image gallery), `related` (related posts by taxonomy), `wpmenu` (loop over a nav menu's items), `favorite` (loop over a user's favorited posts).

## Settings shape: `extrasQuery` and its sub-fields are siblings of `query`, not nested inside it

**This JSON is an example, not the schema.** Check `references/elements/query-loop-extras.json` (or the live schema ability, per `bricksextras-start-here`) before building from it — do not copy settings out of this block without confirming they still exist and mean what's shown here.

```json
{
  "name": "container",
  "settings": {
    "hasLoop": true,
    "query": { "objectType": "queryLoopExtras" },
    "extrasQuery": "related",
    "count": 3,
    "orderby": "rand",
    "taxonomies": ["category"]
  }
}
```

`query` only ever needs `objectType: "queryLoopExtras"` — none of the normal `query` keys (`post_type`, `posts_per_page`, `tax_query`, etc.) apply here. `extrasQuery` and every sub-type-specific field (`adjacentPost`, `count`, `x_favorites_orderby`, etc.) are top-level element settings, confirmed by their `required` rules referencing `extrasQuery` directly (no path prefix) while referencing `query.objectType` with a dot — the dot only appears for the one field that's actually nested.

## `post_type`'s options list is empty outside the real builder — a live MCP gap, not a real limitation

The shared `post_type` control (used by `related` and `favorite`) only populates its options when `bricks_is_builder()` is true. Fetching the schema via MCP doesn't satisfy that check, so the live schema ability (e.g. `bricks/get-element-schema` on the native Bricks MCP) returns `post_type` with just `{"any": "Any"}` as its options — the real registered post types are invisible through that call. When building via MCP, get real post type slugs from elsewhere (e.g. what's already used elsewhere on the site) rather than trusting this control's options list.

## `adjacent` — adjacent posts relative to the current single post

Requires an actual current post in scope (`global $post`) — returns nothing on an archive/non-singular context. `adjacentPost`: `prev`/`next`. `adjacentPostCount` (1–10, default effectively 1). `adjacentPostOrderby`: `date`/`modified`/`menu_order`/`ID`/`title`. `adjacentPostSameTerm` (checkbox) + `adjacentTaxonomy` + `adjacentPostExcludedTerms` (comma-separated term IDs) restrict to posts sharing a taxonomy term.

For the simple case (count ≤ 1, orderby `date`), it uses WordPress's own `get_previous_post()`/`get_next_post()`. Any other combination (multiple posts, or ordering by something other than date) switches to a custom `WP_Query`-based implementation. Each returned post object gets `in_adjacent_posts_loop = true` and `adjacent_direction` (`prev`/`next`) properties set on it.

## `gallery` — loop over a dynamic-data image field, not a WP_Query at all

`x_gallery_data` holds a dynamic data tag resolving to image data (e.g. an ACF gallery field) — required, returns empty if unset. `x_gallery_orderby`/`x_gallery_order`/`x_gallery_offset`/`x_gallery_max` control slicing/ordering of that array. `x_gallery_per_page` (only shown on Bricks 2.2+) slices the array to N items per page, tracked via a global (`$bricksextras_gallery_total_images`) that BricksExtras' own pagination filters (`bricks/pagination/*`) read from — but this only makes the *count/slicing* side correct for whatever page is requested. It does not on its own produce any way to reach page 2.

**A separate native Bricks `pagination` element is required** as a sibling of the loop element, with `queryId` set to the loop element's own id and `ajax: true`:

```json
{ "name": "pagination", "settings": { "queryId": "<loop element's own id>", "ajax": true } }
```

`ajax: true` is required here, not optional the way it might be for a normal `WP_Query`-backed post loop — this loop type isn't a real `WP_Query`, so there's no underlying permalink-based paging to fall back to.

**Consuming the image inside the loop: use `{post_id}` on the child element's image control, not an array-loop tag.** Each iteration binds a real attachment context (same convention as a native media/attachment Posts loop), so an `image` element inside reads `"image": {"useDynamicData": "{post_id}"}`. Don't reach for `{query_array:raw}` or any other array-loop tag here — this loop type isn't the generic `array` object type, even though the underlying data (an ACF gallery field) is array-shaped.

**This JSON is an example, not the schema.** Check `references/elements/query-loop-extras.json` (or the live schema ability, per `bricksextras-start-here`) before building from it — do not copy settings out of this block without confirming they still exist and mean what's shown here.

```json
{
  "name": "block",
  "settings": {
    "hasLoop": true,
    "query": { "objectType": "queryLoopExtras" },
    "extrasQuery": "gallery",
    "x_gallery_data": "{acf_gallery}"
  },
  "children": [
    { "name": "image", "settings": { "image": { "useDynamicData": "{post_id}" } } }
  ]
}
```

## `related` — related posts by shared taxonomy terms

`post_type` (or `any`), `count` (1–4 max), `order`/`orderby` (`orderby` options come from Bricks' own `queryOrderBy` list), `taxonomies` (multi-select, default `category`+`post_tag`) — posts sharing any term in any selected taxonomy with the current post, excluding the current post itself (`post__not_in`).

## `wpmenu` — loop over a WordPress nav menu's items

`menuSource`: `dropdown` (pick a real menu via `menu`, a dropdown only populated in real builder context) or `dynamic` (`x_menu_id` — a dynamic-data-capable text field accepting a menu name/slug/ID). `x_menu_filter_type`: `all`/`top` (top-level items only)/`children` (items whose parent matches `x_menu_parent_id`). `x_menu_parent_id` supports the `{x_menu_item_id}` dynamic tag to reference the current item's ID when this loop is itself nested inside another menu-item loop (building a multi-level menu structure).

## `favorite` — the query-side counterpart of the `xfavorite` button element

Loops over the current user's favorited posts (see `bricksextras-xfavorite` for the button/list-writing side). `post_type` selects which list (defaults to `post`). `x_favorites_orderby`: `post__in` (the order items were added, default), `none`, `ID`, `author`, `title`, `date`, `modified`, `rand`, `meta_value`/`meta_value_num` (pairs with `x_favorites_meta_key`). `x_favorites_order`: `ASC`/`DESC`.

**`newList`/`listSlug` here must exactly match the same fields on the `xfavorite` button** that wrote to that list — they resolve to the identical `{post_type}__{slug}` list-name key (same lowercasing/underscore-replacing logic as the button side). Using this loop with a `newList`/`listSlug` that doesn't match any button's list simply returns an empty result, with no error. Each returned post gets `in_favorites_loop = true` set on it.

## Build workflow

1. Put `hasLoop: true` + `query: {"objectType": "queryLoopExtras"}` on the looping element (container/block/div/xdynamictable), same placement rule as any other query loop — the *loop* goes on the repeating child, not a wrapping parent.
2. Set `extrasQuery` to the specific source needed, then only the fields relevant to that source — the others are inert.
3. For `favorite`, make sure `post_type`/`newList`/`listSlug` exactly match whatever the corresponding `xfavorite` button(s) use — mismatches silently return nothing rather than erroring.
4. For `adjacent`, confirm the loop is placed somewhere with a real current post in scope (single post/page template), not an archive.
