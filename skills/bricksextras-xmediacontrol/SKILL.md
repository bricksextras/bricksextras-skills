---
name: xmediacontrol
description: "Use when building or debugging Media Control (xmediacontrol) from BricksExtras: the individual play/mute/time-slider/settings/etc. leaf element nested inside a Media Player's custom layout. Covers the controlType reference table and its styling behavior. Always load alongside bricksextras-xmediaplayer, which covers the parent player and the custom-layout structure this element gets placed into."
---

**Requires:** BricksExtras 1.7.3+ with xmediacontrol element enabled

# BricksExtras: Media Control (xmediacontrol)

Shipped by the **BricksExtras** plugin, as a sibling to `xmediaplayer` (the player itself — see skill `xmediaplayer`). `xmediacontrol` is a non-nestable leaf element that renders exactly one piece of player UI (play button, time slider, volume, settings menu, etc.). It only does anything when placed inside an `xmediaplayer` that has `layoutType: custom` — see `xmediaplayer`'s skill for that structural requirement, the required Wrapper/row nesting, and the DOM-order/flex-layout gotchas that apply to where these elements sit. This skill covers the control element itself: what each `controlType` renders and how it styles.

Not nestable (`"nestable": false` in the live schema).

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xmediacontrol.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xmediacontrol` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## `controlType` is the only field that determines what renders

Everything else on the element is conditional on it via `required` rules:

| `controlType` | Renders | Notes |
|---|---|---|
| `play` / `play-large` | Play/pause button | Large variant typically used as a center overlay |
| `mute` | Volume/mute button | `volumeSlider` sub-setting adds an attached slider (`focus`/`visible`/`disable`) |
| `time` | Current/duration time display | `currentTime` can show remaining time instead |
| `time-slider` | Scrub bar | Has its own extensive style group (track, thumb, value preview, chapter title) |
| `seek-backward` / `seek-forward` | Jump the playhead back/forward by a fixed increment | Distinct from `previous`/`next`, which are playlist navigation, not in-track seeking |
| `captions` | Caption toggle | On/off labels configurable |
| `chapters` | Chapters menu | Own menu placement/offset controls |
| `chapter-title` | Current chapter title display | — |
| `settings` | Settings menu (speed, quality, captions, loop, accessibility) | Each sub-label independently configurable |
| `fullscreen` | Fullscreen toggle | — |
| `pip` | Picture-in-picture toggle | — |
| `download` | Download button | `downloadSource`: media source URL or a named download option |
| `live-button` | Live/edge indicator | For live streams |
| `next` / `previous` | Playlist navigation | Only meaningful with `playlistMode` enabled on the player |
| `artist` | Artist/metadata text | Prefix/suffix text supported |
| `title` | Media title text | — |
| `custom-text` | Arbitrary static text | `controlText` field |
| `poster-image` | Poster image | Own object-fit/aspect-ratio controls |
| `spacer` | Flexible gap | For pushing groups apart in a flex control row |

**The `*` on some `controlType` labels (`Caption toggle *`, `Download *`, `Fullscreen *`, `PIP *`) marks controls that render conditionally based on the currently-loaded media** — the element only appears in the DOM if the active media actually supports/has that feature. `captions` won't render any content unless the loaded media has caption tracks; `download`/`fullscreen`/`pip` likewise depend on what the current source actually supports. This matters directly for custom layout building: if the layout's structure assumes these controls are always present (e.g. spacing/alignment that depends on a fixed set of children), switching to media that lacks that feature can change the rendered structure. `play`/`play-large`, `mute`, `time`, `time-slider` are stable — always present regardless of what's loaded.

**This is especially relevant with `playlistMode`** (see `xmediaplayer`) — different tracks/videos in the same playlist can have different data (some with captions, some without; some downloadable, some not), so a control that renders fine for one playlist item can disappear entirely once the player switches to another.

**`controlText: "Text here"` appears by default on every `xmediacontrol` instance** regardless of `controlType`, including ones where it's irrelevant (e.g. `mute`, `time-slider`). This is a harmless default field, not something to intentionally set per control type — ignore it unless the type is actually `custom-text`.

---

## Styling applies regardless of the parent player's `layoutType`

The player's `styleGeneral`/`styleControls` groups (icon sizes, colors, slider track styling, menu styling, etc.) theme `xmediacontrol` elements even in custom mode — those controls share the same underlying CSS custom properties (e.g. `--media-brand`) as the built-in UI. Setting the player's brand/primary color still reaches custom-layout controls; there's no separate per-control color system to configure.

---

## Rendered DOM (for custom CSS/targeting)

Icon SVGs elided. Only the control elements themselves are `xmediacontrol` output — the row `block`/`div` each one is placed inside (see `xmediaplayer`'s reference pattern) is a normal Bricks element, not something `xmediacontrol` renders itself:

```html
<media-play-button class="brxe-xmediacontrol xmp-button xmp-play-button" data-x-control-type="play" data-x-tooltip="Play" data-x-pressed-tooltip="Pause" data-paused>...</media-play-button>

<div class="brxe-xmediacontrol" data-x-control-type="time-slider">
  <media-time-slider class="xmp-time-slider xmp-slider">...</media-time-slider>
</div>

<div class="brxe-xmediacontrol xmp-time-group" data-x-control-type="time">
  <media-time class="xmp-time" type="current">0:00</media-time>
  <div class="xmp-time-divider">/</div>
  <media-time class="xmp-time" type="duration">0:00</media-time>
</div>
```

Notes on this structure:

- **`data-x-control-type="{controlType}"` is present on every `xmediacontrol` root** — a reliable, always-present selector hook for targeting a specific control type without relying on its generated class name.
- Same real custom elements as the built-in UI (`<media-play-button>`, `<media-time-slider>`, `<media-time>`).
- **`data-x-tooltip`/`data-x-pressed-tooltip` are conditional on `maybeToolTips`, not always present.** They're added whenever `maybeToolTips` isn't explicitly set to `"disable"` (tooltips are on by default). Setting `maybeToolTips: "disable"` removes these attributes, and the tooltip behavior/markup that depends on them, for that control.
- **A control's own `maybeToolTips` is not the only switch — the parent `xmediaplayer` has its own separate `maybeToolTips` setting that gates every control's tooltip, and its default is the opposite of this element's.** The player's tooltip setting defaults to **disabled**; the control's own defaults to **enabled**. This means tooltips render nowhere by default unless the player's `maybeToolTips` is explicitly set to `"enable"` — a control's own tooltip attributes can be present in the DOM (`data-x-tooltip="Play"`, etc.) while nothing actually shows on hover, because the player-level switch is off.
- **The actual tooltip popup is powered by Tippy.js and portalled to the end of `<body>`, not nested near the control.** On hover/focus, the control gets `data-hocus`/`data-hover` attributes and `aria-describedby` pointing at a `#tippy-{n}` id; that id belongs to a `[data-tippy-root]` element appended elsewhere in the DOM:
  ```html
  <div data-tippy-root id="tippy-1" style="...">
    <div class="tippy-box" data-state="visible" role="tooltip" data-placement="top">
      <div class="tippy-content">Play</div>
    </div>
  </div>
  ```
  Style the popup itself via `.tippy-box`/`.tippy-content`, not via anything nested inside the control — there's nothing there to style since the popup doesn't live in that part of the tree.

---

## Never do

- Do not set `controlText` on control types where it's irrelevant just because it appears by default — it's a harmless leftover field, not a signal to configure.
