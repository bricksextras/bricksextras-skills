---
name: xmediaplaylist
description: "Use when building or debugging the Media Playlist element (xmediaplaylist) from BricksExtras: clickable playlist-item buttons that feed a track/video to a sibling xmediaplayer in playlist mode. Covers why items render as empty buttons without nested children, the section/selector/component connection mechanism (set on the player, not the item), and active/playing/paused state styling."
---

**Requires:** BricksExtras 1.7.3+ with xmediaplayer and xmediaplaylist elements enabled

# BricksExtras: Media Playlist (xmediaplaylist)

Shipped by the **BricksExtras** plugin, as a sibling to `xmediaplayer` (see skill `xmediaplayer`). One `xmediaplaylist` instance = one playlist entry (one track/video). It's a nestable leaf-style element: clicking it hands its configured media data to a connected player — it does not embed or play anything itself.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xmediaplaylist.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xmediaplaylist` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## It renders as a genuinely empty `<button>` unless you nest visible content yourself

It renders as `<button {attrs}>{children}</button>` — nothing else. All of the item's own settings (`title`, `artist`, `image`, `src`, etc.) are written out only as `data-x-*` attributes on that button (`data-x-title`, `data-x-artist`, `data-x-poster`, `data-x-src`, `data-x-stream-type`, `data-x-clip-start`/`data-x-clip-end`, `data-x-texttracks`, `data-x-chapters`, `data-x-srcs`, `data-x-downloads`) — they are **not** rendered as visible text/images anywhere. An `xmediaplaylist` built with settings only (no children) renders as a real, clickable, but completely invisible button.

**You must nest your own visual content as children** — typically an `image` element for the thumbnail and a `heading`/`text-basic` for the title — duplicating whatever you already put in the item's `title`/`image` settings, since nothing binds those settings to visible markup automatically. This matches the plugin's own `get_nestable_children()` default (what a person gets when adding the element via the builder UI, not auto-applied via MCP — see `bricksextras-start-here`'s defaults rule): a plain `block > text-basic` saying "Item description goes here", i.e. static placeholder content, not a dynamic-data binding.

The item's own `flexWrap`/`direction`/`justifyContent`/`alignItems`/`columnGap`/`rowGap` controls (group "layout") apply directly to the button root (`css selector: ""`), so you can arrange a thumbnail + title row without an extra wrapper block — e.g. `direction: row`, `alignItems: center`, `columnGap: "0.75rem"`.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplaylist.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xmediaplaylist",
  "label": "Big Buck Bunny",
  "settings": {
    "provider": "video",
    "sourceType": "url",
    "src": "https://www.youtube.com/watch?v=aqz-KE-bpKQ",
    "title": "Big Buck Bunny - Trailer",
    "direction": "row",
    "alignItems": "center",
    "columnGap": "0.75rem"
  },
  "children": [
    { "name": "text-basic", "settings": { "text": "Big Buck Bunny - Trailer" } }
  ]
}
```

---

## Connecting to a player — configured on the player, not the item

The playlist item has **no targeting field of its own**. The connection is entirely driven by settings on the `xmediaplayer` side, under its "Playlists" group:

- `playlistMode: true` — required for any of this to matter.
- `whichPlaylist` — how the player finds its `xmediaplaylist` items: `"section"` (default — any playlist items within the same enclosing Bricks section), `"selector"` (use `playlistSelector`, a plain CSS selector — same class-over-`_cssId` rule as everywhere else in Bricks, see `bricksextras-start-here`), or `"component"` (within the same component instance).
- `playlistSelector` — only read when `whichPlaylist: "selector"`.
- `componentScope` — string `"true"`/`"false"` (not boolean), only relevant with `whichPlaylist: "selector"` — same pattern as `xproslidercontrol`/`xproslider` sync scoping.
- `playListNext` — auto-advance to the next item when the current one ends.
- `playListLoop` — loop back to the first item after the last one.

**Works with the `"section"` default and no explicit selector**: placing the player and its `xmediaplaylist` items inside the same `section` element is enough — the player automatically loads the first item as its active source, and each item renders `aria-controls` matching the player's own element id.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplaylist.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xmediaplayer",
  "settings": {
    "layoutType": "type_one",
    "playlistMode": true,
    "whichPlaylist": "section",
    "playListNext": true,
    "playListLoop": true,
    "smallControls": [
      { "control": "previous" }, { "control": "title" }, { "control": "next" }
    ]
  }
}
```

Include `previous`/`next` controls in the player's control repeater (`smallControls` for `type_one`, `controls` for `type_two` — see `bricksextras-xmediaplayer`) so playlist navigation is actually reachable from the built-in UI.

---

## Two unrelated `hasLoop` uses on this element — don't conflate them

`xmediaplaylist` has its own `hasLoop` control, but it drives **chapters** (the `data-x-chapters` timeline data for that one track/video), not the playlist items. Enabling it (with `query` and the `chapterText`/`chapterStart`/`chapterEnd` dynamic-tag fields) generates one chapter marker per loop row, as an alternative to manually filling out the element's `chapters` repeater. This is a real, self-contained loop that belongs directly on the element.

**Populating multiple playlist items from a query loop is a separate, unrelated need, and does not use that same `hasLoop` control.** For that, the loop goes on a plain Bricks `block` wrapper, and the repeating element (`xmediaplaylist`) is nested inside that block, same as the `xproslider` slide pattern — putting `hasLoop`/`query` on the `xmediaplaylist` element itself for this purpose does nothing (it only affects chapters).

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplaylist.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "block",
  "settings": {
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": ["video"], "posts_per_page": 10 }
  },
  "children": [
    {
      "name": "xmediaplaylist",
      "settings": {
        "provider": "video",
        "sourceType": "url",
        "src": "{video_url}",
        "title": "{post_title}",
        "direction": "row",
        "alignItems": "center",
        "columnGap": "0.75rem"
      },
      "children": [
        { "name": "text-basic", "settings": { "text": "{post_title}" } }
      ]
    }
  ]
}
```

The item's `src`/`title`/`image` settings — and the nested visual content, since nothing auto-binds those settings to visible markup (see above) — both need dynamic tags bound to the current loop post, not literal values. `{video_url}` above is a placeholder for whatever field actually holds the URL on this site — check the site's actual custom fields/dynamic-data tags before building rather than guessing a name.

The player side (`whichPlaylist`, etc.) doesn't change — it finds `xmediaplaylist` elements at render time regardless of whether they exist statically or were duplicated by a loop.

---

## Active/playing/paused state

Each item exposes its own state as attributes: `data-x-item-active` (present on the current item), `data-x-item-playing` / `data-x-item-paused`, `aria-current`, plus `aria-controls` linking to the player. Style these via the item's own `activeStyles` group controls rather than hand-writing attribute selectors:

- `activeBackgroundColor` / `activeBorder` / `activeBoxShadow` / `activeTypography` → `&[data-x-item-active]`
- `activePlayingBackgroundColor` / etc. → `&[data-x-item-active][data-x-item-playing]`
- `activePausedBackgroundColor` / etc. → `&[data-x-item-active][data-x-item-paused]`

---

## Never do

- Do not expect an `xmediaplaylist` item to show any visible content from its own settings alone — `title`/`artist`/`image` only populate `data-x-*` attributes; nest real content (image + heading/text) as children.
- Do not set `whichPlaylist`/`playlistSelector`/`playListNext`/`playListLoop` on the playlist item — they're player-level (`xmediaplayer`) settings, even though the item is what they act on.
- Do not assume `whichPlaylist` needs a selector — its default, `"section"`, works with zero extra config as long as the player and its playlist items share an enclosing section.
- Do not expect `hasLoop`/`query` directly on `xmediaplaylist` to build a dynamic playlist — that combination only drives chapters. Wrap the element in a plain `block` and put the loop there instead, same as the `xproslider` slide pattern.
