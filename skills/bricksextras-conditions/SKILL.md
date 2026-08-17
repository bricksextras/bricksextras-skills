---
name: bricksextras-conditions
description: "Use when adding a BricksExtras-specific element display condition (_conditions) that isn't in Bricks core's own condition list: post ancestor/category/tag, page type, taxonomy term checks, loop item number, date/datetime field comparisons, favorites-loop membership, language/translation plugin conditions, and similar general-purpose x_* conditions. Covers value-format gotchas not obvious from the key name. For WooCommerce-specific x_* conditions (cart/product/purchase checks), see a separate skill."
---

**Requires:** BricksExtras 1.7.3+, Bricks 2.4+

# BricksExtras: General element conditions (`x_*`)

BricksExtras registers ~40 general-purpose condition types on top of Bricks' native `_conditions` system, via the real Bricks filters `bricks/conditions/groups`/`bricks/conditions/options`/`bricks/conditions/result` (a genuine, documented extension point). `_conditions` is a standard Bricks settings key, so it should be writable through whatever general element-settings-update ability the current MCP connection exposes — but verify this on a connection you haven't confirmed it on yet, don't assume. On the native Bricks MCP, a dedicated `bricks/update-element-conditions` ability exists, with BricksExtras' keys merged into its own validator's allow-list (via the Abilities API's `bricks/abilities/element_conditions/allowed_keys` filter).

## Value-format gotchas not obvious from the key name

- **`x_loop_item_number`** — `compare` isn't limited to the usual `==`/`!=`/etc: it also accepts `every` (value `N` → condition met on every Nth loop item) and `modulo`, which itself takes **two different value syntaxes**: `"3:1"` (divisor:remainder) or `"% 3 = 1"` (a traditional modulo-pattern string). Any other `compare` value falls through to a plain integer comparison against the current 1-indexed loop position.
- **`x_date_field_value`/`x_datetime_field_value`** — `value` must be a **bare dynamic-data tag** (e.g. `{acf_event_date}`), not a literal date string. The code does `str_replace('}', ':Y-m-d}', $value)` (or `:Y-m-d H:i:s}` for the datetime variant), literally injecting a date-format modifier into the tag before rendering it — so the tag must have no existing modifier of its own. An empty `value` returns `true` (condition considered met) rather than false — a safe default worth knowing when debugging why a condition seems to have no effect.
- **`x_body_classes`** — `value` must be exactly **one literal class name** (`in_array($value, get_body_class(), true)`, strict single-value match) — no comma-separated list support despite being a plain text field.
- **`x_post_ancestor`** — `value` **does** support a comma-separated list of post IDs (`explode(',', $value)`, matches if the current post descends from *any* of them) — the opposite pattern from `x_body_classes`, so don't assume single-vs-multi consistency across conditions.
- **`x_author_has_cpt_entry`/`x_cpt_has_at_least_1_entry`** — `value` is a literal **post type slug** (e.g. `"post"`, `"product"`), not a label.
- **`x_post_category`/`x_post_tag`/`x_current_taxonomy_term_is`/`x_current_taxonomy_term_is_descendant_of`** — `value` is always the numeric **term ID**, even though the non-`_id`-suffixed control renders as a searchable dropdown of term names in the builder — the dropdown's underlying value is the ID either way. When building via MCP/JSON directly (no dropdown to search), prefer the explicit `_id`-suffixed variant (`x_current_taxonomy_term_is_id`, `x_current_taxonomy_term_is_descendant_of_id`) since it's unambiguous about expecting a plain ID string — both variants are functionally identical at the value level.
- **`x_has_child_category`** — only evaluates meaningfully on a **top-level** category archive page (`category_parent === 0`); on a subcategory archive, it always evaluates as "no child category" regardless of whether that subcategory actually has children of its own — a real limitation, not a bug to route around with different values.
- **`x_page_type`**'s `value` options include a large fixed set (`front_page`, `blog`, `post`, `singular`, `single`, `archive`, `category`, `tag`, `author`, `tax`, `date`, `error`, plus `wc`/`wc_shop`/`wc_product`/etc. when WooCommerce is active) — each maps directly to a core WP conditional tag (`is_front_page()`, `is_single()`, etc.); unmatched or omitted values fall back to `is_page()`.

## Standard numeric comparisons

Several conditions (`x_post_comment_count`, `x_loop_item_number`'s non-`every`/`modulo` branch, and the WooCommerce-numeric ones covered in the separate WooCommerce conditions skill) share the same `compare` operator set: `==`, `!=`, `>=`, `<=`, `>`, `<`, applied via a shared internal helper — nothing element-specific to configure beyond picking the right operator and a numeric `value`.

## Build workflow

1. Confirm the condition key is actually enabled on this site before relying on it. On the native Bricks MCP, an invalid key is rejected by the validator's accepted-keys list.
2. For date/datetime field conditions, set `value` to a bare dynamic-data tag pointing at the field — never a literal date string.
3. For term/category/tag/post conditions, use numeric IDs (prefer the explicit `_id`-suffixed key when one exists) — never term/post names.
