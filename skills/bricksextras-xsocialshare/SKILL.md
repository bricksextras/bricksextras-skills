---
name: xsocialshare
description: "Use when building or debugging the Social Share element (xsocialshare) from BricksExtras: a repeater of pre-built share links for ~15 services plus custom/email/print/copy actions. Covers that the items repeater's schema default is never auto-applied, the two distinct 'custom URL' concepts at element vs item level, the three real openPopup behaviors, and how print/copy/mastodon are special-cased."
---

**Requires:** BricksExtras 1.7.3+ with xsocialshare enabled

# BricksExtras: Social Share (xsocialshare)

Shipped by the **BricksExtras** plugin. A non-nestable element rendering a `<ul>` of share links, one `items` repeater entry per service. Each built-in service (`facebook`, `twitter`/X, `linkedin`, `whatsapp`, `pinterest`, `telegram`, `vkontakte`, `xing`, `line`, `mastodon`, `threads`, `bluesky`, `email`, `print`, `copy`) already has a correct share-URL template and icon baked into the element — nothing to configure for the URL itself unless overriding per-item label/icon/aria-label.

**Before writing settings, read this element's schema JSON now — do this even if you haven't loaded `bricksextras-start-here` or `bricksextras-element-schemas` this session:** open `references/elements/xsocialshare.json` inside the `bricksextras-element-schemas` skill directory and read it. If that file is missing, or its `schemaVersion` doesn't match the live BricksExtras plugin version, fall back to whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP) for `xsocialshare` instead. Do not proceed to write settings JSON until you've actually opened one of these two sources this session.

## `items` has no fallback if empty

`render()`: `$items = ! empty($this->settings['items']) ? $this->settings['items'] : false;` then, if still falsy, returns `render_element_placeholder(...)` — which only renders inside the builder, not on the live frontend. An `xsocialshare` built with `items` omitted produces nothing in the rendered page at all, not even an empty wrapper. The control's schema shows a `default` array of five preset services, but per `bricksextras-start-here`'s "schema defaults are UI-only" rule, that default is never applied at render — this rule extends to `repeater`-type controls with literal array defaults, not just simple select/checkbox controls. **Always write real `items` explicitly.**

## Two distinct "custom URL" concepts — same field names, different scope

- **Element-level**: `shareURL: "custom"` + top-level `customURL`/`customText` — overrides the URL and title used by *every* service link on the element (e.g. sharing a different page's URL than the one the element is placed on).
- **Item-level**: an `items` entry with `service: "custom"` + that item's own `customURL`/`customURLDynamic` — adds one additional, fully custom share link/service alongside the built-in ones (its own separate target URL, independent of `shareURL`).

These are unrelated despite sharing a name — don't conflate "override what gets shared" (element-level) with "add a custom share button" (item-level).

## Per-item `display` overrides the element-level `overallDisplay`

`render()`: `$display = isset($item['display']) ? $item['display'] : $overall_display;` — the schema's own `info` text ("This can be also changed individually for each link") is accurate. Set `overallDisplay` for the default across all items, override `display` on individual `items` entries as needed.

## `openPopup` has three genuinely different behaviors

- **`true`** (default) — real click-intercepted popup window (600×600, centered), for every service link *except* `email`, `print`, `copy`, and `mastodon`.
- **`false`** — no JS interception; the link just carries `target="_blank"` and opens a normal new tab.
- **`same`** — no `target` attribute at all; normal same-tab navigation.

## Mastodon is always special-cased, regardless of `openPopup`

Clicking a Mastodon share link always triggers a native browser `prompt()` asking for the visitor's own Mastodon instance domain (there's no single central Mastodon share endpoint), then builds the share URL from that input at click time. It still respects `openPopup` for *how* that resulting URL opens (popup window vs new tab), but the domain prompt itself happens unconditionally.

## `print` and `copy` render as `<button>`, not `<a>`, and have their own mechanics

- **`copy`** — uses `navigator.clipboard.writeText()`, which requires HTTPS; on failure (e.g. non-HTTPS), logs a console warning rather than showing a user-facing error. If `copiedLabel` is set on the item, the visible label text swaps to it after a successful copy.
- **`print`** — isolates and prints only the element matched by that item's `printSelector`, not the whole page (via a DOM-tree-walking approach that hides non-ancestor siblings during `window.print()`, then restores them). **Inside a query loop, `printSelector` is resolved relative to the clicked instance** (`closest()`) rather than as a single global match — meaning the same `printSelector` value correctly targets each repeated loop item's own content rather than always printing the first match on the page.

## The `data-x-social` JSON attribute does not contain item data

This attribute only ever carries `isLooping` (set when the element is inside a running Bricks query loop). All actual service items are rendered as real, static `<li><a>`/`<li><button>` markup server-side — don't expect to find share-link configuration by inspecting this attribute when debugging; read the rendered `<li>` elements directly instead.

## Other settings

- **`brandColors`** — a `select` control with string options `"true"`/`"false"`, not a checkbox; sending the boolean `true` is silently ignored (no class added, no error) — confirmed live. Send the string `"true"` to get the `x-social-share_brand-colors` class added to every link, alongside each item's own service class (`.facebook`, `.twitter`, etc.), for brand-color CSS to hook into.
- **`shareURL: "current"`** (default) resolves via `get_the_permalink()` normally, but on an archive page outside any query loop it's built manually from the request URL instead (since a permalink function doesn't apply there).
- **Per-item `customAttributes`** — arbitrary extra HTML attributes on that item's link/button, with dynamic-data support on the value.
- Extensive typography/color/border/spacing controls for the link, label, and icon — direct CSS mappings, no hidden behavior.

## Rendered DOM (for custom CSS/targeting)

Fully server-rendered, no JS dependency for the markup itself:

```html
<ul class="brxe-xsocialshare" data-x-popup="true" data-x-social="[]">
  <li class="x-social-share_item">
    <a class="x-social-share_link facebook x-social-share_brand-colors" href="..." rel="nofollow noopener" target="_blank" aria-label="Share on Facebook">
      <span class="x-social-share_icon"><svg>...</svg></span>
      <span class="x-social-share_label">Facebook</span>
    </a>
  </li>
  <li class="x-social-share_item">
    <button class="x-social-share_link print" aria-label="Print" data-print-selector=".article-content">
      <span class="x-social-share_icon"><svg>...</svg></span>
      <span class="x-social-share_label">Print</span>
    </button>
  </li>
  <li class="x-social-share_item">
    <button class="x-social-share_link copy" data-copy-url="https://example.com/" data-copied-label="Copied!" aria-label="Copy URL">
      <span class="x-social-share_icon"><svg>...</svg></span>
      <span class="x-social-share_label">Copy URL</span>
    </button>
  </li>
</ul>
```

Notes:

- **`print` and `copy` render as real `<button>` elements; every other service renders as a real `<a href>`** — matches the "renders as button, not a" note above, now with the exact markup. `data-print-selector`/`data-copy-url`/`data-copied-label` are how those two items' JS-only mechanics get their config, not `data-x-social` (which stays `[]`, per the note above).
- Every item's own service class (`.facebook`, `.twitter`, `.email`, `.print`, `.copy`, etc.) sits directly on the `.x-social-share_link` element — this, plus `.x-social-share_brand-colors` when enabled, is the real per-service styling hook.
- **`overallDisplay`/per-item `display` conditionally omits the icon or label span server-side — confirmed live** (`display: "icon"` produces no `.x-social-share_label` span at all, not a CSS-hidden one). Don't write CSS assuming both spans always exist; check which `display` value is in effect before targeting one specifically.

## Build workflow

1. Always populate `items` explicitly — there is no working fallback.
2. Set `overallDisplay`, override `display` per item only where it should differ.
3. If sharing a different URL/title than the current page, use element-level `shareURL: "custom"` + `customURL`/`customText` — don't confuse this with an item-level `service: "custom"` entry, which adds a separate custom link instead.
4. Choose `openPopup` deliberately — `true` for the popup-window experience, `false` for a plain new tab, `same` for same-tab navigation.
5. For `print`, set `printSelector` to a real class on the content that should actually print, and verify it inside a query loop specifically if used there.
