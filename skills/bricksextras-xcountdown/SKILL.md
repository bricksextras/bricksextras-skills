---
name: xcountdown
description: "Use when building or debugging the Pro Countdown element (xcountdown) from BricksExtras: a countdown timer with fixed-date and evergreen (per-visitor) modes. Covers the mode-dependent required fields (date/timezone vs fields) and the end action options."
---

**Requires:** BricksExtras 1.7.3+ with xcountdown element enabled

# BricksExtras: Pro Countdown (xcountdown)

Shipped by the **BricksExtras** plugin. Not nestable.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xcountdown.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xcountdown` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: `mode` decides which fields are required — and they're mutually exclusive

`mode` is `"fixed"` (same end date/time for every visitor) or `"evergreen"` (each visitor gets their own countdown, starting fresh from their first visit — e.g. "5 minutes left" that's true for everyone individually, not a shared clock hitting zero at the same moment for all visitors). **Don't default to `"fixed"` just because a countdown "shows a deadline" — check whether the deadline is the same wall-clock moment for everyone (fixed) or a duration that resets per visitor (evergreen).**

- **`mode: "fixed"`** requires `date` (`"YYYY-MM-DD HH:MM:SS"`) and `timezone` (one of the schema's `UTC±HH:MM` offsets or IANA zone names — an offset like `"UTC+10:00"` sidesteps DST-name ambiguity), plus `fieldsFixed` (a repeater of `{type, format}` — no `value`, since the duration is computed from `date`).
- **`mode: "evergreen"`** requires `fields` instead — a repeater of `{type, value, format}`, where **`value` is the actual duration for that unit** (this is where "5 minutes" gets set, not `date`). `date`/`timezone` are irrelevant in this mode; omit them.

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xcountdown.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xcountdown",
  "settings": {
    "mode": "evergreen",
    "fields": [
      { "type": "minutes", "value": 5, "format": "%M minutes" },
      { "type": "seconds", "value": 0, "format": "%S seconds" }
    ],
    "action": "text",
    "actionText": "<p>Time's up!</p>"
  }
}
```

Only include the unit rows actually needed (e.g. minutes + seconds for a short countdown) — there's no requirement to always include all four (days/hours/minutes/seconds).

---

## `action`: what happens at zero

`action` (`countdown`/`hide`/`text`/`redirect`/`sync`/`countUp`/`none`) controls end-of-countdown behavior, each with its own required companion fields:

| `action` | Companion settings |
|---|---|
| `text` | `actionText` (editor/HTML) — swaps the countdown display for this content |
| `redirect` | `redirectLink` or `redirectURL`, `redirectDelay`, `redirectDelayText`, `preventRedirect` |
| `countUp` | Continues counting upward past zero instead of stopping |
| `hide` / `sync` / `none` / `countdown` | No extra required fields beyond the base countdown config |

---

## Critical: the separator setting key is spelled `seperator`, not `separator`

The control's own label is "Separator" (correct spelling), but the actual settings key — and the character it holds — are `seperator` (missing the second `a`). Both the schema and the rendered `data-x-countdown` JSON carry the same misspelled key. `maybeSeperator` (enable/disable, defaults to enabled) gates whether it shows at all; both it and `seperator` have working PHP-side defaults (`"enable"` / `":"`) if left unset, so this only matters when changing the separator character — and if you do, use `seperator`. Authoring the correctly-spelled `separator` silently does nothing; it isn't a recognized key.

---

## Rendered structure — schema alone doesn't show this

```html
<div class="brxe-xcountdown" role="timer" aria-atomic="true"
     data-x-countdown='{"timezone":"UTC+10:00","mode":"evergreen","fields":[{"type":"minutes","value":"10","format":"%M minutes"},{"type":"seconds","value":"0","format":"%S seconds"}],"action":"text","actionText":"<p>Time up!</p>","seperator":":"}'>
  <div class="x-countdown_item">
    <span class="x-countdown_format"><span class="x-countdown_number">09</span> minutes</span>
  </div>
  <div class="x-countdown_seperator">:</div>
  <div class="x-countdown_item">
    <span class="x-countdown_format"><span class="x-countdown_number">53</span> seconds</span>
  </div>
</div>
```

- **One `.x-countdown_item` per row in `fields`/`fieldsFixed`, in the order you list them**, with `.x-countdown_seperator` inserted between adjacent items (not after the last one). Each item's `format` string is split at its `%M`/`%S`/etc. token: the token becomes `.x-countdown_number` (the only part that updates live), and the surrounding literal text (`" minutes"`, `" seconds"`) stays as plain sibling text inside `.x-countdown_format`.
- **`role="timer"` + `aria-atomic="true"`, with no explicit `aria-live`, is the correct accessible pattern already handled for you** — `role="timer"` carries an implicit `aria-live="off"` per the ARIA spec, which is the recommended behavior for a continuously-ticking countdown (screen readers shouldn't announce every second). No extra ARIA work needed here.
- **`data-x-countdown` includes a `timezone` value even in `evergreen` mode**, despite the settings guidance above saying to omit it — it's carried through as a shared config default (here, the site's own timezone) regardless of mode, not something evergreen countdowns actually use for their calculation. Seeing it in rendered output isn't evidence of a misconfiguration; the "omit it in evergreen mode" advice still stands, this is just what the shared JS config object always looks like.
- `value` fields render as strings in the JSON (`"10"`, `"0"`) even though you author them as JSON numbers — cosmetic, not a reason to author them as strings yourself.

---

## Never do

- Do not set `date`/`timezone` on an evergreen countdown, or `fields` (with `value`) on a fixed one — each mode reads its own dedicated repeater (`fields` for evergreen, `fieldsFixed` for fixed); the other is ignored.
- Do not assume "X minutes left" phrasing means a fixed end time — that's the evergreen use case unless the deadline is explicitly a shared, same-for-everyone moment.
- Do not leave `action: "text"` without `actionText` — nothing displays when the countdown reaches zero.
- Do not author a `separator` (correct spelling) setting expecting it to control the divider character — the real key is misspelled `seperator`.

## MCP write notes

- `mode`/required-field mismatches (e.g. `date` set on an evergreen countdown) produce a "valid" write with no visible error, and only show up as a countdown showing the wrong duration or never starting.