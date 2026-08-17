---
name: xpanoramascene
description: "Use when building or debugging a scene inside a Panorama Viewer (xpanoramaviewer) from BricksExtras — one xpanoramascene per 360° image, with manual or query-loop-populated hotspots. Covers the two hotspot population modes (manual repeater vs query loop directly on the scene — a deliberate exception to the usual wrap-in-a-block rule), the hidden-DOM hotspot rendering pattern, scene-to-scene linking, and the double-loop pattern for a fully dynamic multi-scene tour. Always load alongside bricksextras-xpanoramaviewer."
---

**Requires:** BricksExtras 1.7.3+ with xpanoramaviewer and xpanoramascene elements enabled

# BricksExtras: Panorama Scene (xpanoramascene)

Shipped by the **BricksExtras** plugin, nestable child of `xpanoramaviewer` (see skill `bricksextras-xpanoramaviewer` — always load both together). One instance = one 360° image and that image's hotspots. Requires a `panoramaImage`; without one it renders a "No panorama image selected" placeholder on the frontend instead of a panorama.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xpanoramascene.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xpanoramascene` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Hotspots render as hidden DOM data, not visible markup — same convention as elsewhere in this plugin

Hotspots are output as `<div class="x-hotspot-data" style="display:none;">` containing one `<div class="x-hotspot-item" data-pitch="..." data-yaw="..." data-type="...">` per hotspot. **This is the same "structured config as inert DOM, read and turned into a real widget by JS at runtime" pattern already seen in `xmediaplaylist` (empty buttons with `data-x-*` attributes) and `xpagetourstep` (content inside an inert `<template>`).** Don't expect visible hotspot pins from a raw `get-page-elements`/HTML read — verify by triggering a live page load and inspecting Pannellum's own generated `.pnlm-hotspot-icon` elements in the DOM. **For the actual hydrated hotspot markup (what to target for custom CSS), see the "Rendered DOM" section in `bricksextras-xpanoramaviewer`** — Pannellum reads this hidden data and rebuilds it as a positioned `.pnlm-hotspot-base` button elsewhere in the tree, not in place.

---

## Two hotspot population modes

### 1. Manual — `hasLoop` off, `hotSpots` repeater

Each row: `pitch`/`yaw` (position in degrees), `icon`, `text` (tooltip content), `buttonText`, `hotspotName` (unique id, referenceable from tooltip content via `#hotspot-name`), `hotspotAriaLabel`, `type` (`info` default / `scene` / `link`), plus `link` (only when `type: link`) or `sceneId` (only when `type: scene` — the *target* scene to jump to).

### 2. Query loop — `hasLoop`/`query` directly on the scene element itself (an exception to the usual rule)

**Unlike most repeating elements in this plugin, the query loop for hotspots goes directly on `xpanoramascene` — not on a wrapping `block`.** This is a deliberate exception to the general "wrap the repeating element in a `block` and put `hasLoop`/`query` there" pattern (see `bricksextras-start-here` / `xmediaplaylist`): here, no nested element tree duplicates at all. `hasLoop: true` + `query: {...}` on the scene makes Bricks' native query loop mechanism re-evaluate the scene's own singular hotspot fields once per query row via a PHP render callback, emitting one `x-hotspot-item` per row — not one nested Bricks element per row.

Singular fields used in this mode (parallel names to the manual repeater's row fields, each a plain setting on the scene rather than a repeater row): `hotspotPitch`, `hotspotYaw`, `icon`, `hotspotText`, `buttonText`, `hotspotType`, `hotspotLink`, `hotspotSceneId`, `hotspotName`, `hotspotAriaLabel`. These are plain text fields rather than the richer control types (select, link, etc.) their manual-mode equivalents use — that's deliberate, so each one can be populated with a dynamic tag bound to the loop's current item (e.g. an ACF repeater sub-field). Bricks resolves "current item in this loop" natively, no special scoping syntax needed.

**When `hasLoop` is on, the manual `hotSpots` repeater is inert** (`required: ['hasLoop', '=', '']` on that control) — the two modes are mutually exclusive per scene.

`hotspotType` accepts `info`, `scene`, or `link` (the same three values as manual mode's select); any other value falls back to `info`.

---

## Two-tier config: scene overrides viewer

`pitch`/`yaw`/`hfov`/`autoRotate` exist on both the viewer (defaults for the whole tour) and the scene (this scene's initial view, only applied `if isset && !== ''`). Leave a scene's copy blank to inherit the viewer's default for that scene. Both elements independently emit their own `data-panorama-config` JSON attribute — the merge into one effective config happens client-side in the panorama JS, not in PHP.

---

## Every scene always has an id — `sceneId` only matters when you need to link to it

Pannellum requires every scene to have a stable internal id, so if the `sceneId` setting is left blank, it falls back to the element's own generated identifier automatically — a scene with no `sceneId` set still renders `"sceneId": "<element-id>"` in its `data-panorama-config`. You don't need to set `sceneId` just to make a scene work.

**Set `sceneId` explicitly when a hotspot elsewhere needs to link to this specific scene** (`type: scene` + target `sceneId`) — the auto-generated element-id fallback isn't something you can predict or reference in advance, especially for loop-duplicated scenes where the id is generated fresh per instance. Give the scene a real, memorable `sceneId` (a literal string for manually-built scenes, or a dynamic tag like `{acf_slug}`/`{post_slug}` for loop-duplicated ones) so a hotspot's target `sceneId` can reliably point at it. `sceneTitle` (shown via the viewer's `showSceneTitles` toggle) is independent of `sceneId` — title is display text, id is the linking key.

---

## `panoramaImage` size: use `full`, not a generated thumbnail size

Equirectangular source images are typically 2048px+ wide (the standard is 2:1, e.g. 2048×1024 or larger) — well past WordPress's default `large` size cap (~1024px on the long edge). Setting `panoramaImage.size` to `large` (or `medium`, `thumbnail`, etc.) silently serves the downscaled generated thumbnail instead of the original upload: the panorama still renders with no error, but visibly blurred/lower-resolution once Pannellum projects it onto the sphere and the visitor zooms or looks closely. There's no validation warning for this — it just looks soft.

**Always set `panoramaImage: { "useDynamicData": "<tag>", "size": "full" }`** (or the equivalent literal-image-picker `full` option in manual mode) unless a project specifically has a reason to trade resolution for a smaller file size.

---

## Fully dynamic tour: double loop

A `panorama-scene` CPT with two posts ("Living Room", "Bedroom"), each with its own ACF image field and its own ACF repeater of hotspots — including a scene-linking hotspot on each, so clicking one navigates to the other scene.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xpanoramascene.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xpanoramaviewer",
  "children": [
    {
      "name": "block",
      "settings": {
        "hasLoop": true,
        "query": { "objectType": "post", "post_type": ["panorama-scene"] }
      },
      "children": [
        {
          "name": "xpanoramascene",
          "settings": {
            "panoramaImage": { "useDynamicData": "{acf_scene_image}", "size": "full" },
            "sceneId": "{post_slug}",
            "hasLoop": true,
            "query": { "objectType": "acf_scene_popovers" },
            "hotspotPitch": "{acf_scene_popovers_popover_pitch}",
            "hotspotYaw": "{acf_scene_popovers_popover_yaw}",
            "buttonText": "{acf_scene_popovers_popover_label}",
            "hotspotType": "{acf_scene_popovers_type}",
            "hotspotSceneId": "{acf_scene_popovers_scene_to_link_to}",
            "hotspotLink": "{acf_scene_popovers_link_url}"
          }
        }
      ]
    }
  ]
}
```

- **Outer loop**: a plain `block` wraps `xpanoramascene`, `hasLoop`/`query` targets the CPT (`post_type: ["panorama-scene"]`) — the standard wrap-in-a-block pattern, duplicating the whole scene once per post. 2 posts → 2 duplicated `xpanoramascene` elements, each with its own image correctly resolved from its own post.
- **Inner loop**: `hasLoop: true` + `query: { "objectType": "<acf repeater field name>" }` directly on the scene — the ACF repeater field's own name (prefixed `acf_`, e.g. `scene_popovers` → `acf_scene_popovers`) is passed as the `query.objectType` directly, not a generic loop-type keyword. 2 repeater rows on "Living Room" → 2 correctly-positioned hotspots ("Nice table" pitch 20/yaw 50, "Desk" pitch 50/yaw 30), same for "Bedroom" (2 different hotspots, correctly *not* mixed with Living Room's).
- **Dynamic tags for ACF fields use the `acf_` prefix**: a plain field named `scene_image` becomes `{acf_scene_image}`; a repeater named `scene_popovers` with sub-fields `popover_pitch`/`popover_yaw`/`popover_label` becomes `{acf_scene_popovers_popover_pitch}` etc. — pattern is `{acf_<repeater_field_name>_<sub_field_name>}`.
- **No special scoping syntax needed for "current post in the outer loop"** — Bricks' query loop mechanism handles this natively; the inner loop on each duplicated scene correctly resolved to that scene's own post without any extra binding.
- **Scene-linking hotspot rows need `type` set to `scene`** and `scene_to_link_to` set to the target scene's `sceneId` value — in this example that means the *other* room's post slug (e.g. the "Living Room" post's popover row targeting `bedroom`, matching the `sceneId: "{post_slug}"` set on the Bedroom scene). Point `scene_to_link_to` at a `sceneId` that doesn't exist and the hotspot silently renders as a dead `info`-style pin with no navigation and no error.

Before building this on a real project, get the actual field names from the site's real custom fields/dynamic-data tags rather than guessing — the `acf_` prefix convention (if ACF is the source on this site) is not universal, and field/sub-field names, and the exact `type`/`scene_to_link_to`/`link_url` sub-field names shown above, are project-specific and just illustrative.

---

## Never do

- Do not wrap `xpanoramascene` in a `block` to loop its *hotspots* — that wrap-in-a-block pattern is for duplicating the whole *scene* (multi-scene tours). Hotspot-level looping goes directly on the scene element itself.
- Do not set both `hotSpots` (manual repeater) and `hasLoop: true` expecting them to combine — `hasLoop` makes the manual repeater inert; the two hotspot modes are mutually exclusive.
- Do not store anything other than `info`/`scene`/`link` in whatever field backs `hotspotType` in loop mode — same three values as the manual mode's select, just as a plain text value so a dynamic tag can populate it. Any other value falls back to `info`.
- Do not expect hotspot pins to be visible in raw HTML/`get-page-elements` — they're rendered as hidden data divs, turned into real Pannellum hotspot elements by JS at runtime. Verify via a live page load.
- Do not set `sceneId` on every scene "just in case" — it's optional and falls back to the element's own id. Only set it explicitly on scenes that need to be linkable-to from a hotspot elsewhere.
- Do not leave `panoramaImage.size` at a generated thumbnail size (`large`, `medium`, etc.) — equirectangular images are typically wider than WordPress's default thumbnail caps, so anything but `full` silently serves a blurred, downscaled version with no error.
