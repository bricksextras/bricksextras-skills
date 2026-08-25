---
name: bricksextras-element-schemas
description: "Use before writing settings for any BricksExtras (x*) element. Bundled, version-checked schema reference — check here before calling the live schema ability, to skip the round trip when the bundle is current."
---

# BricksExtras: bundled element schemas

`references/index.json` and `references/elements/*.json` are a bundled copy of every BricksExtras element's control schema. Same source of truth as whatever live schema ability the current MCP connection exposes (e.g. `bricks/get-element-schema` on the native Bricks MCP), exported.

## Workflow

1. **Check the bundle's verified plugin version against the live plugin first.** Read `references/index.json`'s `verifiedUpTo` field (not `version` — that field is the pack's own independent release number and is never compared to the plugin), and compare it against the live BricksExtras plugin version — get that from whatever the current MCP connection surfaces as installed-plugin/system info. On the native Bricks MCP that's the `bricks/get-system-information` ability (read the `version` for the `plugins` array entry named `"BricksExtras"` — not `bricks/get-mcp-version`, which reports Bricks core/abilities/adapter/WordPress versions but never the BricksExtras plugin's own version). Other connections may report installed-plugin versions elsewhere instead — e.g. in connection/environment info supplied automatically at session start — check there before assuming no such information exists. Compare with real version ordering (numeric major.minor.patch, pre-release suffixes counted as older than the same numbers without one), not string equality. **If the live plugin version is greater than `verifiedUpTo`, the bundle is stale for this site — skip it and fetch the schema live instead**, the same as if the bundle didn't exist. Don't use a stale bundle "because it's probably close enough" — a settings shape that changed between versions fails silently, not loudly (see the `bricksextras-xproslider`/`bricksextras-xtabs` skills for what that failure mode looks like in practice). **Tell the user the bundle is stale and that you're falling back to a live schema call** — don't just silently work around it; this is exactly the condition `bricksextras-skills-update` exists to surface, so don't detect it here and stay quiet about it.
2. **If the live plugin version is at or below `verifiedUpTo`**, look up the element in `references/index.json` by `name` (e.g. `xcalendar`) to get its `schemaPath`, then read that file directly instead of calling the live ability.
3. **If the element isn't in the bundle at all** (new element shipped after the bundle was generated, or the bundle predates it), fall back to the live ability for that one element — a missing file is not a reason to guess. Mention this to the user too, for the same reason as step 1.

## Schema file shape

Each `elements/*.json` file:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "bricksextras://elements/xexample",
  "type": "object",
  "title": "Example Element",
  "metadata": { "name": "xexample", "category": "extras", "tag": "div", "nestable": false },
  "settings": { "...": "one entry per control, same shape as the live get-element-schema response" }
}
```

Element files carry no version field of their own — `verifiedUpTo` in `references/index.json` is the one field to compare in step 1 above, for the bundle as a whole.

## Never do

- Do not trust the bundle without checking `verifiedUpTo` against the live plugin version first — an outdated bundle looks identical to a current one until something silently doesn't work.
- Do not fall back to a live schema call for staleness or a missing element without also telling the user — silently working around it defeats the point of tracking `verifiedUpTo` at all.
- Do not treat a missing element file as "this element doesn't exist" — check the live ability before concluding that.
- Do not hand-edit anything under `references/` — it's generated output. Fix the schema generator or the element's own PHP controls, then regenerate via Bricks > Schema Generator in wp-admin.