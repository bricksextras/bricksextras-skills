---
name: xnotificationbar
description: "Use when adding notification bars to pages: dismissible header bars for announcements, sales, alerts. Covers content modes, dismiss icon requirement, and visibility settings."
---

**Requires:** BricksExtras 1.7.3+ with xnotificationbar element enabled

# BricksExtras: Notification Bar (xnotificationbar)

A **nestable** dismissible notification bar, typically used at the top of pages for announcements, sales, or alerts.


**Before building from any template below, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xnotificationbar.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xnotificationbar` instead. The templates and examples below show documented required structure and common patterns only — the schema file (or live call) is the source of truth for the complete, current set of available settings. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

---

## Critical: Dismiss icon must be explicitly set

**The schema shows a default icon (`ti-close` from Themify), but this default only applies when a user adds the element in the builder UI.** When building via MCP, the `dismiss_icon` field must be explicitly set or the close button will render with no icon.

**Always include:**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xnotificationbar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "dismiss_icon": {
    "library": "themify",
    "icon": "ti-close"
  }
}
```

This is the same default the builder uses. Without it, the dismiss button appears but has no visible icon.

---

## Two content modes

`notification_content` determines how content is added:

- **`"wysiwyg"`** (default) - Use `notification_wysiwyg` editor field for HTML content
- **`"nestable"`** - Add child elements (text, buttons, images, etc.)

**Example: Simple text notification**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xnotificationbar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xnotificationbar",
  "settings": {
    "notification_content": "wysiwyg",
    "notification_wysiwyg": "<strong>🎉 SALE TODAY!</strong> Get 50% off. Use code: SAVE50",
    "dismiss_icon": {
      "library": "themify",
      "icon": "ti-close"
    },
    "notificationBackground": {
      "color": {"hex": "ef4444"}
    },
    "notificationTypography": {
      "color": {"hex": "ffffff"},
      "text-align": "center"
    },
    "_padding": {"top": "1rem", "bottom": "1rem"},
    "show_again": "page_load"
  }
}
```

**Example: Nestable mode with child elements**

**This JSON is an example, not the schema.** Before copying settings out of this block, read `references/elements/xnotificationbar.json` in `bricksextras-element-schemas` (live schema ability as fallback if missing/stale) and confirm they still exist and mean what's shown here.

```json
{
  "name": "xnotificationbar",
  "settings": {
    "notification_content": "nestable",
    "dismiss_icon": {
      "library": "themify",
      "icon": "ti-close"
    }
  },
  "children": [
    {
      "name": "text-basic",
      "settings": {
        "text": "Limited time offer!"
      }
    },
    {
      "name": "button",
      "settings": {
        "text": "Shop Now"
      }
    }
  ]
}
```

---

## Visibility settings

`show_again` controls when the notification reappears after dismissal:

- **`"page_load"`** - Show on every page load (default)
- **`"dismiss"`** - Show again after X days/hours (set via `show_again_days`/`show_again_hours`)
- **`"never"`** - Never show again once dismissed
- **`"after"`** - Show again after X days/hours (set via `show_again_days`/`show_again_hours`)
- **`"evergreen"`** - Only show if evergreen countdown hasn't ended
- **`"manual"`** - Controlled via JavaScript

---

## Key settings

- **`dismissText`** - Optional text next to the close icon
- **`dismissAriaLabel`** - Accessibility label (default: "Dismiss")
- **`slideDuration`** - Animation duration when dismissing (default: 300ms)
- **`stickyDisplay`** - Behavior in sticky headers: `"hide"`, `"show"`, or `"always"` (default)

---

## Rendered DOM

```html
<div id="brxe-{id}" class="brxe-xnotificationbar" data-x-id="{id}" data-x-notification="{&quot;slideDuration&quot;:300,&quot;show_again&quot;:{&quot;type&quot;:&quot;page_load&quot;,&quot;options&quot;:{&quot;days&quot;:0,&quot;hours&quot;:0}}}" data-x-sticky="always">
  <p>Content here (wysiwyg) or real nested children (nestable mode)</p>
  <button class="x-notification_close" aria-label="Dismiss">
    <span class="x-notification_close-text">Close</span>
    <span class="x-notification_close-icon"><i class="ti-close"></i></span>
  </button>
</div>
```

`data-x-sticky` (this element's `stickyDisplay` value, defaulting to `always`) is always present on the root — same attribute/mechanism `xheaderrow` uses for Header Extras' sticky show/hide system (see `bricksextras-headerextras`/`bricksextras-xheaderrow`). It's not exclusive to `xheaderrow`: `header.js`'s sticky-promotion query is a generic `[data-x-sticky]` attribute match against everything inside the header template, not scoped to any specific element.

## Placed inside a header template: `header.js` specifically accounts for it

**When an `xnotificationbar` sits inside the actual header template (not just anywhere on the page), `header.js` gives it dedicated handling beyond the generic sticky-attribute mechanism** — confirmed in source, not guessed: the script specifically looks up `.brxe-xnotificationbar` inside the header, and factors its height into the header's total height for the sticky-scroll-trigger distance calculation (`xStickyHeaderScrollDistance`, see `bricksextras-headerextras`). When the bar is dismissed, it fires a real `x_notification:close` custom event that `header.js` listens for specifically, and recalculates/animates the header's effective height to exclude the now-gone bar — so the sticky trigger point adjusts live instead of leaving a stale offset that accounts for a bar that's no longer there. This only applies when the notification bar is actually inside the header template; one placed elsewhere on the page doesn't get this treatment.

## Never do

- Don't omit `dismiss_icon` — the close button will render with no icon
- Don't use both `wysiwyg` and `nestable` modes — pick one via `notification_content`
- Don't forget to set `show_again` if you want the notification to persist/reappear after dismissal
