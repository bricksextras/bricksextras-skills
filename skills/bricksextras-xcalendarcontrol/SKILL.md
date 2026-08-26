---
name: xcalendarcontrol
description: "Use when building or debugging Calendar Control (xcalendarcontrol) from BricksExtras: external Today/Prev/Next/view-switcher buttons, event-skip buttons, or a live view-title heading for the xcalendar element. Covers targeting mechanics, the per-view label field naming, and accessibility behavior."
---

**Requires:** BricksExtras 1.7.4+ with xcalendarcontrol element enabled

# BricksExtras: Calendar Control (xcalendarcontrol)

Shipped by the **BricksExtras** plugin, alongside `xcalendar` (the calendar itself — see skill `xcalendar`). Load the Calendar skill alongside this one for any build that needs both — which is most of them.

This is a **sibling utility element**, placed **outside** the calendar it targets (e.g. below the calendar's section, or in a page header). It renders exactly one piece of external UI per instance and has no calendar content of its own — it's a remote control, not the calendar.

**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcalendarcontrol.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `verifiedUpTo` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` with `parameters: {"elementName": "xcalendarcontrol"}` on the native Bricks MCP) instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Targeting a calendar (identical mechanism to xproslidercontrol)

Settings: `calendar` (select: `section` — within the same section / `selector` — specific CSS selector / `component` — within the same component) + `calendarSelector` (text, required when `calendar: selector`) + `componentScope` (select; defaults to false).

**`componentScope` is a select control, not a checkbox — send the string `"true"`, not the boolean `true`.** Sending a boolean is silently ignored (the control falls back to its unset/false state with no error). Same value-shape rule as `xproslidercontrol`'s `componentScope`.

Default (`calendar: section`) works for the common case of the control sitting in the same Bricks section as the calendar it targets — **no selector needed**, as long as both share an actual enclosing `section` element (not just adjacent root-level elements). Under the hood the control resolves `.x-calendar-wrapper` within that scope and reads the live FullCalendar instance off `window.xCalendar.Instances[id]`, with a retry loop (~250ms × 40 attempts, ~10s) that waits out the calendar element's own script finishing initialization — so a control that appears briefly unbound on a slow page load is normal, not a bug.

Use `selector` only when the control lives somewhere else in the layout (e.g. a sticky header). **Target by class, not `_cssId`** — see `bricksextras-start-here`'s "Targeting another element" section for why. Give the calendar a class via `_cssClasses` and point `calendarSelector` at `.thatClass`.

**If both the control and the calendar it targets live inside the same component, set `componentScope: "true"`.** Without it, `.thatClass`-based targeting matches the *first* element with that class anywhere on the page once the component is duplicated.

The control's root gets `aria-controls` pointing at the actual FullCalendar mount's id (`.x-calendar`, not the wrapper) once JS binds — and, separately, `data-x-control-for="{calendarId}"` on the control's root confirms which calendar it actually matched. Both are only present after JS runs, so check a real browser render, not raw HTML, to confirm targeting succeeded.

This element hard-depends on the Calendar element's own script (`x-calendar-control` is enqueued with `x-calendar` as a script dependency) — it cannot function on a page with no `xcalendar` element present.

---

## Control-type cheatsheet (`controlType`)

Each Control element renders exactly one of these — add multiple Control elements (as siblings) for multiple pieces of UI.

| `controlType` | Renders | Key companion settings |
|---|---|---|
| `viewButton` (default) | Switches the calendar to a specific view | `targetView` (required); `aria-pressed` stays live-synced to whichever view is actually active, via the calendar's `datesSet` event — so several `viewButton` controls side by side behave like a tablist |
| `prev` / `next` | Step to the previous/next date range | Calls the calendar's own `prev()`/`next()` method |
| `prevYear` / `nextYear` | Step by a full year | Calls `prevYear()`/`nextYear()` |
| `today` | Jump to today | Calls `today()` |
| `prevEvent` / `nextEvent` | Jump to the nearest event start in that direction, **without changing the current view type** | Uses `gotoDate()` against the nearest matching event start — works for recurring events too (reads the rrule occurrence set directly, not just what's currently rendered) |
| `title` | Live-updating heading showing the calendar's current view title (e.g. "August 2026") | `titleTag` (h1–h6/div/span, default h2); updates on every `datesSet` event; builder preview shows a placeholder current month/year since no real calendar state exists there |

`prev`/`next`/`prevYear`/`nextYear`/`today` **auto-disable** once they'd be a no-op against the calendar's configured navigation limits — real `disabled` attribute on the frontend, `aria-disabled` only (never a real `disabled`, so the builder can still select it) inside the builder. This ties directly into `xcalendar`'s `validRangeMode`/`validRangeStart`/`validRangeEnd`/`validRangeDaysBack`/`validRangeDaysForward` settings (see skill `xcalendar`) — if the calendar has no navigation limit configured, these buttons are simply never disabled.

**Note:** FullCalendar's own built-in toolbar (the calendar element's `headerToolbar`/`footerToolbar` settings) already offers `prev`/`next`/`today`/`prevYear`/`nextYear`/title/view buttons inline. Reach for this element when that UI needs to live somewhere else in the layout (a sticky header, outside the calendar's own box) or needs independent styling/positioning the built-in toolbar can't give it — same reasoning as `xproslidercontrol` vs. the slider's own built-in arrows/pagination.

---

## Content, icons, and per-type/per-view labels

`buttonContent` (only present for non-`title` types): `icon` / `text` / `icon-text` / `text-icon` / `nest` (add your own elements as the button's content).

**Default icons exist for `prev`, `next`, `prevYear`, `nextYear`, `prevEvent`, `nextEvent`, and `viewButton` — but NOT for `today`.** In icon-only content mode with no custom icon explicitly set, `today` has nothing to fall back to and silently renders its aria-label text as **visible plain text** instead of an icon — a real, silent behavior change, not an error. Set `buttonIcon` explicitly for `today` if icon-only is intended, or use `text`/`icon-text` content instead.

Each control type gets its own `label{Type}`/`ariaLabel{Type}` text-field pair (e.g. `labelPrev`/`ariaLabelPrev`, `labelToday`/`ariaLabelToday`), shown/required depending on both `controlType` and the current `buttonContent` (label fields only apply when content includes text; aria-label fields only apply when content is icon-only).

For `viewButton` specifically, the label pair is keyed by **`targetView`, not a fixed key** — `labelDayGridMonth`/`ariaLabelDayGridMonth`, `labelTimeGridWeek`/`ariaLabelTimeGridWeek`, etc., one pair per possible view — mirroring the Calendar element's own per-view label controls (`xcalendar`'s `labelDayGridMonth` and friends) so the two stay visually consistent by default. Get the exact view key right (from the schema's `targetView` enum) or the label field simply won't apply to the selected view.

---

## Gotchas

- **`today` has no default icon** (see above) — the one control type that doesn't follow the "icon content mode just works" pattern the others do.
- **`componentScope` takes the string `"true"`, not a boolean `true`** — identical trap to `xproslidercontrol`.
- **Auto-disable only fires if the calendar actually has a `validRange` configured.** No `validRangeMode` set on the calendar means `prev`/`next`/`prevYear`/`nextYear`/`today` are never disabled by this control, regardless of how far a user navigates.
- **This element cannot function without an `xcalendar` element already on the page** — its script hard-depends on `x-calendar`'s script having run and exposed `window.xCalendar.Instances`.
- **`prevEvent`/`nextEvent` change the date but not the view type** — a user on Month view stays on Month view, just jumped to whichever month contains the next/previous event.

---

## Reference pattern: title + prev/next + today, targeting by section

A calendar with a stripped-down `headerToolbar` (or none at all), replaced by four external Control elements sitting in the same Bricks section, using the default `calendar: section` targeting:

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcalendarcontrol.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
[
  { "name": "xcalendarcontrol", "settings": { "controlType": "title", "titleTag": "h2" } },
  { "name": "xcalendarcontrol", "settings": { "controlType": "prev", "buttonContent": "icon" } },
  { "name": "xcalendarcontrol", "settings": { "controlType": "next", "buttonContent": "icon" } },
  { "name": "xcalendarcontrol", "settings": { "controlType": "today", "buttonContent": "text" } }
]
```

`data-x-calendar-control` on each nav/view control carries `{"control":"section","calendar":false,"type":"{controlType}"}` (`viewButton` also adds `targetView`) — a static server-rendered placeholder, not a live binding-status indicator. The real confirmation of a successful bind is `data-x-control-for="{calendarId}"`, added by JS after it resolves the target, plus `aria-controls` pointing at the calendar's own mount id.

**`prev`/`next`/`prevYear`/`nextYear` exact markup** (prev shown; the others swap class suffix, `data-x-type`, and icon):

```html
<button class="x-calendar-control" data-x-type="prev" type="button" aria-label="Previous" data-x-calendar-control='{"control":"section","calendar":false,"type":"prev"}' data-x-control-for="{calendarId}" aria-controls="{calendarElId}">
  <span class="x-calendar-control_icon" aria-hidden="true"><svg>...</svg></span>
</button>
```

**`today` exact markup in `icon` content mode with no custom icon set** — the silent no-default-icon fallback in practice:

```html
<button class="x-calendar-control" data-x-type="today" type="button" ...>
  Today
</button>
```

(No icon span at all — the aria-label text ("Today") is rendered as the button's visible text instead, because `today` has no default icon to fall back to.)

**`viewButton` exact markup** — `aria-pressed` toggles live as the calendar's active view changes:

```html
<button class="x-calendar-control" data-x-type="viewButton" type="button" aria-pressed="true" data-x-calendar-control='{"control":"section","calendar":false,"type":"viewButton","targetView":"dayGridMonth"}' ...>
  Month
</button>
```

**`title` exact markup** — tag configurable via `titleTag`, text kept live-synced to the calendar's current view title:

```html
<h2 class="x-calendar-control" data-x-type="title" aria-live="polite">August 2026</h2>
```

---

## Workflow for adding controls to a calendar

1. **Identify the target calendar first.** If a specific page is already in scope, read its element tree and locate the `xcalendar` instance this control set should target.
2. **Get the current schema** for `xcalendarcontrol` — check the bundled schema first per `bricksextras-element-schemas`; only if the bundle is missing or stale, call whatever live schema ability the current MCP connection exposes. Don't assume control keys from memory, especially the per-view `viewButton` label pairs.
3. **Decide targeting.** Same-section placement + default `calendar: "section"` covers most cases with zero extra config. Inside a component with a single calendar, prefer `calendar: "component"`. Only reach for `calendar: "selector"` + a class on the calendar (+ `componentScope: "true"`, the string, if both live in the same component) when the control lives elsewhere in the layout. Never use `_cssId` for this.
4. **Consider whether to strip the calendar's own `headerToolbar`/`footerToolbar` buttons** for whichever pieces are being replaced by external Control elements, to avoid duplicate UI.
5. **Add one Control element per piece of UI.** Remember `today` needs an explicit icon if icon-only content is wanted.
6. **Insert and verify** — check the rendered HTML/`data-x-control-for` attribute rather than trusting the settings alone, especially for `selector`/`component` targeting.
