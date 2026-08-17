---
name: xmediaplayeraudio
description: "Use when building, debugging, or styling Media Player (Audio) (xmediaplayeraudio) from BricksExtras: an audio-focused sibling of xmediaplayer built on the same VidStack foundation, with a single artist/title field pair, optional album-art poster, and a download-links repeater. Covers what's shared with xmediaplayer versus what's genuinely audio-specific (single built-in layout instead of two, downloads, no video-only playback chrome). Load bricksextras-xmediaplayer alongside this skill for the shared mechanics (custom layout structure, chapters, playlist mode), and bricksextras-xmediacontrol for what each controlType renders."
---

**Requires:** BricksExtras 1.7.3+ with xmediaplayeraudio and xmediacontrol elements enabled

# BricksExtras: Media Player (Audio) (xmediaplayeraudio)

Shipped by the **BricksExtras** plugin. A nestable audio player — a sibling of `xmediaplayer`, built on the same underlying VidStack player and sharing the large majority of its controls (source handling, chapters, playlist mode, keyboard shortcuts, captions/text tracks, tooltips, and nearly all of the `styleGeneral`/`styleControls` theming groups verbatim). **This skill only documents where audio genuinely diverges from video** — for everything else, `bricksextras-xmediaplayer` applies unchanged. Load it alongside this skill. Load `bricksextras-xmediacontrol` too whenever a custom layout is in play.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xmediaplayeraudio.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xmediaplayeraudio` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## What's identical to `xmediaplayer`

- **Source plumbing:** `sourceType` (`media`/`url`), `src`, `srcMediaType`, `clipStartTime`/`clipEndTime`, `streamType`, `multipleSources`.
- **Chapters:** `hasLoop` + `query` (group "Chapters") drives the manual `chapters` repeater (`text`/`startTime`/`endTime`, plain values, no dynamic tags) vs. the loop-driven `chapterText`/`chapterStart`/`chapterEnd` (dynamic-tag fields, one evaluated per query result) — mutually exclusive, gated by `hasLoop`. `chapterStart`/`chapterEnd` accept `H:M:S`, `M:S`, or plain seconds. **This `hasLoop` only ever builds that one player's chapter list — it does not repeat/loop the player element itself.** Populating multiple player instances from a query is a separate, unrelated need: put `hasLoop`/`query` on a plain wrapping `block` instead, with `xmediaplayeraudio` nested inside it — same pattern as `xproslider` slides. Setting `hasLoop` on the player directly for that purpose does nothing except try to build a chapter list. Identical mechanics to `xmediaplayer` (shared trait) — see that skill's "Chapters" section for a worked JSON example.
  **Populating chapter data alone doesn't make it reachable.** A visitor can only browse/jump chapters if a `"chapters"` control is actually present — a `{"control": "chapters"}` repeater item in the built-in `controlsTopAudio`/`controlsCenterAudio`/`controlsBottomAudio` layout, or an `xmediacontrol` with `controlType: "chapters"` in a custom layout. `"chapter-title"` is a passive current-chapter display only, not a way to navigate chapters — don't mistake it for the real control.
- **Playlist mode:** `playlistMode`, `whichPlaylist` (`section`/`selector`/`component`), `playlistSelector`, `componentScope` (string `"true"`/`"false"`, not boolean — same gotcha as elsewhere in BricksExtras), `playListNext`, `playListLoop`. Pairs with sibling `xmediaplaylist` elements exactly as documented in `bricksextras-xmediaplaylist`.
- **Keyboard shortcuts group:** all `*Shortcut` fields, identical.
- **Text tracks / captions:** `textTracks` repeater, `thumbnailFile`, `detectLanguage`, `autoEnableCaptions`. Same access rule as chapters: `textTracks` data only becomes visible/toggleable to a visitor if a `"captions"` control is present (built-in repeater item or `xmediacontrol`) — populating `textTracks` alone leaves it inert with no on-screen toggle.
- **Tooltips group:** `maybeToolTips`, `tooltipBg`/`tooltipBorder`/`tooltipTypography`/`tooltipPadding`/offsets/`defaultTooltipPlacement`.
- **Styling:** `styleGeneral` and `styleControls` groups (icon sizing, slider track/thumb styling, menu theming, chapter menu item states, time/title/artist typography) are present with the same keys on both elements — these theme the shared `xmediacontrol` UI regardless of which player it's attached to.
- **Custom layout mechanism:** same pattern as `xmediaplayer` — set the layout-type control to `custom`, then nest a `block` (or `div`) directly under the player with `xmediacontrol` elements inside it for each piece of UI. The schema's own separator description for audio's custom mode is near-identical wording to video's: *"Add a block element inside the media player for the layout. Add the 'media controls' element inside block."* `bricksextras-xmediaplayer`'s "DOM order is visual order" and "layout is not automatic" sections apply here too. As with video, nothing gets added automatically — the `block` and its `xmediacontrol` children must be built explicitly.
- **`_background`/`_border`** on the player itself.

---

## What's genuinely audio-specific

### 1. Media field and metadata

- Source media control is `audio` (control type `audio`), not `media` (type `video`). Multi-source repeater is `srcRepeaterAudio`, not `srcRepeater`.
- **`artist` is a dedicated top-level field**, paired with `title`. `xmediaplayer` only has `title` — no `artist` equivalent. Both feed the shared `titleTypography`/`artistTypography`/`artistPrefixTypography`/`artistSuffixTypography` style controls (those style keys exist on both elements, but only audio has a field that populates `artist`).
- **Entering `title`/`artist` on the player is not enough to display them, in either layout mode.** `xmediacontrol`'s `controlType` select has dedicated `title` and `artist` options, alongside `time`, `time-slider`, `play`, etc. In a `layoutTypeAudio: custom` build, an `xmediacontrol` with `controlType: "title"`/`"artist"` must be placed somewhere in the layout for that metadata to render. The `default` built-in layout is no different: with no `title`/`artist` entries in `controlsTopAudio`/`controlsCenterAudio`/`controlsBottomAudio`, the player renders only its structural controls (e.g. the time slider) — title/artist data alone produces no visible output. Add `{"control": "title"}`/`{"control": "artist"}` to one of the control-zone repeaters for the default layout, or an `xmediacontrol` for a custom layout.

### 1a. Waveform — an option on the time-slider control, not the player

There is no player-level "waveform" setting. It lives on `xmediacontrol` itself, scoped to `controlType: "time-slider"`:

- `enableWaveform` (checkbox) — `required: ["controlType", "=", ["time-slider"]]`, i.e. only appears/applies when that control is a time-slider.
- Sub-fields once enabled, all under `group: "waveform"`: `waveformHeight` (px, no unit suffix — plain number, placeholder `26`), `waveformBarWidth` (placeholder `2`), `waveformBarGap` (placeholder `1`), `waveformColor`, `waveformPlayedColor` (both `color` controls, unplayed vs. played portion of the bars).
- These waveform sub-fields carry no `required` condition of their own — there's no conditional visibility on time-slider fields generally, waveform or otherwise. They're grouped under `group: "waveform"` for UI organization only; gating is functional (they only have a visible effect once `enableWaveform: true`), not schema-enforced.
- Since `xmediacontrol` is the shared sibling element for both `xmediaplayer` and `xmediaplayeraudio`, this waveform option is technically available on video's custom layouts too — but it's overwhelmingly an audio-player pattern in practice (replacing the plain time-slider track with an audio waveform visualization, as seen in podcast-style players).
- **Only reachable in a custom layout.** The built-in `layoutTypeAudio: default` repeaters (`controlsTopAudio`/`controlsCenterAudio`/`controlsBottomAudio`) reference controls by `{"control": "time-slider"}` name only — they don't expose per-item sub-settings like `enableWaveform`. To get a waveform, build a custom layout with an explicit `xmediacontrol` (`controlType: "time-slider"`, `enableWaveform: true`) rather than trying to configure it through the default layout's repeater.

### 2. Only one built-in layout, not two

This is the biggest structural difference. `xmediaplayer` has **two independent built-in layouts** (`layoutType: type_one` using `smallControlsTop`/`smallcontrolsCenter`/`smallControls`, and `type_two` using `controlsTop`/`controlsCenter`/`controls`) — separate repeaters for a "large" and "small" responsive variant, per the `bricksextras-xmediaplayer` skill.

`xmediaplayeraudio` has **only one built-in layout**, controlled by `layoutTypeAudio` with just two options: `default` and `custom`. There is no type_one/type_two split. The single set of control-zone repeaters is:

- `controlsTopAudio`, `controlsCenterAudio`, `controlsBottomAudio` (group `controlsAudioLayout`)
- Matching gap controls: `audiocontrolGapTop`, `audiocontrolGapCenter`, `audiocontrolGapBottom`
- Full-width time slider toggle: `maybeTimeSliderAudio` (not `maybeTimeSlider`/`maybeTimeSliderSmall`)

**Do not reuse `xmediaplayer`'s `controlsTop`/`smallControlsTop`-style key names on an audio player — they don't exist on this element and will silently do nothing.** Always pull the live schema (or this skill) for the correct `*Audio`-suffixed key names before writing settings.

### 3. Poster / album art

Audio still supports an optional poster image (typically album art): `image`, `altText`, `objectFit`, `imgLoading` (group `poster`), plus a dedicated style sub-group under `styleControls`: `posterImageHeight`, `posterImageWidth`, `posterImageAspectRatio`, `posterImageBorder`.

Contrast with `xmediaplayer`, which has no `posterImage*` style sub-group at all — video's own visible surface uses a single top-level `aspectRatio` text control on the player instead, because the video frame itself is the visual element. On audio, the *poster image* is the only visual surface, so its aspect ratio is controlled separately via `posterImageAspectRatio` rather than a player-level `aspectRatio` field (audio has no top-level `aspectRatio` control at all).

### 4. Downloads — audio-only feature

`downloadAttributes` (repeater, group `downloads`) lets you offer direct download links for the media — described in-schema as *"Optionally add extra download options for the media."* No equivalent exists on `xmediaplayer`.

### 5. Video-only features that do NOT exist on audio

These have no effect on `xmediaplayeraudio` — they don't exist on this element:

- `modalAutoplay`, `playReset` (modal-reopen playback behavior)
- `controlsVideoOpacity`, the entire `bufferingIndicator*` spinner group
- `autoLocalPoster`, `playsinline`, `posterReshow`, `controlsDelay`, `fullscreenFallback` (fullscreen is not meaningful for an audio-only surface)
- `loadingUI`/`loadingUISep`
- Bunny Stream fields: `enableSecurity`, `securityExpiry`, `autoCaptionsBunny`
- `gestures`/`clickToPlay`
- `maybeNotice` and the whole notice-overlay group (`noticeText`, `noticeVisibility`, `noticeTextBg`, etc.)
- `controlsTopVisibility`/`controlsCenterVisibility`/`controlsBottomVisibility` (the "hide until loaded/played" per-zone visibility selects that exist on video's `controlsLarge`/`controlsSmall` groups have no counterpart under `controlsAudioLayout`)

---

## Build workflow

1. **Pull the live schema for `xmediaplayeraudio`** (not `xmediaplayer`'s) before writing settings — the `*Audio`-suffixed key names above are easy to get wrong from memory since the two elements are so similar. Pull `xmediacontrol`'s schema too if building a custom layout.
2. **Decide built-in vs custom** via `layoutTypeAudio` (`default` or `custom`) — remember there's no type_one/type_two choice here, unlike video.
3. For a **default layout**, populate `controlsTopAudio`/`controlsCenterAudio`/`controlsBottomAudio` repeaters (each item `{"control": "..."}`, same control-name vocabulary as `xmediacontrol`'s `controlType` options — confirm exact names against the live `xmediacontrol` schema rather than assuming full parity with video's default sets).
4. For a **custom layout**, nest a `block` under the player with `xmediacontrol` elements inside, exactly as described in `bricksextras-xmediaplayer`'s custom-layout section — nothing gets added automatically, the full layout must be built explicitly.
5. **Set `audio` (not `media`) for the source**, and use `artist`/`title` for the metadata fields if the built-in layout's `title` control is in use.
6. **Actually look at the rendered page**, not just the saved element tree — same caution as `xmediaplayer`: structurally-valid settings JSON does not prove the layout renders correctly.
