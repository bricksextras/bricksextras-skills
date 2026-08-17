---
name: xstarrating
description: "Use when building or debugging the Star Rating element (xstarrating) from BricksExtras: renders a row of marked/half-marked/empty star icons from a numeric rating, with two structurally different rendering modes (icons vs. color/percentage-fill). Covers the half-star rounding behavior, dynamic-data support on the rating fields, and the aria-label using the unrounded value."
---

**Requires:** BricksExtras 1.7.3+ with xstarrating enabled

# BricksExtras: Star Rating (xstarrating)

Shipped by the **BricksExtras** plugin. Not nestable. Renders `totalStars` icons, with `starRating` of them shown as marked (in half-star increments).

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xstarrating.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xstarrating` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `starRating`/`totalStars` are `text` controls, not `number` — but accept dynamic data

Both fields are typed as plain `text` (their `min`/`step` schema properties are inert — those only function on `number`-type controls, so don't expect min/step enforcement in the builder UI). They default to `4`/`5` respectively if left unset. Either field accepts a dynamic data tag (anything containing `{`) and resolves it via `render_dynamic_data_tag()` before use — e.g. binding `starRating` to `{average_rating}` from a review/CMS field works directly, no separate "dynamic" toggle needed.

## Rating is rounded to the nearest half star for display — the aria-label is not

`render()` computes `round( $rating * 2 ) / 2` for the actual star display, so any input rating (e.g. `4.3` or `4.7`) always visually renders as whole or half stars only — finer granularity isn't possible. The `aria-label` ("Rating: X out of Y stars"), however, is built from the **original, unrounded** rating value — so screen readers announce the precise number even though the visual stars only show the nearest half.

## `iconBehaviour` — two structurally different render modes, not just a style swap

- **`icons`** (default) — three independent icon controls: `markedIcon`, `halfmarkedIcon`, `icon` (the empty-star icon), each with their own icon/color. Stars are built by literally repeating whichever icon markup matches each star's state.
- **`color`** — uses only `markedIcon`'s icon shape for every star position (marked, half, and empty all reuse the same glyph), and instead achieves the marked/unmarked look via CSS custom properties (`--x-star-color`, `--x-star-empty-color`, `--x-star-percentage`) that fill/color the shape by percentage, scoped to `&[data-x-icon=color]` on the root. `halfmarkedIcon`/`icon` (empty) controls are hidden in this mode since they're unused.

Because these are genuinely different rendering strategies (not the same markup with different CSS), switching `iconBehaviour` after configuring per-icon settings means re-checking which controls actually apply — `fullColor`/`empytyColor` (color mode) vs `markedIcon`/`halfmarkedIcon`/`icon` (icons mode) are mutually exclusive via `required` gating.

**Default to `color` unless told otherwise.** It only needs one icon (`markedIcon`) to represent full, half, and empty states via CSS fill percentage — `icons` mode instead requires sourcing three visually-matching icons (full, half, empty) from whatever icon library is in use, which is more setup and more likely to look mismatched.

**`markedIcon` has no render-time fallback in either mode — omitting it renders a completely blank element, not just an unstyled one.** Its schema `default` (`fas fa-star`) is UI-only, same as every other icon control in BricksExtras. Since `color` mode reuses `markedIcon` for every star state, skipping it there means nothing renders at all: no marked stars, no half star, nothing. Always set `markedIcon` explicitly.

## Other

- `iconSize`/`iconGap`/`iconMargin` control the layout/sizing of each star icon directly.
- `data-x-star-rating` (the original numeric rating) and `data-x-icon` (the current `iconBehaviour`) are written onto the root as real attributes — useful for custom CSS/JS hooking into a specific rating value or mode.

## Rendered DOM (for custom CSS/targeting)

Fully server-rendered, no JS dependency. `starRating: 3.5`, `totalStars: 5` (3 marked, 1 half, 1 empty) — each star's wrapper class reflects its own state, the icon inside differs only by `iconBehaviour`:

**`color` mode** — every star reuses `markedIcon`'s exact icon, state is CSS-driven via the root's `--x-star-percentage`:

```html
<div class="brxe-xstarrating" aria-label="Rating: 3.5 out of 5 stars" role="img" data-x-star-rating="3.5" data-x-icon="color" style="--x-star-percentage: 50%">
  <div class="x-star-rating_star-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star-half-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star"><i class="fas fa-star"></i></div>
</div>
```

**`icons` mode** — same wrapper classes, but each state gets its own genuinely different icon (`markedIcon`/`halfmarkedIcon`/`icon`):

```html
<div class="brxe-xstarrating" data-x-icon="icons" style="--x-star-percentage: 50%">
  <div class="x-star-rating_star-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star-marked"><i class="fas fa-star"></i></div>
  <div class="x-star-rating_star-half-marked"><i class="fas fa-star-half-alt"></i></div>
  <div class="x-star-rating_star"><i class="far fa-star"></i></div>
</div>
```

Notes:

- **The empty-star wrapper class is `x-star-rating_star`, not `x-star-rating_star-empty`** — easy to guess wrong since marked/half both follow an obvious `-marked`/`-half-marked` naming pattern that the empty state doesn't continue.
- `--x-star-percentage` is written on the root regardless of `iconBehaviour` — present (and unused) even in `icons` mode, not conditionally omitted.

---

## Build workflow

1. Set `starRating`/`totalStars` as plain numbers, or bind either to dynamic data directly (e.g. `{average_rating}`) — no separate toggle required.
2. Choose `iconBehaviour` first, then only configure the controls relevant to that mode — the other mode's icon/color controls have no effect. Default to `color` unless the build specifically calls for custom marked/half/empty icon shapes.
3. Always set `markedIcon` explicitly — without it, the element renders nothing at all, in either mode.
3. Don't expect finer than half-star visual precision regardless of the input rating's actual decimal value.
