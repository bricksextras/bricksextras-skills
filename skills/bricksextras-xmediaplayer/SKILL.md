---
name: xmediaplayer
description: "Use when building, debugging, or styling Media Player (xmediaplayer) from BricksExtras: video/audio players built on VidStack, including custom control layouts using sibling xmediacontrol element and playlist mode using sibling xmediaplaylist elements. Covers built-in vs custom UI distinction and the required layout structure for custom mode. Load bricksextras-xmediacontrol alongside this one for custom layouts, and bricksextras-xmediaplaylist for playlist mode."
---

**Requires:** BricksExtras 1.7.3+ with xmediaplayer and xmediacontrol elements enabled

# BricksExtras: Media Player (xmediaplayer)

Shipped by the **BricksExtras** plugin. A nestable video/audio player built on VidStack. For a **custom control layout**, its children are `block`/`div` layout wrappers holding `xmediacontrol` elements — a non-nestable sibling element that renders exactly one piece of player UI (play button, time slider, volume, settings menu, etc.). **Load `bricksextras-xmediacontrol` alongside this skill whenever `layoutType: custom` is in play** — it covers what each `controlType` renders and how it styles; this skill covers the player itself and the layout structure those controls get placed into.

For **playlist mode** (`playlistMode: true`), the player pulls its source from sibling `xmediaplaylist` elements instead of its own `src`. **Load `bricksextras-xmediaplaylist` alongside this skill whenever playlist mode is in play** — it covers the playlist item element, why it renders empty without nested children, and the section/selector/component connection mechanism.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xmediaplayer.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xmediaplayer` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Two completely different rendering modes — this is the core thing to understand

The player's `layoutType` setting (`type_one` / `type_two` / `custom`) doesn't just restyle the same markup — it changes *what actually renders*:

- **`type_one` / `type_two` (built-in UI):** No nestable child elements are used or rendered at all. The entire control surface — top/center/bottom control zones, which buttons appear, labels, icons — is configured entirely through repeater fields living directly on the player's own settings (`controlsTop`, `controlsCenter`, `controls`, `smallControlsTop`, `smallcontrolsCenter`, `smallControls`, plus their gap/visibility settings). Nesting a `block`/`xmediacontrol` under the player while in this mode has no visible effect — the built-in UI ignores them.
- **`custom`:** The built-in UI is removed entirely. Nothing renders except the raw media surface unless real child elements are built — a `block` (or `div`) inside the player for layout, with `xmediacontrol` elements nested inside for each piece of UI. The schema's own separator description for this setting says as much: *"All default UI is removed, add a block element inside the media player for the layout. Add 'media controls' element inside block to build custom UI."*

**If a person asks for a custom control layout, `layoutType` must be set to `custom` first** — otherwise the child elements you build are inert, present in the tree but not rendered.

**Switching modes doesn't clear the other mode's settings.** A player that previously used `type_one`/`type_two` and was then switched to `custom` will still carry populated `controlsTop`/`controls`/etc. arrays in its settings — these are just inert leftovers once `layoutType: custom` is set, not a conflict to clean up.

---

## Built-in UI (`type_one` / `type_two`)

When using built-in layouts, controls are configured via repeater fields on the player settings. Each repeater item is an object with a `control` key.

**Critical: `type_one` and `type_two` use different repeater fields:**

- **`type_one`** uses: `smallControlsTop`, `smallcontrolsCenter`, `smallControls`
- **`type_two`** uses: `controlsTop`, `controlsCenter`, `controls`

This is enforced by schema `required` conditions — using the wrong set silently does nothing.

**Default control layouts (from Bricks builder):**

**`type_one` defaults:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplayer.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "smallControlsTop": [
    {"control": "mute"},
    {"control": "spacer"},
    {"control": "captions"},
    {"control": "chapters"},
    {"control": "fullscreen"},
    {"control": "settings"}
  ],
  "smallcontrolsCenter": [
    {"control": "spacer"},
    {"control": "play-large"},
    {"control": "spacer"}
  ],
  "smallControls": [
    {"control": "title"}
  ]
}
```

**`type_two` defaults:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplayer.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "controlsTop": [
    {"control": "title"},
    {"control": "spacer"}
  ],
  "controlsCenter": [],
  "controls": [
    {"control": "play"},
    {"control": "mute"},
    {"control": "time"},
    {"control": "spacer"},
    {"control": "captions"},
    {"control": "chapters"},
    {"control": "fullscreen"},
    {"control": "settings"}
  ]
}
```

**Note:** `spacer` controls create flexible gaps that push other controls apart in flex layouts.

**Example: Simple video player with `type_one`**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplayer.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xmediaplayer",
  "settings": {
    "layoutType": "type_one",
    "sourceType": "url",
    "src": "https://example.com/video.mp4",
    "_aspectRatio": "16/9",
    "smallControlsTop": [
      {"control": "mute"},
      {"control": "spacer"},
      {"control": "fullscreen"},
      {"control": "settings"}
    ],
    "smallcontrolsCenter": [
      {"control": "spacer"},
      {"control": "play-large"},
      {"control": "spacer"}
    ],
    "smallControls": [
      {"control": "time"}
    ]
  }
}
```

For playlist mode, add `"playlistMode": true` and include `previous`/`next` controls in the bottom row repeater (`smallControls` for `type_one`, `controls` for `type_two`).

---

## Custom layouts need an explicit `block` added — nothing is auto-added

Switching `layoutType` to `custom` (whether in the builder or via JSON) does not add any children by itself — a player left with no children in custom mode renders no controls at all. Building a custom layout means explicitly adding a `block` (or `div`) as the player's child to control the layout (flex/grid), then adding `xmediacontrol` elements inside it for each piece of UI. This `block` is a normal, deliberate part of the JSON you write — not something to detect or clean up after the fact.

---

## Reference pattern for a custom layout

```
xmediaplayer (layoutType: custom, sourceType: url, src: ...)
└── block "Wrapper"
    ├── block "Controls Top"
    │   ├── xmediacontrol (controlType: mute)
    │   ├── xmediacontrol (controlType: settings)
    │   └── xmediacontrol (controlType: fullscreen)
    ├── block "Controls Middle"
    │   └── xmediacontrol (controlType: play-large)
    └── block "Controls Bottom"
        └── xmediacontrol (controlType: time-slider)
```

**Only one top-level child ("Wrapper") sits directly on the player.** Everything below it — as many nested `block`/`div` layout wrappers as needed, arranged however a normal Bricks layout would be (rows, columns, absolute-positioned overlays, etc.) — holds the actual `xmediacontrol` leaves. Layout naming ("Controls Top/Middle/Bottom") is a convention, not a requirement; any layout structure works as long as `xmediacontrol` elements end up somewhere inside the Wrapper subtree.

### DOM order is the visual order

With no `order`/positioning override, flex/block layout renders children top-to-bottom in DOM order. **Row blocks must be added in the same order they should visually appear** — Top, then Middle, then Bottom for a standard overlay layout — unless CSS `order` is explicitly used to decouple visual position from DOM position.

### None of this layout is automatic — it must be set explicitly

Creating the Wrapper and row blocks with empty `settings` (no `_display`/`_direction`) leaves every control stacked vertically on top of each other — plain `block` elements have no inherent row layout, regardless of how the elements are nested. Every layer needs explicit layout settings:

- **Wrapper:** `_display: flex`, `_direction: column`, `_justifyContent: space-between` (spreads the three rows to top/middle/bottom of the available height) — plus, to actually overlay the video rather than push it down, `_position: absolute` with `_top`/`_left`/`_width: 100%`/`_height: 100%`.
- **Each row block** (Controls Top/Middle/Bottom): `_display: flex`, `_direction: row`, plus whatever `_justifyContent`/`_alignItems`/`_gap` the row's specific layout needs (e.g. `_justifyContent: flex-end` to right-align a row of icon buttons, `_justifyContent: center` + `_flexGrow: 1` for a centered play button that also claims the remaining vertical space between top and bottom rows).

Full example — the player and its complete layout in one call:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplayer.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xmediaplayer",
  "label": "My Player",
  "settings": { "layoutType": "custom", "sourceType": "url", "src": "https://example.com/video.mp4" },
  "children": [
{
  "name": "block",
  "label": "Wrapper",
  "settings": {
    "_display": "flex",
    "_direction": "column",
    "_justifyContent": "space-between",
    "_position": "absolute",
    "_top": "0",
    "_left": "0",
    "_width": "100%",
    "_height": "100%"
  },
  "children": [
    {
      "name": "block",
      "label": "Controls Top",
      "settings": {
        "_display": "flex",
        "_direction": "row",
        "_justifyContent": "flex-end",
        "_alignItems": "center",
        "_gap": "8"
      },
      "children": [
        { "name": "xmediacontrol", "label": "Volume", "settings": { "controlType": "mute" } },
        { "name": "xmediacontrol", "label": "Settings Menu", "settings": { "controlType": "settings" } },
        { "name": "xmediacontrol", "label": "Fullscreen", "settings": { "controlType": "fullscreen" } }
      ]
    },
    {
      "name": "block",
      "label": "Controls Middle",
      "settings": {
        "_display": "flex",
        "_direction": "row",
        "_justifyContent": "center",
        "_alignItems": "center",
        "_flexGrow": 1
      },
      "children": [
        { "name": "xmediacontrol", "label": "Play", "settings": { "controlType": "play-large" } }
      ]
    },
    {
      "name": "block",
      "label": "Controls Bottom",
      "settings": {
        "_display": "flex",
        "_direction": "row",
        "_alignItems": "center",
        "_gap": "8"
      },
      "children": [
        { "name": "xmediacontrol", "label": "Time slider", "settings": { "controlType": "time-slider" } }
      ]
    }
  ]
}
  ]
}
```

---

## Rendered DOM (for custom CSS/targeting)

**This is the final, hydrated DOM — what actually exists once VidStack initializes, and what a real browser inspection shows.** Captured live, `type_two`, video, icon SVGs and most runtime state attributes elided for brevity (the real thing carries many more `aria-*`/`data-*` attributes VidStack manages itself):

```html
<media-player id="brxe-{id}" class="brxe-xmediaplayer xmp-video-layout" data-x-id="{id}" data-x-layout="type_two"
              data-media-player role="region" aria-label="Video Player" data-paused data-can-play ...>
  <media-provider data-media-provider>
    <video crossorigin="anonymous" preload="metadata" src="..." playsinline></video>
    <media-poster class="xmp-poster" data-hidden>
      <img crossorigin="anonymous">
    </media-poster>
  </media-provider>
  <media-layout class="xmp-layout_type-two">
    <media-controls class="xmp-controls dark" role="group">
      <div class="xmp-controls_group xmp-controls_group_top" data-x-control-visibility="default">
        <media-title class="xmp-title repeater-item"></media-title>
        <div class="xmp-controls_spacer"></div>
      </div>
      <!-- auto-included, not from any controls repeater - see note below -->
      <div class="xmp-controls_group xmp-controls_group_time-slider" data-x-control-visibility="default">
        <media-time-slider class="xmp-time-slider xmp-slider" data-media-time-slider role="slider" aria-valuenow="0" aria-valuetext="...">...</media-time-slider>
      </div>
      <div class="xmp-controls_group xmp-controls_group_bottom" data-x-control-visibility="default">
        <media-play-button class="xmp-button xmp-play-button repeater-item" data-media-tooltip="play" aria-label="Play" role="button" aria-pressed="false" data-paused>...</media-play-button>
        <div class="xmp-volume repeater-item"><media-mute-button class="xmp-button xmp-mute-button" data-media-mute-button aria-label="Mute" aria-pressed="false">...</media-mute-button></div>
        <div class="xmp-time-group repeater-item">
          <media-time class="xmp-time" type="current">0:00</media-time>
          <div class="xmp-time-divider">/</div>
          <media-time class="xmp-time" type="duration">10:29</media-time>
        </div>
        <div class="xmp-controls_spacer"></div>
        <media-fullscreen-button class="xmp-button xmp-fullscreen-button repeater-item" aria-label="Enter Fullscreen">...</media-fullscreen-button>
        <media-menu class="xmp-menu xmp-settings-menu repeater-item" data-root>
          <media-menu-button class="xmp-button xmp-settings-button" aria-label="Settings" aria-haspopup="menu" aria-expanded="false">...</media-menu-button>
          <media-menu-items class="xmp-settings-menu-items xmp-menu-items dark" role="menu">...</media-menu-items>
        </media-menu>
      </div>
    </media-controls>
  </media-layout>
  <div class="xmp-media-features" data-x-color="dark">
    <div class="xmp-buffering-indicator"><media-spinner class="xmp-buffering-spinner">...</media-spinner></div>
    <media-gesture event="pointerup" action="toggle:controls"></media-gesture>
    <media-gesture event="click" action="toggle:paused"></media-gesture>
    <media-gesture event="keyup" action="play"></media-gesture>
    <media-captions class="xmp-captions"></media-captions>
    <media-announcer role="status" aria-live="polite"></media-announcer>
  </div>
</media-player>
```

Notes on this structure:

- **The root renders as a real custom element, `<media-player>`, not a `<div>`.** Same for many pieces inside it — `<media-provider>`, `<media-poster>`, `<media-mute-button>`, `<media-play-button>`, `<media-time>`, `<media-time-slider>`, `<media-menu>` are all genuine custom elements (VidStack's own web components), not styled divs/buttons. Selector-wise they behave like any other element (class/attribute selectors work normally), but don't expect to find plain `<button>`/`<div>` tags for these pieces.
- **The actual media surface (`<video>`) and a hydrated `<media-poster><img>...</media-poster>` only exist after VidStack initializes** — the server-rendered HTML wraps the whole `type_one`/`type_two` control layout in a `<template data-x-mediaplayer-template="{id}">`, and `<media-provider>` starts essentially empty. Reading raw source (`get-page-elements`, a non-JS HTTP fetch) is enough to confirm your settings/structure were written correctly, but won't show the real player surface or controls as they actually exist — verify against a real browser render for anything involving the visual DOM.
- **Each repeater-driven control is wrapped in an element carrying the class `repeater-item`**, alongside its own semantic class (`xmp-volume`, `xmp-time-group`, etc.) — a reliable general hook if styling needs to target "any control in this row" without listing every specific control type.
- **A `spacer` control renders as a literal empty `<div class="xmp-controls_spacer"></div>`** — pure layout, nothing to style inside it.
- **The time-slider group is a separate feature from the `smallControls`/`controls` repeater, controlled by its own setting** — `maybeTimeSliderSmall` (`type_one`) / `maybeTimeSlider` (`type_two`), both **enabled by default**. It renders on its own regardless of what's in the repeater, so adding a `time-slider` repeater item on top/center/bottom produces *two* time sliders unless the separate setting is turned off first. If a `time-slider` control is being manually placed in a specific row, disable `maybeTimeSliderSmall`/`maybeTimeSlider` to avoid the duplicate.
- **The `settings` control expands into a large, deeply-nested submenu system built entirely by VidStack** — a `media-menu`/`media-menu-items` tree with its own sub-menus (Playback speed, Quality, Captions, Accessibility), each with their own sliders/radio groups/toggles. None of this is something you build or configure through `xmediacontrol` beyond adding the one `settings` control — if custom styling needs to reach into a specific piece of it, inspect that piece live rather than guessing a selector; it's too extensive to catalog exhaustively here.
- **A `xmp-media-features` wrapper (buffering spinner, click/tap gesture zones, captions overlay, screen-reader announcer) always renders, as a sibling after `media-layout`, regardless of `layoutType` or what's in the controls repeater.** It's not part of the configurable control layout — don't look for it inside `media-controls`, and don't expect the controls repeater to affect its presence.
- Both layouts support the same three groups (`_top`/`_center`/`_bottom` — `smallControlsTop`/`smallcontrolsCenter`/`smallControls` for `type_one`, `controlsTop`/`controlsCenter`/`controls` for `type_two`). The center group only renders when its repeater setting is non-empty — this capture's `controlsCenter` was empty, which is why `.xmp-controls_group_center` doesn't appear above, not because `type_two` lacks one structurally.

### Player state attributes for CSS targeting

VidStack keeps the `<media-player>` root's state reflected as a set of boolean `data-*` attributes (and a couple of value attributes), added/removed live as playback state changes — usable directly as CSS attribute selectors (`[data-paused]`, `&[data-fullscreen]`, etc.) regardless of `layoutType`:

| Attribute | Meaning |
|---|---|
| `data-autoplay` | Autoplay has successfully started |
| `data-autoplay-error` | Autoplay has failed to start |
| `data-buffering` | Media is not ready for playback / waiting for more data |
| `data-can-fullscreen` | Fullscreen mode is available |
| `data-can-load` | Media can now begin loading |
| `data-can-pip` | Picture-in-Picture mode is available |
| `data-can-play` | Media is ready for playback |
| `data-can-seek` | Seeking operations are permitted |
| `data-captions` | Captions are available and visible |
| `data-controls` | Controls are visible |
| `data-ended` | Playback has ended |
| `data-error` | Issue with media loading/playback |
| `data-fullscreen` | Fullscreen mode is active |
| `data-ios-controls` | iOS controls are visible |
| `data-loop` | Media is set to replay on end |
| `data-media-type` | Current media type (`audio`/`video`) — value attribute, not boolean |
| `data-muted` | Volume is muted |
| `data-orientation` | Current screen orientation (`landscape`/`portrait`) — value attribute |
| `data-paused` | Playback is paused |
| `data-pip` | Picture-in-Picture mode is active |
| `data-playing` | Playback is active |
| `data-playsinline` | Media should play inline by default (iOS) |
| `data-pointer` | The user's pointer device type (`coarse`/`fine`) — value attribute |
| `data-preview` | The user is interacting with the time slider |
| `data-seeking` | User is seeking to a new playback position |
| `data-started` | Media playback has started |
| `data-view-type` | Current view type (`audio`/`video`) — value attribute |
| `data-waiting` | Media is waiting for more data to resume playback |
| `data-focus` | Player is being keyboard-focused |
| `data-hocus` | Player is being keyboard-focused or hovered over |
| `data-x-wait` | Player is waiting for a source change |

These are VidStack's own attributes, not something BricksExtras adds — the same set applies whether the layout is `type_one`, `type_two`, or `custom`.

### `maybeToolTips` is a master switch — controls have their own copy of the same setting

The player has its own `maybeToolTips` (Tooltips group, "Enable"/"Disable"), separate from the identically-named setting on each individual `xmediacontrol`, and their defaults are opposite: the player's defaults to **disabled**, each control's own defaults to **enabled**. The player-level setting gates everything — set it to `"enable"` first, and every control's tooltip works unless that specific control's own `maybeToolTips` is explicitly set to `"disable"`. Leaving the player's setting at its default means no control shows a tooltip no matter what any individual control's own setting is, even though the tooltip attributes (`data-x-tooltip`, etc.) are still present in that control's markup — see `xmediacontrol`'s Rendered DOM section for what the actual tooltip popup looks like and where it renders.

---

## `xmediacontrol` reference — see the sibling skill

For what each `controlType` renders, styling behavior, and how to verify a control actually rendered, see skill `bricksextras-xmediacontrol` — load it whenever placing `xmediacontrol` elements inside a custom layout.

---

## Player-level settings worth knowing

- **Source:** `sourceType` (`media` library picker vs `url`), then `media` (video field) or `src` (text URL). `multipleSources`/`srcRepeater` for multi-format fallback. `playlistMode` switches the player to pull from sibling playlist elements instead of a single source.
- **Behavior:** `loop`, `muted`, `autoplay` (only available when `muted: true` — browser autoplay policy), `crossorigin`, `providerControls` (falls back to the provider's native controls, disables clip start/end support).
- **Poster:** `image`, `aspectRatio` (e.g. `16/9`), `objectFit`.
- **Style groups apply regardless of `layoutType`** — `styleGeneral`/`styleControls` (icon sizes, colors, slider track styling, menu styling, etc.) still theme the custom `xmediacontrol` elements even in custom mode, since those controls share the same underlying CSS custom properties as the built-in UI.

---

## Chapters: `hasLoop` lives on the player itself, but only drives chapters — not multiple player instances

The player has its own `hasLoop` + `query` (group "Chapters"). This is a **narrow, single-purpose** query loop: it only ever generates that one player's chapter markers. It does **not** repeat/loop the player element itself.

Two mutually exclusive ways to populate chapters, gated by `hasLoop`:

- **`hasLoop` off (default):** the manual `chapters` repeater — `text`/`startTime`/`endTime` per row, all plain values (`hasDynamicData: false`, no dynamic tags accepted here).
- **`hasLoop` on:** the manual repeater is replaced by three single-value fields evaluated once per query result — `chapterText`, `chapterStart`, `chapterEnd` — each a real dynamic-tag field (e.g. `{post_title}`, `{acf_chapter_start}`). The manual `chapters` repeater control disappears entirely once `hasLoop` is on; don't expect both to combine.

`chapterStart`/`chapterEnd` accept `H:M:S`, `M:S`, or a plain integer number of seconds — a dynamic tag resolving to any of those formats works.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xmediaplayer.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "hasLoop": true,
  "query": { "objectType": "post", "post_type": ["chapter"], "posts_per_page": -1 },
  "chapterText": "{post_title}",
  "chapterStart": "{acf_chapter_start}",
  "chapterEnd": "{acf_chapter_end}"
}
```

**Don't confuse this with populating multiple player instances from a query** (e.g. a video block per post in an archive) — that's a completely different, unrelated need. This `hasLoop` control never repeats the `xmediaplayer` element itself; setting it for that purpose does nothing except try (and likely fail silently, since `chapterText`/etc. are probably unset in that scenario) to build a chapter list. To actually repeat the player across multiple posts, put `hasLoop`/`query` on a plain wrapping `block` instead, with `xmediaplayer` nested inside it — the same pattern used for `xproslider` slides.

Applies identically to `xmediaplayeraudio` (shared trait, same fields, same gotcha).

**Feeding chapters from a repeater-style custom field** (e.g. a per-track chapters list on a playlist CPT, via ACF, Meta Box, JetEngine, or similar): set `query.objectType` to that field's own registered provider loop type rather than a generic `post`/`array` type — check what's actually available in the current environment (a query-loop-type listing ability, the field provider's own docs, or the live builder's query-type picker) rather than assuming a type name. Once looping, each sub-field's dynamic tag follows that provider's own naming convention for repeater sub-fields — this varies by provider and is easy to guess wrong. **Verify the exact tag against the current site** (a dynamic-data preview/tag-listing tool if the environment has one, or a real render check) before relying on it, rather than assuming a pattern from another provider or another field.

### Populating chapter/caption data is not the same as giving visitors a way to use it

Adding `hasLoop`/`chapterText` etc. (or a manual `chapters` repeater, or `textTracks` for captions) only prepares the *data*. None of it is reachable by a visitor unless a matching control is actually present among the player's rendered controls — **true whether the layout is built-in (`controlsTop`/`controls`/`smallControls`/etc. repeater items) or custom (`xmediacontrol` elements).** This isn't a custom-layout-specific rule; it applies to whichever mechanism is actually populating controls:

- **Chapters need a `"chapters"` control** (the chapters menu — lets a visitor browse and jump to any chapter) to be genuinely usable — a `{"control": "chapters"}` repeater item in built-in mode, or an `xmediacontrol` with `controlType: "chapters"` in custom mode. `"chapter-title"` is a **passive display only** — it shows the current chapter's title as playback progresses, but gives no way to open a list or jump to a different chapter. Adding only `chapter-title` (easy to do by mistake, since it's the more obviously chapters-related name) leaves the chapters data populated but effectively inaccessible.
- **Captions need a `"captions"` control** (the caption toggle) for `textTracks` data to be turned on/off by a visitor at all — same built-in-repeater-item vs. custom-`xmediacontrol` choice as above.

If chapters or captions data is being wired up (via `hasLoop` or the manual/repeater fields), always add the corresponding control — built-in repeater item or `xmediacontrol` — to whatever layout mode is in use. Don't stop at populating the data and assume a control exists by default; nothing is auto-added.

---

## Build workflow

1. **Confirm plugin active, pull live schema** for both `xmediaplayer` and `xmediacontrol`.
2. **Build the parent `xmediaplayer` (`layoutType: custom`, source settings) with its full layout nested inline** — a `block` (or `div`) directly under the player, with as much further `block`/`div` nesting as the design needs, and `xmediacontrol` elements (with the correct `controlType`) wherever a piece of UI belongs, all in one `add-element`/`set-page-elements` call.
3. **Actually look at the rendered page, not just `get-page-elements`/`get-page-structure`.** Those only prove the tree exists with the right `controlType` values; they don't prove the controls are positioned correctly or laid out horizontally rather than stacked. A settings-only check will not catch wrong row order or missing `_display: flex` — both look completely fine in the structural JSON and only show up as visibly broken layout in the browser.
