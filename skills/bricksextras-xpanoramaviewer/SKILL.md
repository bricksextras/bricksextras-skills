---
name: xpanoramaviewer
description: "Use when building or debugging a 360°/equirectangular panorama viewer or multi-scene virtual tour with the Panorama Viewer (xpanoramaviewer) and Panorama Scene (xpanoramascene) elements from BricksExtras, built on Pannellum.js. Covers the viewer/scene two-tier structure, the click-to-load facade pattern, and why nothing here works without at least one nested scene with an image. Load bricksextras-xpanoramascene alongside this one — always used together."
---

**Requires:** BricksExtras 1.7.3+ with xpanoramaviewer and xpanoramascene elements enabled

# BricksExtras: Panorama Viewer (xpanoramaviewer)

Shipped by the **BricksExtras** plugin. Wraps **Pannellum.js** to display 360°/equirectangular panorama images, optionally as a multi-scene virtual tour with clickable hotspots linking scenes together. `xpanoramaviewer` is the nestable parent — one instance = one panorama widget/tour, holding shared/default viewer config. Its children are `xpanoramascene` elements (see skill `bricksextras-xpanoramascene`) — one per panorama image/scene. **Always load both skills together**, since a viewer is never used without at least one scene, and a scene never exists outside a viewer.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xpanoramaviewer.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xpanoramaviewer` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session — and do the same for `xpanoramascene`'s own schema before building its children.

---

## Structure — at least one scene is always required

```
xpanoramaviewer (shared viewer config: aspect ratio, interaction, loading, scroll-instructions, scene-title, hotspot-icon/tooltip styling)
└── xpanoramascene (one image + this scene's hotspots)
```

There is no "viewer with no scenes" state — `get_nestable_item()` seeds a single default scene (with a placeholder Unsplash image) when added via the builder UI (not auto-applied via MCP, per the usual defaults rule — build it explicitly). A scene with no `panoramaImage` set renders a "No panorama image selected" placeholder on the frontend instead of a panorama.

For a single static image, one scene, no `sceneId` needed. For a multi-scene tour, nest multiple scenes (or duplicate one via a query loop wrapper — see `bricksextras-xpanoramascene`), each with a unique `sceneId`, and use `type: scene` hotspots to link between them.

---

## Rendered DOM: the pre-JS markup only carries config — Pannellum builds the entire visible viewer itself

The server-rendered HTML never contains the actual panorama or hotspots — just a `data-panorama-config` JSON blob and an empty mount point:

```html
<div data-panorama-config="{...json...}" data-loading-strategy="lazy" data-show-scene-titles="false">
  <div class="x-panorama-viewer"></div>              <!-- Pannellum mounts its canvas here -->
  <div class="x-panorama-viewer_inner">{children}</div>  <!-- scenes (still hidden data, see xpanoramascene) + optional facade content -->
</div>
```

`.x-panorama-viewer_inner` is a flex container — the viewer's own `_justifyContent`/`_alignItems`/`_gap`/`_flexDirection` controls are retargeted to it (not the root), so they're the right controls for centering/arranging whatever's nested there (scenes, and any click-to-load facade content — see below).

**For custom styling, the DOM that actually matters is what Pannellum generates at runtime inside `.x-panorama-viewer`** — this is real class names from the Pannellum library itself, not something BricksExtras' own controls fully expose:

```html
<div class="x-panorama-viewer pnlm-container" role="region" aria-label="Panorama viewer">
  <div class="pnlm-ui pnlm-grab">
    <div class="pnlm-panorama-info">
      <div class="pnlm-title-box">Living Room</div>       <!-- sceneTitle, shown when showSceneTitles -->
      <div class="pnlm-author-box"></div>
    </div>
    <div class="pnlm-controls-container">
      <div class="pnlm-zoom-controls pnlm-controls">
        <div class="pnlm-zoom-in pnlm-sprite pnlm-control" role="button" aria-label="Zoom in"></div>
        <div class="pnlm-zoom-out pnlm-sprite pnlm-control" role="button" aria-label="Zoom out"></div>
      </div>
      <div class="pnlm-fullscreen-toggle-button pnlm-sprite pnlm-controls pnlm-control" role="button" aria-label="Toggle fullscreen"></div>
      <div class="pnlm-back-button pnlm-sprite pnlm-controls pnlm-control x-panorama-back-btn" role="button" aria-label="Go back to previous scene"><svg>...</svg></div>
    </div>
  </div>
  <div class="pnlm-render-container">
    <canvas></canvas>
    <!-- one .pnlm-hotspot-base per hotspot, positioned via inline transform: translate/rotate -->
    <div class="pnlm-hotspot-base pnlm-hotspot pnlm-sprite pnlm-info pnlm-tooltip" style="transform: translate(1029.95px, 354px) translateZ(9999px) rotate(0deg);">
      <div class="pnlm-hotspot-wrapper">
        <button class="pnlm-hotspot-icon" aria-label="A nice sofa" data-hotspot-name="sofa-info" aria-describedby="hotspot-tooltip-{id}" aria-expanded="false">
          <span class="pnlm-hotspot-icon-inner"><i class="fas fa-info-circle" aria-hidden="true"></i></span>
        </button>
        <span class="pnlm-hotspot-tooltip" id="hotspot-tooltip-{id}" role="tooltip" aria-hidden="true">A nice sofa</span>
      </div>
    </div>
  </div>
</div>
```

Notes:

- **The hotspot icon rendered here (`.pnlm-hotspot-icon-inner > i`) is the exact icon markup lifted from the scene's own hidden `.x-hotspot-item > .x-hotspot-icon`** — Pannellum's JS reads the hidden data div (see `bricksextras-xpanoramascene`) and rebuilds it as a real, positioned button. The hidden source markup and the real rendered hotspot are two different elements in two different places in the DOM; don't expect to style one and have it reflect on the other via normal CSS inheritance — style the real one, using the `hotspotIcon*`/`hotspotTooltip*` viewer-level controls (targeting `.pnlm-hotspot-icon`/`.pnlm-hotspot-tooltip`, see below) or custom CSS against those same classes.
- **Every hotspot's on-screen position is a JS-computed inline `transform`, recalculated continuously as the view rotates/zooms** — there's no static `top`/`left`, so don't attempt to override position with CSS positioning properties; `pitch`/`yaw` (set on the scene) are the only real controls over where a hotspot sits.
- `pnlm-title-box`/`pnlm-author-box` are always in the DOM regardless of `showSceneTitles` — the setting only controls their content/visibility state, not whether the elements exist.
- The zoom/fullscreen/back-button controls are all real `<div role="button">` elements (not `<button>`), each independently toggleable via the viewer's `showZoomCtrl`/`showFullscreenCtrl` settings — `display:none` inline when a given control is disabled, rather than being omitted from the DOM entirely.

---

## Click-to-load facade

`loadingStrategy` (viewer-level): `eager` / `lazy` (default) / `click` / `interactions`. With `click`, the panorama image doesn't load until the visitor clicks — nest a CTA (button/text) as a child of the viewer to serve as that click target and prompt; it renders inside `.x-panorama-viewer_inner`, which is why that wrapper is a flex container in the first place (for centering the CTA over the placeholder). `hideNestedContent` (builder-only cosmetic toggle) + `hideNestedContentOverlayColor` (tints `.x-panorama-viewer_inner` while waiting for the click) support this pattern. `placeholderImage` shows a small image before the real panorama loads, independent of the loading-strategy facade.

---

## Viewer-level settings worth knowing

- **Dimensions**: `aspectRatio` (plain number-ish text, not a units control — e.g. `"2/1"`, the standard equirectangular ratio; refresh the canvas after changing if the image looks stretched).
- **Initial view** (defaults for scenes that don't override them — see `bricksextras-xpanoramascene`'s two-tier config note): `pitch` (vertical °), `yaw` (horizontal °), `hfov` (field of view, 10–120), `autoRotate` (°/sec, `0` disables).
- **Interaction**: `showZoomCtrl`/`showFullscreenCtrl` (+ `fullscreenFallback` for browsers without native fullscreen)/`mouseZoom` (+ `mouseZoomSensitivity`)/`draggable`/`disableKeyboardCtrl` — all `enable`/`disable` selects, all default enabled.
- **Movement feel**: `friction` (0.01–1, higher = stops faster).
- **Accessibility**: `ariaViewer`/`ariaZoomIn`/`ariaZoomOut`/`ariaFullscreen`/`ariaBackButton` — text overrides for Pannellum's built-in ARIA labels.
- **Scroll/gesture hint overlay**: `showScrollInstructions` (default enabled unless explicitly `disable`) + `scrollInstructionsDesktopText`/`scrollInstructionsMobileText` + full style group targeting `.pnlm-scroll-instructions` (Pannellum's native "ctrl+scroll to zoom" hint).
- **Scene titles**: `showSceneTitles` (`'true'`/`'false'` string select, not boolean, default false) + style group targeting `.pnlm-title-box` — displays each scene's own `sceneTitle`.
- **Loading spinner**: `spinnerColor`/`spinnerTrackColor`/`spinnerStrokeWidth`/`spinnerSize`, all via CSS custom properties, styling a `.x-panorama-viewer::before` pseudo-element.
- **Hotspot icon/tooltip styling** (`hotspotIcon*`/`hotspotPulse*`/`hotspotTooltip*` groups): these live on the *viewer*, not the scene, even though hotspots themselves are configured per-scene — one shared visual style across every hotspot in the tour. Targets `.pnlm-hotspot-icon`/`.pnlm-hotspot-icon-inner`/`.pnlm-hotspot-tooltip`. **Don't set `hotspotIconWidth`/`hotspotIconHeight` to a fixed size when hotspots carry real button text** (via `buttonText`/manual `hotSpots` rows) — a fixed width/height clips or wraps the label instead of letting the button size to its content. Fixed dimensions only make sense for icon-only hotspots with no visible text.
- **Debug**: `hotSpotDebug` (checkbox, builder-preview-only) — shows live pitch/yaw values, useful for finding exact hotspot coordinates without trial-and-error guessing on the frontend.

---

## Never do

- Do not expect a viewer to render anything meaningful with zero nested scenes, or a scene with no `panoramaImage` — both produce a placeholder, not a panorama.
- Do not put hotspot icon/tooltip styling on the scene — it's viewer-level (shared across all scenes in the tour), even though hotspot *data* is scene-level.
- Do not assume `showSceneTitles` is a checkbox — it's a select with string values `'true'`/`'false'`.
- Do not skip the click-to-load facade content when `loadingStrategy: click` is set — without a nested CTA, there's nothing for the visitor to click.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real Pannellum instance directly off the viewer DOM node — not from an `Instances` registry, unlike this plugin's other JS-library elements. Find `.x-panorama-viewer` inside the `xpanoramaviewer` element, then read `.pannellumInstance` off it: `document.querySelector('.brxe-xpanoramaviewer .x-panorama-viewer').pannellumInstance`. Do this in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), then drive it via Pannellum's own API (`viewer.loadScene()`, `viewer.lookAt()`, etc.). It's set synchronously on init, no artificial delay to wait out.
