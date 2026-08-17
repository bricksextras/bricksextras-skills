---
name: xcalendar
description: "Use when building or debugging the Calendar element (xcalendar) from BricksExtras: displaying events from posts/CPTs on a FullCalendar-based calendar, recurring events, exclusion dates, event popups. Covers the exact value format each field-mapping and recurrence setting expects."
---

**Requires:** BricksExtras 1.7.3+ with xcalendar element enabled

# BricksExtras: Calendar (xcalendar)

Shipped by the **BricksExtras** plugin. Wraps FullCalendar with an RRULE recurrence engine. **Not nestable** — unlike most other BricksExtras elements, the query loop lives directly on the calendar element itself (`hasLoop`/`query`), not on a child block. There is no per-event child element to build; every event's data comes from a set of field-mapping settings, each a plain text field that accepts a dynamic tag.

The field-format tables below describe the actual parsing rules for each setting — several are stricter or more permissive than the builder UI hints suggest.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcalendar.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xcalendar` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Find the content model first — don't assume a CPT/fields exist

This element only displays data; it has no opinion on where that data lives. Before building:

1. Check for an existing events post type and field group: use whatever ability the current MCP connection exposes for listing registered post types and field-provider sources. Look for a post type that clearly models events (custom fields for start/end date at minimum) — the field group doesn't need to be named anything in particular, but a `date_time_picker` (or two) is the tell.
2. If a matching CPT + field group already exists, use it — check real field values on a few real posts via whatever dynamic-tag preview/resolve ability the current MCP connection exposes before wiring anything into the calendar's settings, same as any other dynamic-tag/query-loop build.
3. **If nothing suitable exists, this skill cannot create it for you.** There is no CPT-registration or ACF-field-group-creation ability on most MCP connections — CMS-source discovery abilities are explicitly read-only for third-party field providers. Creating a custom post type and its fields needs a different path: a developer, WP-CLI (if exposed), an ACF-specific MCP, or hand-written PHP in a child theme/plugin. What you *can* do is hand over an exact target spec, using the field-format table below, so whoever/whatever creates the fields knows precisely what shape each one needs to be (field type, and for `date_time_picker` fields specifically, that any common date format works — see Field mapping below).

---

## Query loop — on the element itself, not a child

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcalendar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xcalendar",
  "settings": {
    "hasLoop": true,
    "query": { "objectType": "post", "post_type": ["event"], "posts_per_page": -1 }
  }
}
```

Same `post_type`/`posts_per_page` snake_case requirement as every other BricksExtras query loop — see `bricksextras-start-here`. Use `posts_per_page: -1` (unlimited) — FullCalendar needs every event that could possibly appear across whatever date range a visitor navigates to, not just the current month, so capping the query hides events once a visitor pages far enough forward or back.

---

## Field mapping — exact value format per field

Every field below is a plain text control on the calendar element's own settings (`fields` control group), evaluated with `bricks_render_dynamic_data()` against the current loop item. Put a dynamic tag in each (e.g. `{acf_your_field_name}`), not a literal value.

| Setting | Required | Expected value | Parsing notes |
|---|---|---|---|
| `titleField` | Yes | Any text | Default `{post_title}` if omitted entirely (control placeholder only — still must be set explicitly in loop context). |
| `startField` | Yes | A date, or date+time | Very tolerant — tries ISO (`Y-m-d` or `Y-m-d\TH:i:s`) first, then a fallback list: `Y-m-d H:i:s`, `Y-m-d`, `d/m/Y H:i:s`, `d/m/Y H:i`, `d/m/Y g:i a`, `d/m/Y`, `m/d/Y H:i:s`, `m/d/Y H:i`, `m/d/Y g:i a`, `m/d/Y`, then generic date parsing as a last resort. ACF `date_time_picker`'s default display format (`d/m/Y g:i a`) works as-is. **Prefer ISO (`Y-m-d` or `Y-m-d H:i:s`) when you control the ACF return format**, since it's matched first and isn't ambiguous. |
| `endField` | No | Same as `startField` | If omitted (or unparsable), the event auto-becomes a 1-hour event (timed) or 1-day event (all-day) starting from `startField`. |
| `allDayField` | No | `"1"`, `"true"`, `"yes"`, or `"on"` (case-insensitive) to mean **true** — anything else means false | ACF `true_false` fields render `"True"`/`"False"` by default — `"true"` (lowercased) is in the truthy list, so this works out of the box. If using a custom truthy string, it must be exactly one of these four tokens; there's no generic boolean coercion. |
| `colorField` | No | A standard CSS color: hex (`#f00`, `#ff0000`), `rgb()`, or a named color | Passed through to FullCalendar's event `color`. The dynamic tag must resolve to one of these formats — anything else won't be applied for that event. |
| `displayField` | No | One of: `auto`, `block`, `list-item`, `background`, `inverse-background`, `none` | The control's own UI hint only mentions `auto, block, list-item, none` — `background` and `inverse-background` are valid too but undocumented in the builder. Anything else, or empty, falls back to `eventDisplay` (the element's own default-display setting). |
| `locationField` | No | Any text | Stored in `extendedProps.location` for use in popup/custom templates. |
| `descriptionField` | No | Text or HTML | stored in `extendedProps.description`. |

---

## Include the header toolbar by default

When a user adds this element through the builder UI, it comes with a full header toolbar (prev/next navigation, title, view-switcher buttons) already configured — that's the normal, complete state of the element. Building one via MCP without explicitly setting `headerToolbar`/`headerToolbarLeft`/`headerToolbarCenter`/`headerToolbarRight` does **not** reproduce that default; the calendar renders with no toolbar and no way to navigate. Always set these explicitly, matching the builder's own defaults, unless the user specifically asks for a bare/controls-less calendar:

```json
{
  "headerToolbar": true,
  "headerToolbarLeft": ["prev", "next"],
  "headerToolbarCenter": ["title"],
  "headerToolbarRight": ["dayGridMonth", "timeGridWeek", "listWeek"]
}
```

A calendar with events but no toolbar is an incomplete build, not a minimal one. The equivalent `footerToolbar*` settings are off by default — leave them off unless asked.

---

## Recurring events

`enableRecurring` is a **calendar-level** checkbox (on/off for the whole element), not per-event. When enabled, each looped event's `rruleRules` repeater is evaluated per-event using dynamic tags — **this is the key architectural pattern**: build one shared `rruleRules` template mapping every recurrence axis you might need to its ACF field, and let per-event blank values drop out naturally. An event with no recurrence data at all (`{acf_frequency}` renders empty) simply gets no RRULE and displays as a normal single event — you don't need a separate non-recurring code path.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcalendar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "enableRecurring": true,
  "rruleRules": [
    { "ruleName": "FREQ", "ruleValue": "{acf_frequency}" },
    { "ruleName": "INTERVAL", "ruleValue": "{acf_interval}" },
    { "ruleName": "COUNT", "ruleValue": "{acf_count}" },
    { "ruleName": "UNTIL", "ruleValue": "{acf_until}" },
    { "ruleName": "BYDAY", "ruleValue": "{acf_byday}" },
    { "ruleName": "BYMONTHDAY", "ruleValue": "{acf_bymonthday}" },
    { "ruleName": "BYMONTH", "ruleValue": "{acf_bymonth}" }
  ]
}
```

**Any `ruleValue` that renders empty for a given event is silently skipped for that event** — not an error, just omitted from that event's RRULE. This is what makes one shared template work across events with completely different recurrence shapes (weekly-by-day, monthly-by-date, count-limited, date-limited, non-recurring) simultaneously.

**`FREQ` is mandatory — without it, the entire RRULE is dropped and the event renders as non-recurring**, even with `enableRecurring: true` and other rules populated. If a looped event's frequency field is empty, that event just won't repeat; it won't error.

### Per-rule value format

| `ruleName` | Format | Notes |
|---|---|---|
| `FREQ` | One of `SECONDLY`, `MINUTELY`, `HOURLY`, `DAILY`, `WEEKLY`, `MONTHLY`, `YEARLY` (case-insensitive) | Anything else is dropped, which drops the whole RRULE. |
| `INTERVAL` | A positive integer | Non-numeric or `< 1` silently becomes `1`, not dropped. |
| `COUNT` | A positive integer | `< 1` or non-numeric → dropped. **Mutually exclusive with `UNTIL`: if both are present, `COUNT` is discarded and `UNTIL` wins.** Don't map both to non-empty values on the same event. |
| `UNTIL` | A date. Prefer ISO (`Y-m-d` or full ISO datetime) or the compact `YYYYMMDDTHHMMSS[Z]` form | Unlike `startField`/`endField`, this does **not** use the tolerant multi-format list — only the compact RRULE form is special-cased, everything else goes through generic date parsing, which is more ambiguity-prone for slashed dates. Use an ISO-formatted ACF return value here specifically. |
| `BYDAY` | Comma-separated tokens: optional sign + optional ordinal (1–5) + 2-letter day code, e.g. `MO`, `TU,TH`, `1MO`, `-1FR` | Regex-validated: `^([+-]?[1-5]?)?(MO\|TU\|WE\|TH\|FR\|SA\|SU)$` per token. Whitespace after commas is fine (each token is trimmed). If `FREQ=WEEKLY` and `BYDAY` is empty, the element auto-fills it from the event's own start-date weekday — so `BYDAY` is optional for simple "same weekday every N weeks" recurrence. |
| `BYDAY_MONTHLY` | Same format as `BYDAY`, e.g. `2TU` (2nd Tuesday), `-1FR` (last Friday) | Internally this is just an alias — the code renames it to `BYDAY` before validating, so it uses the exact same regex. It exists as a separate `ruleName` purely for UI clarity ("by weekday of month" vs. plain "by day of week"). |
| `BYMONTH` | Comma-separated integers, 1–12 | |
| `BYMONTHDAY` | Comma-separated integers, -31–31, excluding 0 | Negative values count from month-end (`-1` = last day of month). |
| `BYYEARDAY` | Comma-separated integers, -366–366, excluding 0 | |
| `BYWEEKNO` | Comma-separated integers, -53–53, excluding 0 | |
| `BYSETPOS` | Comma-separated integers, -366–366, excluding 0 | **Requires at least one other `BY*` rule present** (`BYDAY`/`BYMONTH`/`BYWEEKNO`/`BYYEARDAY`/`BYMONTHDAY`) to have something to filter — dropped otherwise. |
| `BYHOUR` | Comma-separated integers, 0–23 | |
| `BYMINUTE` / `BYSECOND` | Comma-separated integers, 0–59 | |
| `WKST` | A single 2-letter day code | |

**An ACF select field returning `"LABEL : Description"`-style values is tolerated** — the code splits on `:` and keeps only the part before it, so you don't need a separate "raw value" return format if your ACF select is set up that way.

### Exclusion dates (`exclusionDatesField`)

One dynamic tag, comma- **or** newline-separated, mixing any of three entry shapes freely:

- A specific date (any `startField`-compatible format) → excludes that one occurrence.
- `MM-DD` (e.g. `12-25`) → shorthand for "exclude this date every year."
- A raw RRULE fragment starting with `FREQ=`/`BYDAY=`/`BYMONTH=`/etc. → treated as a full recurring exclusion rule.

### Event overrides (`eventOverridesField`)

One dynamic tag, comma- or newline-separated lines, each line: `DATE START_TIME [DURATION]` (space-separated), e.g.:

```
2026-04-18 18:00 02:00:00
2026-05-02 09:30
```

`DATE` goes through the same date parsing as `startField`. `DURATION` is optional, format `HH:MM:SS` or `D.HH:MM:SS` for multi-day. Each override date is automatically added to the exclusion list (so the normal recurring instance doesn't also show on that date) and a modified instance is rendered at the overridden time instead.

---

## Click action and popup

- `eventClickAction`: `"url"` (default), `"popup"`, or `"none"`.
- `"url"` uses `eventUrlField` (defaults to `{post_url}` if the field itself is left set to that).
- `"popup"` uses `eventPopup`, whose value is a **Bricks popup template's post ID as a string** (e.g. `"31277"`) — not a name or slug. Get real IDs via whatever template-listing ability the current MCP connection exposes, filtered to popup templates (e.g. `bricks/list-templates` with `type: "popup"` on the native Bricks MCP). This restriction is only about the `eventPopup` field itself — it must be a literal template ID, not a dynamic tag. It does **not** mean the popup template's own *content* can't use dynamic tags — the popup template still resolves dynamic data normally against whichever event was clicked, same as any other popup.

### Two settings the popup template itself needs, beyond just wiring `eventPopup`

Setting `eventClickAction: "popup"` + `eventPopup: "<id>"` on the calendar is necessary but not sufficient — the popup template itself needs two things configured, or it either doesn't open at all or opens with the wrong (static) content:

1. **A template condition covering the page/post the calendar is actually viewed on.** Bricks popups need a condition to know where they're allowed to render, set via whatever template-condition ability the current MCP connection exposes (e.g. `bricks/set-template-conditions` on the native Bricks MCP). Without this, the popup template has empty `conditions`/`openers` — it's never registered as openable anywhere, and the JS call that's supposed to trigger it (`bricksOpenPopup`) silently does nothing. This is easy to miss because the popup is being triggered programmatically by the calendar's own JS, not through a normal Bricks opener element — so there's no builder-UI opener to remind you a condition is still needed.
2. **`popupAjax: true` on the popup template's settings**, set via whatever popup-settings ability the current MCP connection exposes (e.g. `bricks/update-popup-settings` on the native Bricks MCP). Without this, the popup renders once, statically, against whatever context it happened to first render in — not per-click against the specific event that was actually clicked. Symptom: the popup opens correctly, but shows the same (wrong, or blank) event content no matter which calendar event triggered it.

Both are required together — condition alone gets you a popup that opens but shows static/wrong content; AJAX alone (without a condition) gets you a popup that never opens at all.

---

## Styling: CSS variables, not rendered structure

Unlike most other BricksExtras elements, this skill doesn't include a rendered-DOM breakdown. The calendar's actual markup is FullCalendar's own generated output — deeply nested, view-dependent (month/week/day/list all render completely differently), and versioned by FullCalendar itself, not this plugin. **Confirmed live: FullCalendar's own elements (buttons, toolbar sub-divs, day cells) carry atomic/hashed classes regenerated per build** — `fc-classic-dl6`, `fc-classic-1Wx`, etc. — not stable semantic names. These are not meant to be hand-targeted with custom selectors at all; a class captured today may not exist in the next FullCalendar version bundled with this plugin, and a static DOM snapshot would misrepresent whichever view isn't currently shown anyway (month/week/day/list all render completely different internal structure). The supported customization surface is a fixed set of CSS custom properties, all applied to `.x-calendar` (FullCalendar's built-in "Classic" theme variables, wired up as element settings) — style through these, not `_cssCustom` against FullCalendar's internal classes.

**Two exceptions are genuinely stable and safe to target, confirmed live across multiple views:**

- **`.fc-header-toolbar`/`.fc-footer-toolbar`** — real, non-hashed classes that survive across FullCalendar versions (this plugin's own `toolbarDirection`/`toolbarJustifyContent`/`toolbarButtonsGap`/etc. controls already target them internally — see their `css` mappings in the schema). Safe to extend with custom CSS if a toolbar layout tweak isn't covered by those controls.
- **`data-calendar-view` on the `.x-calendar` root** — a BricksExtras-owned attribute (not FullCalendar's own), confirmed live to update on every view switch (`dayGridMonth` → `timeGridWeek` → `listWeek`, etc., matching exactly the view name strings used in `initialView`/toolbar button settings). This is the correct, stable hook for any styling that should differ by view — e.g. `.x-calendar[data-calendar-view="listWeek"] { ... }` — without touching any FullCalendar-internal class.
- The view-switcher buttons also expose real ARIA state (`role="tablist"`/`role="tab"`/`aria-selected="true"`) — a more stable way to identify the active view button via `[aria-selected="true"]` than any class, if that's ever needed (e.g. via `_cssCustom` attribute selectors), though `data-calendar-view` above is the simpler, element-level hook for most styling needs.

| Setting | CSS variable | Controls |
|---|---|---|
| `fcBackground` | `--fc-classic-background` | Overall calendar background |
| `fcForeground` | `--fc-classic-foreground` | Primary text color |
| `fcBorder` | `--fc-classic-border` | Grid line / cell border color |
| `fcFaintBg` | `--fc-classic-faint` | Subtle background shade (e.g. non-business hours) |
| `fcMutedBg` | `--fc-classic-muted` | Muted background (e.g. other-month / disabled cells) |
| `fcStrongBg` | `--fc-classic-strong` | Emphasized background (e.g. selected cell) |
| `fcFaintForeground` | `--fc-classic-faint-foreground` | Subtle/secondary text color |
| `fcMutedForeground` | `--fc-classic-muted-foreground` | Muted text color |
| `fcStrongBorder` | `--fc-classic-strong-border` | Emphasized border color |
| `fcButtonBg` | `--fc-classic-button` | Toolbar button background |
| `fcButtonBorder` | `--fc-classic-button-border` | Toolbar button border |
| `fcButtonForeground` | `--fc-classic-button-foreground` | Toolbar button text color |
| `fcButtonOutline` | `--fc-classic-button-outline` | Toolbar button focus outline |
| `fcButtonStrongBg` | `--fc-classic-button-strong` | Toolbar button active/hover background |
| `fcButtonStrongBorder` | `--fc-classic-button-strong-border` | Toolbar button active/hover border |
| `fcButtonStrongForeground` | `--fc-classic-button-strong-foreground` | Toolbar button active/hover text color |
| `fcEvent` | `--fc-classic-event` | Default event background |
| `fcEventContrast` | `--fc-classic-event-contrast` | Default event text color |
| `fcEventPadding` | `--fc-classic-event-padding` | Padding inside event blocks |
| `fcHighlight` | `--fc-classic-highlight` | Date-range selection highlight color |
| `fcBgEventColor` | `--fc-classic-background-event` | Color of "background event" display mode (shades a time range instead of showing a block) |
| `fcBgEventOpacity` | `--fc-classic-background-event-opacity` | Opacity of that shaded region |
| `fcBgEventForegroundOpacity` | `--fc-classic-background-event-foreground-opacity` | Opacity of text within background events specifically |
| `fcToday` | `--fc-classic-today` | Today-cell highlight background |
| `fcNow` | `--fc-classic-now` | Now-indicator line/dot color |

---

## Never do

- Do not put `hasLoop`/`query` on a child element — `xcalendar` is not nestable; the loop lives directly on the calendar element's own settings.
- Do not map both `COUNT` and `UNTIL` to non-empty values expecting both to apply — `UNTIL` silently wins and `COUNT` is discarded.
- Do not target FullCalendar's own internal DOM/classes with `_cssCustom` for styling — use the `fc*` CSS-variable settings instead. The rendered markup varies by view and isn't a stable target.
- Do not assume `displayField` only accepts the four values the builder UI hints at (`auto`/`block`/`list-item`/`none`) — `background` and `inverse-background` are valid too.
- Do not use a slashed date format for `UNTIL` — it isn't run through the same tolerant multi-format parser as `startField`/`endField`; use ISO.
- Do not expect this skill (or any MCP ability) to create the underlying custom post type or field group — that capability doesn't exist here. Hand over the field-format spec instead.
- Do not leave the header toolbar settings unset and assume the calendar will still ship with navigation — it won't. Set `headerToolbar`/`headerToolbarLeft`/`headerToolbarCenter`/`headerToolbarRight` explicitly, matching the builder's own default, unless the user asked for a bare calendar.

## MCP write notes

- Check the bundled schema first per `bricksextras-element-schemas`; call the live schema ability only if the bundle is missing or stale.
- Verify every dynamic tag against real data with whatever dynamic-tag preview/resolve ability the current MCP connection exposes before wiring it into a field-mapping setting — especially recurrence fields, since a wrong tag renders empty and just silently produces a non-recurring event with no error.
- Recurrence, exclusions, and overrides are all client-side FullCalendar behavior that a settings dump can't confirm — check the rendered calendar itself.
- To verify events actually loaded after a write, don't count rendered event nodes — that only reflects whichever date range the calendar's current view happens to be showing, so real events outside that window are a false negative. Instead read the full parsed dataset straight off the element instance, independent of current view: `document.querySelector('<selector for this calendar instance>').xEventsData` (run via whatever live-JS-evaluation tool the current connection exposes). Check its count and a few real values (title/start/end) against what was intended.

## If needed: custom behavior via the live instance

For anything beyond this element's own controls, get the real FullCalendar instance from `window.xCalendar.Instances[dataXId]` (keyed by the `data-x-id` on the `.x-calendar-wrapper` root; also mirrored at `<wrapper>.calendarInstance` on that same element) in a Bricks Code element with `executeCode: true` set (otherwise the JS renders as inert text), and drive it directly via FullCalendar's own API (`calendar.refetchEvents()`, `calendar.gotoDate()`, etc.). It's registered synchronously on init, no artificial delay to wait out.
