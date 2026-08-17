---
name: dynamictags
description: "Use when building or debugging any BricksExtras custom dynamic-data tag (all `{x_*}` tags — reading time, post terms list, URL parameter, loop index, parent loop index, menu item label/url/description/classes/target/id, favorite IDs/counts, or attachment alt text/title/url/description/caption). Covers the colon-argument syntax each tag accepts and which tags require a specific active loop/query context to resolve at all (and silently render empty otherwise, with no error)."
---

**Requires:** BricksExtras 1.7.3+

# BricksExtras: Custom Dynamic Data Tags (`{x_*}`)

19 custom dynamic-data tags the plugin registers on top of Bricks' own tag system, via the real Bricks filters `bricks/dynamic_tags_list` (builder registration), `bricks/dynamic_data/render_tag` (single-tag rendering), and `bricks/dynamic_data/render_content`/`bricks/frontend/render_data` (tags embedded inside larger text). All genuine extension points — nothing here is a parallel/private tag system.

**Fully discoverable live, no PHP needed to find them:** `bricks/list-dynamic-data-tags` (or whatever equivalent ability the current MCP connection exposes) returns every one of them alongside Bricks core's own tags — confirmed by paging through the full list rather than trusting only the first page. They're distinguishable by their labels ending in `(extras)` and an empty `provider` field (`""`, not `"wp"`/`"acf"`/etc. — worth knowing if you're filtering by `provider`, since filtering that field to a real provider key silently excludes all of these). Groups are `Post` (for the two post-specific tags) and `Extras` (everything else) — not a `BricksExtras`-named group, despite the plugin origin.

**What isn't discoverable live: per-tag argument syntax and context requirements.** `bricks/list-dynamic-data-tags`'s `includeModifiers` only documents Bricks' own generic modifier grammar (`:plain`, `:link`, `@fallback`, etc.) — none of which apply here. These tags use their own, unrelated colon-argument convention (below), and `bricks/preview-dynamic-tag` is the only way to verify a specific tag's behavior live — it's the right tool to reach for per tag, but won't tell you the rules up front. That's what this skill is for.

## Argument syntax: everything after the first `:` is one opaque string, not per-tag-typed modifiers

`{tag_name:whatever-comes-after}` — the whole substring after the first colon is captured as one raw argument string, then each tag's own code decides how to interpret it (comma-split, treat as a single value, etc.). There's no shared modifier grammar across these tags the way Bricks core tags share `:plain`/`@fallback`/etc. — read the per-tag entry below, don't assume a pattern from one tag applies to another.

## Post tags

- **`{x_post_reading_time}`** — optional arg `singular,plural,wpm` (comma-separated, all three optional with defaults `minute`/`minutes`/`225`). Reads the *current* post's `post_content` (not the loop item unless the loop has set post data), splits on whitespace, divides by words-per-minute, rounds up. `{x_post_reading_time:min,mins,50}` confirmed live → `"1 min"`.
- **`{x_post_terms_list}`** — optional arg is a taxonomy slug (default `category`). Comma-space-joined term list, tags stripped.

## General

- **`{x_url_parameter}`** — **deprecated; prefer Bricks core's own `{url_parameter}` instead.** BricksExtras shipped this before Bricks core added its own native equivalent — they are not aliases of each other, but `{url_parameter}` now covers the same job as a first-party tag. Optional arg is the `$_GET` key to read (default `s`), same convention as the core tag. Don't reach for this one in new builds; it's kept only for backward compatibility with content that already uses it.
- **`{x_est_year_current_year}`** — no arg → current year alone (confirmed live → `"2026"`, matching server date). With an arg (a literal year, e.g. `{x_est_year_current_year:2020}`) → `"{arg} - {current year}"` unless they're equal, in which case just the year (confirmed live → `"2020 - 2026"`). The arg is a **literal year string**, not a dynamic tag or date field — pass the actual founding/established year directly.
- **`{x_loop_index}`** — optional numeric-offset arg, added to the current query loop's 0-indexed position. **Requires an active Bricks query loop context to resolve at all** — confirmed live outside any loop: renders as empty string (`isEmpty: true`), not the raw tag, and not an error. Only meaningful inside a query-loop element's own children.
- **`{x_parent_loop_index}`** — optional arg is an ancestor level (default `1` = immediate parent loop). For when this tag is used inside a loop that's itself nested inside another loop, and you want the *outer* loop's index rather than the current one's `{x_loop_index}`/`{query_loop_index}`.

## Menu item tags — all five require an active `wpmenu`-type query loop specifically

`{x_menu_item_label}`, `{x_menu_item_url}`, `{x_menu_item_description}`, `{x_menu_item_classes}`, `{x_menu_item_target}`, `{x_menu_item_id}` all read from Bricks' current "loop object," which only exists inside a query loop — and in practice, the loop object shape these tags expect (`->title`, `->url`, `->description`, `->classes`, `->target`, `->ID`) only comes from the `queryLoopExtras` element's `wpmenu` sub-type (see `bricksextras-query-loop-extras`), not from a generic post/term loop. Confirmed live outside any loop context: renders empty (`isEmpty: true`), same silent-empty pattern as `x_loop_index`. `x_menu_item_target` has a real fallback (`_self` when the menu item has no explicit target) — the one tag in this group that doesn't just go empty when its own specific value is unset.

`x_menu_item_classes` joins the item's classes array with a single space; empty array → empty string, not an error.

These are exactly the tags used by the `x_menu_parent_id` nested-submenu pattern in `bricksextras-query-loop-extras` — `{x_menu_item_id}` referencing the *current* (outer) loop's item to filter the *inner* (children) loop.

## Favorites tags

- **`{x_favorite_ids}`** — optional arg is a list name (default `post`, matching `xfavorite`'s own list-naming convention — see `bricksextras-xfavorite`). Renders as a literal bracketed string, e.g. `"[12, 45]"` — **not JSON**, don't parse it as such. Empty list renders as the literal string `"[0]"`, not `"[]"` — a real, deliberate placeholder value, not a bug to route around.
- **`{x_favorite_count}`** — same list-name arg, but **supports multiple list names merged together via a second, nested colon**: `{x_favorite_count:post:page}` sums favorites across both the `post` and `page` lists. This is a different colon from the outer `tag:arg` split — the whole `post:page` string is the one argument, then split again internally. Always renders wrapped in `<span data-x-favorite-count="{list}">{count}</span>` — real HTML markup, not a bare number, confirmed live even at zero (`<span data-x-favorite-count="post">0</span>`, not empty). The `data-x-favorite-count` attribute is a live hook some other part of the plugin's JS likely updates on the fly — treat this tag's output as markup, not text, when using it somewhere that would escape/strip HTML.
- **`{x_favorite_count_number}`** — same list-name/multi-list arg syntax as `x_favorite_count`, but returns a bare integer (`0`, `3`, ...) with no wrapping markup — use this one instead when the count needs to be a plain number (e.g. inside a condition's `value`, or concatenated into other text).

## Attachment tags

`{x_attachment_alt_text}`, `{x_attachment_title}`, `{x_attachment_url}`, `{x_attachment_description}`, `{x_attachment_caption}` all resolve against the current query loop's attachment item, including inside a `queryLoopExtras` `gallery` loop — use any of them the same way, on an element nested inside that loop.

## The silent-empty pattern: know it applies here before debugging "nothing showing"

None of these 19 tags fall back to displaying the raw `{tag}` text when their required context is missing — they resolve to an **empty string**. If a value silently isn't showing, the first thing to check is whether the element carrying the tag is actually inside the loop/context that tag needs (per the sections above) — not whether the tag name is spelled right or the plugin is active.

## Build workflow

1. Confirm the tag is one of these 19 via whatever tag-listing ability the current MCP connection exposes (e.g. `bricks/list-dynamic-data-tags` on the native Bricks MCP; or paginate — they don't all fit on page 1 alongside Bricks core's own tags) before assuming it exists; the `(extras)` label suffix and empty `provider` field are the tell.
2. Check this skill for the tag's argument syntax and context requirement — none of it is visible from the live tag list itself.
3. For any context-dependent tag (`x_loop_index`, `x_parent_loop_index`, all six `x_menu_item_*`), place it only inside the specific loop type it needs, and verify with whatever dynamic-tag preview/resolve ability the current MCP connection exposes (e.g. `bricks/preview-dynamic-tag` on the native Bricks MCP) in that real context — an empty preview outside a loop is expected and not proof the tag is broken.

## Never do

- Don't assume a `{x_*}` tag's colon-argument follows Bricks core's own modifier grammar (`:plain`, `:{number}`, `@fallback`, etc.) — each tag defines its own argument meaning independently.
- Don't parse `{x_favorite_ids}`'s output as JSON — it's a literal `"[id, id]"` string, including the placeholder `"[0]"` for an empty list.
- Don't treat `{x_favorite_count}`'s output as plain text needing to be escaped/stripped — it's real `<span>` markup by design; use `{x_favorite_count_number}` instead when a bare number is required.
- Don't place `x_loop_index`, `x_parent_loop_index`, or any `x_menu_item_*` tag outside the loop context they need and then conclude the tag is broken when it renders empty — that's the designed behavior, verify context first via `bricks/preview-dynamic-tag`.
