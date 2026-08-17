---
name: xaddtocalendar
description: "Use when building or debugging the Add to Calendar element (xaddtocalendar) from BricksExtras: a button/dropdown letting visitors add an event to Google/Outlook/Yahoo Calendar or download an .ics file. Covers the required calendarServices setting, dynamic-data mode, and field formats."
---

**Requires:** BricksExtras 1.7.3+ with xaddtocalendar element enabled

# BricksExtras: Add to Calendar (xaddtocalendar)

Shipped by the **BricksExtras** plugin. **Not nestable.** Renders a button that opens a small menu of calendar services (Google, Outlook, Yahoo, ICS download) for one specific event. Commonly placed inside a query loop over event posts (one button per looped event), or standalone on a single event's own page.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xaddtocalendar.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xaddtocalendar` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: `calendarServices` must be set explicitly, or nothing renders at all

Unlike most settings with a schema-declared default, `calendarServices` (the repeater listing which calendar services show in the dropdown) has **no runtime fallback**. If it's omitted from the element's settings, the element renders **nothing** — not a broken button, not a default set of services, literally an empty element with no output and no error.

**The default should be all four services, matching the schema's own declared default — set them explicitly unless told to show fewer:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xaddtocalendar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xaddtocalendar",
  "settings": {
    "calendarServices": [
      { "service": "google", "label": "Google Calendar" },
      { "service": "outlook", "label": "Outlook Calendar" },
      { "service": "yahoo", "label": "Yahoo Calendar" },
      { "service": "ics", "label": "Download ICS" }
    ]
  }
}
```

Each repeater row: `service` (`google`/`outlook`/`yahoo`/`ics`), `label` (display text), and optionally `icon`/`iconColor`/`background`/`textColor` for per-service styling. Only build a subset of the four rows if the user explicitly asks to limit which services show — don't drop any by default.

---

## Data mode: use `dynamicdata`, not `auto`, outside the calendar's own popup

`dataMode` is `"auto"` (default) or `"dynamicdata"`:

- **`"auto"`** reads event data from the calendar element's own popup context. It only works when this element sits inside an `xcalendar` event-click popup template — there, the calendar has already exposed the clicked event's data, and this element picks it up with no dynamic tags of its own.
- **`"dynamicdata"`** is what to use everywhere else — standalone on a page, inside a query loop, on a single event's own post page. It requires setting the event fields yourself via dynamic tags.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xaddtocalendar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xaddtocalendar",
  "settings": {
    "dataMode": "dynamicdata",
    "eventTitle": "{post_title}",
    "eventStart": "{acf_your_start_date_field}",
    "eventEnd": "{acf_your_end_date_field}",
    "eventAllDay": "{acf_your_all_day_field}",
    "eventLocation": "{acf_your_location_field}",
    "eventDescription": "{post_content}",
    "calendarServices": [
      { "service": "google", "label": "Google Calendar" },
      { "service": "outlook", "label": "Outlook Calendar" },
      { "service": "yahoo", "label": "Yahoo Calendar" },
      { "service": "ics", "label": "Download ICS" }
    ]
  }
}
```

## Field format, `dataMode: "dynamicdata"`

| Setting | Expected value | Notes |
|---|---|---|
| `eventTitle` | Any text | |
| `eventStart` | A date or date+time | Same tolerant parsing as `xcalendar`'s `startField` — ISO preferred, common ACF display formats also work. See `bricksextras-xcalendar` for the exact tolerance list if the source field's format is in question. |
| `eventEnd` | Same as `eventStart` | If empty/unparsable, defaults to a 1-hour (timed) or 1-day (all-day) event from the start. |
| `eventAllDay` | Truthy string to mean **true** — same accepted set as `xcalendar`'s `allDayField` (`"1"`/`"true"`/`"yes"`/`"on"`, case-insensitive) | ACF `true_false`'s default `"True"`/`"False"` output works as-is. |
| `eventLocation` | Any text | |
| `eventDescription` | Text or HTML | |

If this element is paired with an `xcalendar` on the same page pulling from the same event fields, map both elements to the **same** dynamic tags — there's no shared/inherited event data between them even when they're querying the same post.

---

## Rendered structure — schema alone doesn't show this

```html
<div class="brxe-xaddtocalendar x-add-to-calendar"
     data-mode="dynamicdata"
     data-tooltip-trigger="click" data-tooltip-placement="bottom" data-delay="0"
     data-offset-skidding="0" data-offset-distance="10" data-move-transition="50"
     data-event='{"title":"test","start":"2026-07-15T00:00:00","end":"2026-07-15T18:00:00","allDay":false,"extendedProps":{"description":"","location":""}}'>
  <button type="button" class="x-add-to-calendar_button" aria-expanded="false" aria-controls="x-add-to-calendar_menu-{id}">
    <span class="x-add-to-calendar_icon" aria-hidden="true"><svg>...</svg></span>
    <span class="x-add-to-calendar_text">Add to Calendar</span>
  </button>
  <div class="x-add-to-calendar_menu" style="display: none;">
    <a href="https://calendar.google.com/calendar/render?action=TEMPLATE&text=..." class="x-add-to-calendar_link" data-service="google" target="_self">
      <span class="x-add-to-calendar_service-icon" aria-hidden="true"><svg>...</svg></span>
      <span class="x-add-to-calendar_service-label">Google Calendar</span>
    </a>
    <!-- one <a> per calendarServices row: outlook, yahoo deep-link the same way -->
    <a href="data:text/calendar;charset=utf-8,BEGIN:VCALENDAR..." class="x-add-to-calendar_link" data-service="ics" target="_self" download="event.ics">...</a>
  </div>
</div>
```

- **A standard button-triggers-disclosure pattern**, not a native `<details>`/`<dialog>` — a real `aria-expanded`/`aria-controls` relationship between the button and the menu `<div>`, toggled via `style="display:none"`. Accessible out of the box; no extra ARIA wiring needed from you.
- **`data-event` is the plugin's own normalized model, not a passthrough of your dynamic tags.** Whatever tolerant format you feed `eventStart`/`eventEnd` (see the field-format table above), it gets parsed into plain ISO-without-offset (`"2026-07-15T00:00:00"`) here, `eventAllDay`'s truthy-string is coerced to a real boolean, and `eventDescription`/`eventLocation` land nested under `extendedProps`. Each service link's URL (Google/Outlook/Yahoo deep links, and the `ics` link's `data:text/calendar` URI) is built from this normalized model, not from your raw dynamic-tag output — so if a calendar link looks wrong, check `data-event` on the rendered element before suspecting the dynamic tag itself.
- **The `data-tooltip-*`/`data-offset-*`/`data-move-transition` attributes mirror the same popover-positioning config style used by `xcopytoclipboard`'s tooltip settings** (`delay`, `offsetSkidding`, `offsetDistance`, `popoverTranslateX`/`Y`) — but here they're fixed/internal to the menu's own open behavior, not exposed as settings on `xaddtocalendar` itself. Don't go looking for equivalent controls on this element; there aren't any.
- **Every service link (including the external Google/Outlook/Yahoo ones) renders `target="_self"`** — there's no setting to open them in a new tab. If a design needs that, it isn't achievable through this element's native settings.
- **The `ics` download always saves as the literal filename `event.ics`**, not derived from the event title — every card in a query loop downloads to the same filename. Not configurable via settings; if this matters for a design, it needs a workaround outside the element (or should be raised with the plugin author as a possible enhancement).

---

## Never do

- Do not omit `calendarServices` — the element renders nothing at all, not a fallback button. This is the single most common way to build this element and have it silently do nothing.
- Do not use `dataMode: "auto"` (or leave `dataMode` unset) outside an `xcalendar` popup template — there's no event context to read, so it has nothing to show.
- Do not assume a control's schema-declared `default` is applied when the key is omitted from settings — some are (informational only), some aren't (`calendarServices` renders nothing rather than falling back to its declared default list). Check rendered output for any setting whose absence you're relying on, don't assume.
- Do not build with fewer than the full four services unless the user explicitly asked to limit them.

## MCP write notes

- Always include `calendarServices` explicitly — treat it as required, not optional, regardless of what the schema marks as required.
- An empty `calendarServices` array produces a "valid" settings write with no visible error — only shows up as a missing element in the browser.