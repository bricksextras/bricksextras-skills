---
name: xtoast
description: "Use when building or debugging Toast (xtoast) and Toast Notification (xtoastnotification) from BricksExtras: a Sonner-based toast/snackbar system where xtoast is a positioning container and each nested xtoastnotification is a template that gets cloned and shown when its own trigger fires. Covers the container/notification split, the triggers repeater, and why notification content isn't visible in the normal page flow."
---

**Requires:** BricksExtras 1.7.3+ with xtoast/xtoastnotification enabled

# BricksExtras: Toast (xtoast) / Toast Notification (xtoastnotification)

Shipped by the **BricksExtras** plugin, built on the third-party **Sonner** toast library (`sonner-vanilla.js`). Two-part element pair: `xtoast` is a nestable **positioning/stacking container**; one or more `xtoastnotification` elements nest inside it, each representing one distinct toast message with its own triggers.

**Before writing settings, read both elements' schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xtoast.json` and `references/elements/xtoastnotification.json` inside the `bricksextras-element-schemas` skill directory and read them. If either file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for that element instead. Do not proceed to write settings JSON until you've actually opened these sources this session.

## The container/notification split

- **`xtoast`** — controls where toasts stack on screen (`position`: one of 6 corner/edge options, default `bottom-right`), how many show at once (`visibleToasts`, default 3), and spacing (`gap`, `offset`). It has no message content of its own.
- **`xtoastnotification`** — the actual message. **Its rendered HTML in the page is a template, not visible content** — `toast.js` clones the element's DOM node and hands the clone to Sonner's `toast.custom()` API only once that notification's own trigger condition fires. Don't expect to see an `xtoastnotification`'s content anywhere in normal page flow; it only appears in the DOM (inside Sonner's own toast stack) once triggered.

A single `xtoast` container can hold multiple `xtoastnotification` children, each with independent triggers — e.g. one that fires on page load and a separate one that fires on scroll, both stacking in the same corner.

## `triggers` repeater — same family of trigger types as `xpromodalnestable`, plus a looping option unique to toast

Types: `pageLoad` (`delay` ms), `pageLoadURLParameter` (`urlParameter`), `scroll` (`scrollDistance`, px or %), `scrolledToElement` (`selector`), `exitIntent`, `pageViews` (`pageViews` count), `elementClick`/`elementHover` (`selector`), and **`intervals`** — unique to toast: repeats the notification on a fixed `intervalSeconds` cadence, optionally forever via the `intervalLoop` checkbox. Good for periodic reminders/announcements rather than a one-time trigger.

`scrolledToElement`/`elementClick`/`elementHover` use the same class-not-`_cssId` targeting rule as everywhere else in BricksExtras — see `bricksextras-start-here`.

**The `triggers` repeater is optional, not the only way to show a toast.** The control group's own separator text states notifications "can be triggered via interactions, via code or via these options" — a Bricks Interaction (or custom JS) can trigger the same toast without any repeater row configured at all, useful for showing a toast in direct response to another action (e.g. "show a toast after this form submits" or "after this button is clicked") rather than a passive page-level condition.

## `show_again`/`show_once`/`show_once_session` — narrower option set than `xproalert`

Same reshow-tracking concept as `xproalert` (`page_load`/`never`/`after` + `show_again_days`/`show_again_hours`), but **no `dismiss` or `manual` option here** — those two are specific to `xproalert`'s always-visible-until-acted-on model, which doesn't apply to a toast that's inherently transient. `show_once`/`show_once_session` are mutually exclusive checkboxes (each schema-gated to the other being off), same pattern as `xpromodalnestable`.

## `duration`, `dismissible`, icons

- **`duration`** (ms, default `5000`) — set to `0` to disable auto-dismiss entirely (rendered to Sonner as `Infinity`).
- **`dismissible`** (select `enable`/`disable`, default enabled) — whether the toast can be swiped away by the visitor; unrelated to whether the close button shows.
- **`closeButton`** (checkbox) — adds a visible close button. Independent of `dismissible`: a toast can be swipe-dismissible without a visible button, or have a button without being swipeable.
- **`contextIcon`** (checkbox) — adds an icon on the notification's left side.
- **`closeButtonIcon`/`contextIconIcon`** — like `xfavorite`'s icons, both have real inline-SVG fallbacks in `render()` if left unset, so (unlike most BricksExtras icon controls) omitting them is safe rather than resulting in nothing rendering.

## Per-item notification inside a product/post loop (e.g. "Added to cart")

Placing an `xtoastnotification` **inside a query loop** (e.g. a WooCommerce product loop) creates one notification instance per loop item, each scoped to that specific product/post. `toast.js` matches a firing trigger (e.g. a WooCommerce "add to cart" event) against the specific loop item's own product ID (via `data-interaction-loop-id`, compared against the ID of whatever was actually just added) before showing that item's toast — so with, say, a "Added to cart" message templated per product in the loop, adding *Product A* to the cart shows only *Product A*'s toast, not every other product's notification in the loop firing at once.

## Other

- **`ariaLive`** (`polite`/`assertive`/`off`, default `polite`) — screen-reader announcement behavior for the toast's appearance.
- Layout/style controls (`_display`, `_padding`, `_position`, etc.) on `xtoastnotification` target `.x-toast-notification-inner` specifically, not the element root — check that selector when styling doesn't seem to apply.
- `builderHidden` (both elements) only affects builder-canvas visibility, same general rule as elsewhere.

## Rendered DOM (once triggered — for custom CSS/targeting)

Confirmed live: the triggered toast is portalled into Sonner's own `<ol data-sonner-toaster>` stack (appended to `<body>`, not inside `xtoast`'s own position in the page), with a real, cloned copy of the `xtoastnotification` element inside a Sonner `<li>`:

```html
<ol data-sonner-toaster data-sonner-theme="light" data-y-position="bottom" data-x-position="right">
  <li data-sonner-toast data-mounted="true" data-visible="true" data-removed="false" data-swiped="false"
      data-swiping="false" data-dismissible="true" data-expanded="false" data-front="true"
      class="x-toast-active" role="status" aria-live="polite">
    <div class="brxe-xtoastnotification x-toast-notification" data-x-notification="{...}">
      <div class="x-toast-notification-content">
        <span class="x-toast-context-icon" aria-hidden="true"><svg>...</svg></span>
        <div class="x-toast-notification-inner">
          <!-- the notification's own nested children render here, e.g. -->
          <h3 class="brxe-heading">Welcome!</h3>
          <div class="brxe-text-basic">Thanks for visiting.</div>
        </div>
        <button class="x-toast-close" data-x-toast-close><span class="x-toast-close-icon"><svg>...</svg></span></button>
      </div>
    </div>
  </li>
</ol>
```

Notes:

- **The `<li data-sonner-toast>` wrapper carries real, live-updated Sonner state attributes** — `data-mounted`, `data-visible`, `data-removed`, `data-swiping`, `data-expanded`, `data-dismissible`, `data-front` (whether it's the topmost toast in the stack), etc. These are genuine Sonner library hooks (not BricksExtras-specific), useful for state-based custom CSS (e.g. `[data-swiping="true"]` for a mid-swipe style) beyond what this element's own controls expose.
- **`.x-toast-active` is BricksExtras' own class**, added to that Sonner `<li>` alongside Sonner's own attributes — the one BricksExtras-owned hook at the wrapper level.
- **Confirms the existing rule above**: nested children render inside `.x-toast-notification-inner` specifically, sibling to `.x-toast-context-icon` and `.x-toast-close` inside `.x-toast-notification-content` — not directly inside the notification root.
- A raw HTML/settings read never shows this — it only exists once a trigger has actually fired in a real browser.

## Build workflow

1. Add one `xtoast` per on-screen stacking position needed (e.g. a separate container if some toasts should appear bottom-right and others top-center).
2. Nest one `xtoastnotification` per distinct message, each with its own `triggers` rows — or leave `triggers` empty and fire it via a Bricks Interaction/custom JS instead.
3. Set `duration: 0` for messages that must stay until manually dismissed, and pair with `closeButton` so there's a way to dismiss it.
4. Use `intervals` triggers for recurring/periodic messages; use the other trigger types for one-time, condition-based ones.
