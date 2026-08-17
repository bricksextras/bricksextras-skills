# BricksExtras Skills v0.1 (beta)

## Requirements
Bricks 2.4+, BricksExtras 1.7.3+ and an MCP-compatible client that supports skills.

If the client you are running in cannot install or load skill/plugin-style guidance files, stop here and tell the user that this client is not supported.

## Step 1: Fetch the repository

Clone this repository into a fixed local checkout directory — do **not** rely on the repo's own name, since it may change. Use this exact path so later steps and future updates work regardless of the repo's name:

```bash
git clone https://github.com/bricksextras/bricksextras-skills.git ~/.bricksextras/skills/bricksextras-skills
```

If a checkout already exists at that path, update it instead of re-cloning:

```bash
git -C ~/.bricksextras/skills/bricksextras-skills pull
```

### Preferred: latest tagged release

If the repository has published GitHub Releases, check out the latest tag instead of `main`:

```bash
cd ~/.bricksextras/skills/bricksextras-skills
LATEST_TAG=$(curl -fsSL https://api.github.com/repos/bricksextras/bricksextras-skills/releases/latest | grep -m1 '"tag_name"' | cut -d '"' -f4)
[ -n "$LATEST_TAG" ] && git fetch --tags && git checkout "$LATEST_TAG"
```

If no releases exist yet (`releases/latest` returns 404), stay on `main` — this is expected while the package is in beta.

All skill folders live under `skills/`, each prefixed `bricksextras-`.

## Step 2: Link the skills into your client

Symlink (do not copy) the skill folders into your client's skills directory, so a future `git pull` / re-run of Step 1 updates everything downstream without repeating this step.

**Claude Code (system-wide, all projects):**
```bash
mkdir -p ~/.claude/skills
for d in ~/.bricksextras/skills/bricksextras-skills/skills/*/; do
  ln -s "$d" ~/.claude/skills/"$(basename "$d")"
done
```

**Claude Code (this project only):**
```bash
mkdir -p .claude/skills
for d in ~/.bricksextras/skills/bricksextras-skills/skills/*/; do
  ln -s "$d" .claude/skills/"$(basename "$d")"
done
```

**Other MCP-compatible clients:** Symlink each folder under `skills/` into whatever directory your client scans for skills. If your client only supports a single downloadable archive (no filesystem/git access), download the `.zip` asset attached to the latest GitHub Release instead of cloning — do not attempt to invent a plugin/marketplace install method if the client doesn't have one.

If a target symlink already exists, leave it alone and report it as skipped rather than overwriting.

## Step 3: Confirm and report

List the skill folder names that were linked (not copied), and tell the user whether their client needs a restart or a new chat/session before the skills become available — most clients load the skills list once at session start.

## What to load

`bricksextras-start-here` isn't optional — it's the one skill every session should load regardless of what you're building. Add the element-specific ones on top, as needed.

## What's included

| Skill | Covers |
|---|---|
| **bricksextras-start-here** | The stuff that applies everywhere: naming conventions, MCP schema fetching, targeting other elements, and where BricksExtras elements diverge from Bricks core. Always load this one. |
| **bricksextras-element-schemas** | Bundled, version-checked schemas for every element — the first stop before falling back to a live `get-element-schema` call. |
| **bricksextras-conditions** | Non-commerce `x_*` display conditions — post ancestry, category/tag, page type, taxonomy, loop position, dates, favorites, language/translation plugins. |
| **bricksextras-conditions-commerce** | Commerce and membership `x_*` conditions: WooCommerce cart/product/coupon state, EDD, MemberPress, RCP, WishList Member, SureMembers, PMP, Woo Subscriptions/Memberships, FluentCart. |
| **bricksextras-dynamictags** | Custom `{x_*}` dynamic-data tags — reading time, post terms, URL parameters, loop index, menu item fields, favorite IDs/counts, attachment metadata — and which need an active loop/query context to resolve. |
| **bricksextras-headerextras** | The sticky/overlay header controls injected into page and header-template settings, plus the data attributes that actually drive the behavior. |
| **bricksextras-interactions** | Wiring a Bricks Interaction to a BricksExtras-specific trigger or action — toasts, alerts, modals, offcanvas, slider events — instead of an element's own built-in trigger. |
| **bricksextras-interactive-controls** | Parallax, floating, tilt, and tooltip style-tab controls that BricksExtras adds to ~40 native Bricks elements. |
| **bricksextras-query-loop-extras** | The `queryLoopExtras` query type — adjacent posts, related posts, image galleries, nav menu loops, favorites lists. |
| **bricksextras-xaddtocalendar** | Add-to-calendar button/dropdown for Google, Outlook, Yahoo, and `.ics` downloads. |
| **bricksextras-xbacktotop** | Scroll-to-top button with progress-circle option. |
| **bricksextras-xbeforeafterimage** | Draggable before/after image comparison. |
| **bricksextras-xbreadcrumbs** | Breadcrumb trail, with pass-through support for Rank Math, Yoast, and other SEO plugins. |
| **bricksextras-xburgertrigger** | The hamburger/menu-toggle trigger button. |
| **bricksextras-xcalendar** | FullCalendar-backed event calendar — recurrence, exclusion dates, popups. |
| **bricksextras-xcolorschemetoggle** | Light/dark/system theme switch. |
| **bricksextras-xcontentswitcher** | Content panel switching, driven externally by a toggle switch element. |
| **bricksextras-xcontenttimeline** | Vertical or horizontal timeline, query-loop capable. |
| **bricksextras-xcopytoclipboard** | Copy-to-clipboard button. |
| **bricksextras-xcountdown** | Countdown timer, fixed-date and evergreen (per-visitor) modes. |
| **bricksextras-xdynamicchart** | Chart.js bar/line/doughnut charts from manual data or a query loop. |
| **bricksextras-xdynamiclightbox** | GLightbox-powered lightbox/modal, including inline content mode. |
| **bricksextras-xdynamictable** | Sortable, searchable, paginated Grid.js table. |
| **bricksextras-xfavorite** | AJAX favorite/wishlist button — add, remove, clear, count. |
| **bricksextras-xfluentform** | Fluent Forms embed. |
| **bricksextras-xheaderrow** | Styleable header row wrapper with per-row sticky/overlay visibility. |
| **bricksextras-xheadersearch** | Header search toggle across four layout modes. |
| **bricksextras-ximagehotspots** | Clickable/hoverable image hotspots with attached popovers. |
| **bricksextras-xinteractivecursor** | Custom animated cursor with hover/text/click states. |
| **bricksextras-xlottie** | Lottie vector animation with interactive triggers. |
| **bricksextras-xmediacontrol** | The individual control pieces (play, mute, slider, etc.) used inside a custom Media Player layout. |
| **bricksextras-xmediaplayer** | VidStack video/audio player — custom layouts, playlist mode. |
| **bricksextras-xmediaplayeraudio** | The audio-focused counterpart to Media Player. |
| **bricksextras-xmediaplaylist** | Playlist items that feed a sibling Media Player. |
| **bricksextras-xnestabletable** | HTML tables assembled from nested thead/tbody/tr/th/td elements, query-loop capable. |
| **bricksextras-xnotificationbar** | Dismissible announcement/alert bar. |
| **bricksextras-xoffcanvas** | Deprecated legacy OffCanvas (Template) element, superseded by xoffcanvasnestable — content comes from a selected Bricks Template rather than nested children. Only for editing existing pages that already use it. |
| **bricksextras-xoffcanvasnestable** | Slide-in/fade-in offcanvas panel, built from nested children directly. |
| **bricksextras-xpagetour** | Shepherd.js guided walkthrough / product tour. |
| **bricksextras-xpanoramascene** | A single scene inside a Panorama Viewer. |
| **bricksextras-xpanoramaviewer** | Pannellum.js 360°/equirectangular viewer, multi-scene tours. |
| **bricksextras-xpopover** | Popper.js popover/tooltip, including external-target tooltip mode. |
| **bricksextras-xproaccordion** | FAQ-style collapsible accordion. |
| **bricksextras-xproalert** | Dismissible alert/banner with localStorage-based show-again logic. |
| **bricksextras-xpromodal** | Deprecated legacy Modal (Template) element, superseded by xpromodalnestable — content comes from a selected Bricks Template or wysiwyg field rather than nested children. Only for editing existing pages that already use it. |
| **bricksextras-xpromodalnestable** | MicroModal.js dialog with a multi-trigger system, built from nested children. |
| **bricksextras-xproslider** | Splide-based carousel — manual or gallery-mode slides, slider-to-slider sync. |
| **bricksextras-xproslidercontrol** | External nav/progress/pagination/play-pause controls for Pro Slider. |
| **bricksextras-xproslidergallery** | Populates Pro Slider slides from a WordPress gallery. |
| **bricksextras-xqrcode** | Canvas/SVG QR code generator. |
| **bricksextras-xreadingprogressbar** | Scroll-based reading progress bar. |
| **bricksextras-xreadmoreless** | Readmore.js content collapse/expand. |
| **bricksextras-xshortcodewrapper** | Wraps content inside a third-party shortcode's tags. |
| **bricksextras-xslidemenu** | Nav-menu renderer with slide/collapse behavior and mega-menu support. |
| **bricksextras-xsocialshare** | Share links for ~15 services, plus custom/email/print/copy actions. |
| **bricksextras-xstarrating** | Star rating display — icon mode and fill-percentage mode. |
| **bricksextras-xtableofcontents** | tocbot-generated heading list. |
| **bricksextras-xtabs** | Tabbed content panels that collapse to an accordion on mobile. |
| **bricksextras-xtoast** | Sonner-based toast/snackbar notifications. |
| **bricksextras-xtoggleswitch** | Two-label or multi-label switch, drives a Content Switcher. |
| **bricksextras-xwsforms** | WS Form embed. |
