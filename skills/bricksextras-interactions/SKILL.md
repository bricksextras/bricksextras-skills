---
name: interactions
description: "Use when wiring a Bricks Interaction (_interactions) to a BricksExtras-specific event (e.g. 'show a toast when a favorite is added', 'run something when a slider moves', 'react to a modal opening') or to a BricksExtras action (e.g. showing/hiding a toast, alert, modal, offcanvas via an interaction rather than the element's own built-in trigger). Covers how BricksExtras bridges into Bricks' native _interactions system, the full custom trigger catalog, the bricksextras.* JS action namespace, and the placeholder-templated trigger gotcha."
---

**Requires:** BricksExtras 1.7.3+, Bricks 2.4+

**Check for a general/core Bricks interactions skill and load it first, if one exists in this environment.** This skill only covers the BricksExtras-specific half (extra trigger names, `bricksextras.*` actions) — it does not document the native `_interactions` schema itself. Don't assume this skill is a complete reference for building an interaction from scratch; it assumes the core mechanics are already known or covered elsewhere.

# BricksExtras: Interactions bridge (custom triggers + `bricksextras.*` actions)

BricksExtras does **not** register a separate interactions system — everything here lives inside Bricks' own native `_interactions` array on an element (same shape as any core Bricks interaction: `{id, trigger, action, target, ...}`, read/written via whatever interactions ability the current MCP connection exposes, e.g. `bricks/get-element-interactions`/`bricks/update-element-interactions` on the native Bricks MCP). BricksExtras extends this in two independent ways: it supplies extra **trigger** names Bricks core doesn't know about, and it supplies `bricksextras.*` JS functions to call as the **action**, via Bricks' existing native `action: "javascript"` type. Nothing new is added to the Bricks schema itself — both halves work entirely through existing native fields.

## How a custom trigger actually fires — BricksExtras' own listener registry, not a Bricks hook

Bricks' own runtime only knows how to bind its own built-in trigger types (`click`, `enterView`, `formSubmit`, the Woo events, etc. — the full core list). For anything else, BricksExtras' own `interactions.js` scans every element's `_interactions` rows on page load (and again after Bricks AJAX events — see below) and, for any `trigger` value it recognizes via its own hardcoded catalog, calls `element.addEventListener(...)` itself, then calls Bricks' own `bricksInteractionCallbackExecution(element, interaction)` to actually run the configured action — so the action side stays 100% native Bricks regardless of which half (core or BricksExtras) supplied the trigger.

**Practical effect: don't assume a trigger name is invalid just because it's outside Bricks' documented trigger list** — the write ability doesn't reject non-core trigger strings (there's no way to enumerate BricksExtras' triggers from the API surface itself; they only exist in this plugin's JS), and the full custom catalog below is what actually works.

## Full custom trigger catalog, by element/feature

Most are plain custom DOM event names, dispatched by that element's own JS and requiring no extra settings — just set `trigger` to the literal string, listed here without further note. A few (marked **templated**) require an extra companion setting — see the next section.

| Element / feature | Trigger(s) |
|---|---|
| Header (`#brx-header`) | `x_sticky_header:active`, `x_sticky_header:inactive`, `x_hide_header:active`, `x_hide_header:inactive` |
| `xmediaplayer`/`xmediaplayeraudio` | `xmediaplayer-started`, `xmediaplayer-ended`, `xmediaplayer-pause`, `xmediaplayer-seeked`, `xmediaplayer-replay`, `xmediaplayer-resume`, `xmediaplayer-x_media_player:switched-media`, `xmediaplayer-time-update`, `xmediaplayer-time-update-end` (both take `xMediaPlayerTime`/`xMediaPlayerTimeMinutes`, or `xMediaPlayerTimeFormat: "percentage"` + `xMediaPlayerTimePercentage`), `xmediaplayer-watch-time` (cumulative watch time, same time-format options) — most also respect `xMediaPlayerFullscreenAction` (`true`/`false`/`wait`/`exit`) governing whether the interaction fires while the player is fullscreen |
| `xtoggleswitch` | **templated:** `x_toggle_switch:toggled_{label number}` (+ `xToggleSwitchLabelNumber`); plain: `x_toggle_switch:change` |
| `xproaccordion` | **templated:** `x_accordion:expand_{index}` (+ `xAccordionItemIndex`); plain: `x_accordion:expand` |
| `xtabs` (accordion mode) | **templated:** `x_tabs_accordion:expand_{index}` (+ `xAccordionTabsItemIndex`); plain: `x_tabs_accordion:collapse`, `x_tabs_accordion:expand` |
| `xtabs` | `x_tabs:accordion`, `x_tabs:tabs`, `x_tabs:switch`, and `x_tabs:tab_` + a literal `xTabNumber` value appended (e.g. `xTabNumber: "2"` → listens for `x_tabs:tab_2`) |
| `ximagehotspots` | **templated:** `x_image_hotspot:selected_{index}` (+ `xImageHotspotsIndex`) |
| `xfavorite` | **templated:** `x_favorite:count-reached_{count}` (+ `xFavoriteCountReachedCount`); plain: `x_favorite:added`, `x_favorite:removed`, `x_favorite:cleared`, `x_favorite:maximum-reached` |
| `xpromodalnestable` | `x_modal:open`, `x_modal:close` |
| `xproalert` | `x_alert:show`, `x_alert:close` |
| `xoffcanvasnestable` | `x_offcanvas:open`, `x_offcanvas:close` |
| `xdynamiclightbox` | `x_lightbox:open`, `x_lightbox:close` |
| `xcopytoclipboard` | `x_copy:copied`, `x_copy:failed`, `x_copy:reset`, `x_copy:empty` |
| `xtoastnotification` | `x_notification:show`, `x_notification:close` |
| `xcountdown` | `x_countdown:ended` (always behaves as if `runOnce` were set, regardless of the interaction's own `runOnce` value) |
| `xtableofcontents` | `x_toc:link-clicked` |
| `xmediaplaylist` | `x_media_playlist:active`, `x_media_playlist:inactive`, `x_media_playlist:paused`, `x_media_playlist:playing`, `x_media_playlist:ended` |
| `xreadmoreless` | `x_readmore:collapsed`, `x_readmore:expanded`, `x_readmore:collapse`, `x_readmore:expand` |
| `xslidemenu` | `x_slide_menu:expand`, `x_slide_menu:collapse` |
| `xlottie` | `x_lottie:complete` |
| `xpagetour`/`xpagetourstep` | `x_page_tour:completed`, `x_page_tour:cancelled`, `x_page_tour:started`, `x_page_tour_step:shown`, `x_page_tour_step:hidden` |
| `xproslider` | `x_slider:moved`, `x_slider:move` (bound to the underlying slider library instance, not a plain DOM event — waits for the slider's own `x_slider:init` event before attaching); `x_slider:active-slide` + `xSliderNumber` (fires only when the slide that becomes active matches that index) |
| WS Form (`xwsforms`) | `wsf-submit-success`, `wsf-submit-error`, `wsf-reset-complete`, `wsf-validate-fail` (scoped to the specific form instance inside the element) |
| Fluent Forms (`xfluentform`) | `fluentform_submission_success`, `fluentform_reset`, `fluentform_submission_failed` |

## Templated triggers: keep the literal `{placeholder}` text, don't substitute the number yourself

For triggers marked **templated** above, the `trigger` field's stored value keeps the placeholder text **verbatim** (e.g. `"x_accordion:expand_{index}"`, `"x_favorite:count-reached_{count}"`, `"x_toggle_switch:toggled_{label number}"`) — the JS strips the placeholder off the end of that exact string at runtime and appends the real number taken from the separate companion setting (`xAccordionItemIndex`, `xFavoriteCountReachedCount`, `xToggleSwitchLabelNumber`, etc.). Writing the real number directly into the `trigger` field instead of the placeholder (e.g. `"x_accordion:expand_2"`) does **not** work — the string-slice logic expects the placeholder's exact character length and silently produces a broken listener target if it's missing. Always set both: the literal placeholder-containing trigger string, and the matching companion field with the real value.

## Actions: `action: "javascript"` calling a `bricksextras.<namespace>.<method>` function

There is no BricksExtras-specific action *type* — every BricksExtras action is invoked through Bricks' own native `javascript` action, with `jsFunction` set to a global function BricksExtras exposes on `window.bricksextras`. Each element with runtime-controllable behavior registers its own namespace (only if `bricksextras` already exists, so load order matters — this is handled automatically by the plugin's own script dependencies).

Every method takes `brxParam` as its first argument (pass `%brx%` for this one) and reads `brxParam.target` to get the resolved element. Methods with a second parameter need an extra `jsFunctionArgs` entry supplying a literal value (number/string) after the `%brx%` entry — full method list, confirmed directly from each element's JS:

| Namespace | Methods | Notes |
|---|---|---|
| `bricksextras.toast` | `show(brxParam, message, announce, duration, contextIcon, dismissible, closeButton)`, `showByNotificationId(notificationId, args)` | Extra params after `brxParam` are all optional overrides of the notification's own configured settings. |
| `bricksextras.alert` | `show`, `hide`, `toggle` | `xproalert` only. |
| `bricksextras.modal` | `open`, `close` | `xpromodalnestable`. |
| `bricksextras.offcanvas` | `open`, `close`, `toggle` | `xoffcanvasnestable`. |
| `bricksextras.slidemenu` | `expand`, `collapse`, `toggle` | |
| `bricksextras.popover` | `open`, `close` | `xpopover`. |
| `bricksextras.accordion` | `open(brxParam, number)`, `close(brxParam, number)`, `toggle(brxParam, number)`, `openall`, `closeall` | **Inconsistent indexing:** `open`'s `number` is 0-indexed (used directly as an array index); `close`'s and `toggle`'s `number` is 1-indexed (the JS subtracts 1 before use). Passing the same `number` to `open` vs. `close`/`toggle` targets different items — double-check which one you're calling. |
| `bricksextras.tabs` | `goto(brxParam, number)`, `prev`, `next` | `number` (0-indexed) is the tab index. |
| `bricksextras.slider` (Pro Slider) | `forward(brxParam, number)`, `backward(brxParam, number)`, `toslide(brxParam, number)`, `topage(brxParam, number)` | `number` is a step count (forward/backward, default 1) or a target slide/page index (toslide/topage, 0-indexed). `toslide` treats `-1` specially — jumps to the last slide rather than erroring. |
| `bricksextras.readmore` | `toggle`, `expand`, `collapse` | |
| `bricksextras.lottie` | `play`, `reverse`, `stop` | |
| `bricksextras.pagetour` | `start`, `maybestart`, `resume` | `maybestart` respects the tour's own show-again/show-once settings; `start` always starts regardless. |
| `bricksextras.panorama` | `load` | `xpanoramaviewer`. |
| `bricksextras.mediaplayer` | `play`, `pause`, `toggleplay`, `togglemute`, `enterfullscreen`, `exitfullscreen`, `skip(brxParam, number)`, `load(brxParam, remember)` | `skip`'s `number` is seconds to skip (+/-); `load`'s `remember` (boolean) controls whether playback position persists. |
| `bricksextras.calendar` | `next`, `prev`, `nextYear`, `prevYear`, `today`, `gotoDate(brxParam, date)`, `view(brxParam, viewName)` | `xcalendar`. |
| `bricksextras.dynamictable` | `refresh`, `export(brxParam, title)` | `title` (optional) names the exported file. |
| `bricksextras.tableOfContents` | `open`, `close`, `toggle` | Note the capital `C` — `tableOfContents`, not `tableofcontents`. |
| `bricksextras.click` | `click(brxParam)` | Ungrouped universal helper — simulates a real click on the resolved target. |

Confirmed example (`xfavorite`'s `x_favorite:added` triggering a toast show):

**This JSON is an example, not the schema.** Check `references/elements/interactions.json` (or the live schema ability, per `bricksextras-start-here`) before building from it — do not copy settings out of this block without confirming they still exist and mean what's shown here.

```json
{
  "id": "rybdnk",
  "trigger": "x_favorite:added",
  "action": "javascript",
  "target": "custom",
  "targetSelector": ".favorites-toast-class",
  "jsFunction": "bricksextras.toast.show",
  "jsFunctionArgs": [{ "jsFunctionArg": "%brx%", "id": "palaec" }]
}
```

- `target: "custom"` + `targetSelector` is Bricks' own native mechanism for resolving which element the interaction acts on — here, a class placed on the specific `xtoastnotification` to reveal (same class-based targeting rule as everywhere else in BricksExtras — see `bricksextras-start-here`).
- `%brx%` is a Bricks-native placeholder (not BricksExtras'), resolved at runtime to a context object built from the interaction's own `target` resolution — `bricksextras.*` action functions expect exactly this shape and read `brxParam.target` to get the resolved DOM element.
- Most `bricksextras.<namespace>.show/open/etc.` functions check that the resolved target actually is an instance of the expected element (e.g. `bricksextras.toast.show` checks `target.classList.contains('brxe-xtoastnotification')`) before acting — pointing `targetSelector` at the wrong kind of element silently does nothing.

Literal (non-`%brx%`) arguments for methods with a second parameter use the identical `jsFunctionArg` key, just with the literal value instead of the placeholder — confirmed via whatever interactions-write ability the current MCP connection exposes (e.g. `bricks/update-element-interactions` on the native Bricks MCP):

**This JSON is an example, not the schema.** Check `references/elements/interactions.json` (or the live schema ability, per `bricksextras-start-here`) before building from it — do not copy settings out of this block without confirming they still exist and mean what's shown here.

```json
"jsFunctionArgs": [
  { "jsFunctionArg": "%brx%" },
  { "jsFunctionArg": "2" }
]
```

(e.g. for `bricksextras.accordion.open(brxParam, number)`, opening accordion item index `2` on the resolved target).

## Re-binding after AJAX

BricksExtras' custom-trigger listeners are re-attached automatically after Bricks' own `bricks/ajax/load_page/completed`, `bricks/ajax/pagination/completed`, `bricks/ajax/popup/loaded`, and `bricks/ajax/end` events — so interactions using these custom triggers still work on content loaded via Bricks AJAX pagination or popups, without extra configuration.

## Build workflow

1. Decide the trigger: a core Bricks trigger (`click`, `formSubmit`, etc.) if the interaction should fire on standard interaction, or one of the BricksExtras triggers above if it should react to another BricksExtras element's own state change.
2. If the trigger is templated (`{index}`/`{count}`/`{label number}`), keep the placeholder text verbatim in `trigger` and set the matching companion field to the real number.
3. For the action, use `action: "javascript"` with `jsFunction` set to the matching `bricksextras.<namespace>.<method>` from the reference table above — methods taking a second parameter (index/number/date/etc.) need an extra `jsFunctionArgs` entry with that literal value after the `%brx%` one.
4. Pass `%brx%` as a `jsFunctionArgs` entry when the target function expects `brxParam.target` (the norm) — and set `target: "custom"` + `targetSelector` (a class on the specific instance to act on) rather than relying on `self`, unless the action is genuinely meant to act on the triggering element itself.
