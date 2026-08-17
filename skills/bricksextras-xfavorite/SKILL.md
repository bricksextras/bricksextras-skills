---
name: xfavorite
description: "Use when building or debugging the Favorite Button element (xfavorite) from BricksExtras: an AJAX-backed add/remove/clear/count control for per-user or per-visitor favorite/wishlist lists. Covers when to use each of the four buttonType modes, list scoping via postType/newList, and why icon controls are safe to omit here unlike most BricksExtras elements."
---

**Requires:** BricksExtras 1.7.3+ with xfavorite enabled

# BricksExtras: Favorite Button (xfavorite)

Shipped by the **BricksExtras** plugin. Nestable. Persists a "favorites" list per list-name, stored in user meta for logged-in users or a cookie for logged-out users (only reachable when `access: "all"`).

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xfavorite.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xfavorite` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `buttonType` — four modes for different places in a favorites flow, not interchangeable

- **`add_remove`** (default) — a toggle button bound to **the current post** (via `get_the_ID()`). Use this **inside a query loop of posts being favorited** (a product/post card grid, a single post page, etc.) — one instance per post, letting the user add or remove that specific post. Its pressed/unpressed appearance (`aria-pressed`, icon, text) is driven by whether that post is already in the list.
- **`remove`** — also bound to the current post, but always renders in a fixed "remove" state (no toggle logic). Use this **inside the query loop on the favorites-listing page itself** (where the loop is querying the user's own saved posts) — a per-row "remove this one from my list" action, since being in the list is already guaranteed by the fact it's being displayed there.
- **`clear`** — a single button to wipe an entire list at once. Use this **once on the same favorites-listing page** (not inside the loop) to clear everything matching that list's post type/list-name in one action.
- **`count`** — not a button (renders `<span>`, or `<a>` if `link` is set); just displays how many items are in a given list. **The server-rendered count in the initial HTML is always `0`** — this is expected, not a bug: `favorite.js` reads the real count client-side from the user's actual data (AJAX/cookie) and overwrites it on load, for both logged-in and logged-out visitors. Don't read the raw HTML as the real count.

## List scoping — `postType`/`newList`/`listSlug`

- `postType` left as `default` auto-resolves: inside a running query loop, it uses that loop's actual post type; outside any loop, it falls back to `post`.
- `newList` + `listSlug` create an **additional, separately-tracked list** under `{post_type}__{slug}` — use this when the same post type needs more than one independent list (e.g. a "wishlist" and a separate "compare" list of the same post type), not for switching which post type is tracked.
- `listMaximum` caps a list's size; when set, `data-x-list-max` is written and (in `add_remove` mode, if `maxReachedButtonText` is set) the button's text switches once the cap is hit.

## Icon controls have real PHP fallbacks — unlike most BricksExtras icon controls, these are safe to leave unset

`addIcon`/`removeIcon`/`clearedIcon` each fall back to a real hardcoded inline SVG in `render()` if left empty — this is the opposite of the usual BricksExtras rule (schema icon defaults are normally UI-only and render nothing if omitted). `maybeIcons: "disable"` is the only way to remove icons entirely.

## Known rough edges

- **`maybeToolip`** is the literal setting key for the tooltip enable/disable toggle — note the typo (missing second "t" in "Tooltip"), not `maybeTooltip`.

## Rendered DOM (for custom CSS/targeting)

`add_remove` button, unpressed state, both icons always present in the markup regardless of state:

```html
<button class="x-favorite" aria-pressed="false" aria-label="Add item to favorite" data-x-click="" aria-expanded="true">
  <span class="x-favorite_text"><span class="x-favorite_text-inner">Add to favorites</span></span>
  <span class="x-favorite_icons">
    <span class="x_favorite-added-icon x-favorite_icon" aria-hidden="true"><svg>...</svg></span>
    <span class="x_favorite-removed-icon x-favorite_icon" aria-hidden="true"><svg>...</svg></span>
  </span>
</button>
```

Both icon spans (`x_favorite-added-icon`, `x_favorite-removed-icon`) render at all times — `favorite.js` toggles which one is visible (and the text/`aria-pressed` state) client-side as the list membership changes, rather than the server swapping markup per state.

**The tooltip's server-rendered markup (a set of `tippy-root`/`tippy-box`/`tippy-content` divs) only renders in the Bricks builder canvas's preview rendering — never the real frontend.** On the actual frontend, `maybeToolip: enable` produces a genuine Tippy.js tooltip, built at runtime (same library/theme as `xpopover`/`xmediacontrol`) and appended to the element's own `.brxe-xfavorite` wrapper:

```html
<div data-tippy-root id="tippy-1" style="position: absolute; ...">
  <div class="tippy-box" data-state="visible" data-theme="extras" data-animation="extras" role="tooltip" data-placement="bottom">
    <div class="tippy-content">Add to favorites</div>
    <div class="tippy-arrow"></div>
  </div>
</div>
```

Style via `.tippy-box[data-theme="extras"]` scoped under the element's own id/class.

## The generic `data-x-favorite-count` attribute — a second, independent count display mechanism

Separately from the dedicated `count` button type, `favorite.js` also updates the text content of **any** element carrying a `data-x-favorite-count` attribute (a colon-separated list of post types/list names, e.g. `"post:product"`, to sum counts across multiple lists) — this is meant for arbitrary custom markup (e.g. a header cart-style icon badge) rather than requiring an actual `xfavorite` element. It's excluded from elements inside toast notifications, so it only updates persistent on-page counters.

## Build workflow

1. Decide which of the four `buttonType` modes fits the placement: `add_remove` inside a general posts loop, `remove` inside the favorites-listing loop, `clear` once on the favorites-listing page, `count` anywhere a live tally is needed.
2. Leave `postType` as `default` when placing inside a loop — it resolves automatically to that loop's post type.
3. Use `newList`/`listSlug` only when the same post type needs more than one independent favorites list.
4. Icon controls can be safely left unset if the built-in fallback icons are acceptable — this element doesn't need explicit icon values the way most other BricksExtras elements do.
