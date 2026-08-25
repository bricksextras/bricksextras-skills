---
name: bricksextras-skills-update
description: "Use when the user asks to check for or install BricksExtras skills updates."
allowed-tools:
  - Bash
  - Read
---

**Requires:** BricksExtras skills installed from `https://github.com/bricksextras/bricksextras-skills`

# BricksExtras: check for skills updates

Use this skill when the user asks something like "check for bricksextras skills updates" or "are my bricksextras skills up to date".

## What it does

Compares the BricksExtras plugin version this schema bundle has been verified against to the version actually running on the site you're working on right now, and tells the user whether to re-pull the repo.

Note: the skills pack's own `version` field (in `references/index.json`) is an independent release number for the pack itself — it does **not** track the BricksExtras plugin version and must not be used for this comparison.

## Steps

1. **Read the installed pack's verified-compatible plugin version.** Open `references/index.json` in the `bricksextras-element-schemas` skill directory and read its `verifiedUpTo` field (e.g. `"1.7.3"`). This is the exact BricksExtras plugin version the schema bundle's content (elements, settings) has been confirmed accurate for — not a "works for anything up to this" range. A plugin release with no schema-relevant changes just gets this field bumped without touching the schema files.

2. **Read the live site's BricksExtras plugin version.** Use whatever ability the current MCP connection exposes for system/plugin information (e.g. `bricks/get-system-information` on the native Bricks MCP — read the `version` field of the `plugins` array entry named `"BricksExtras"`). If no MCP connection to a site is active, tell the user you need one to compare against and stop here.

3. **Compare the two version numbers using real version ordering, not string equality.** These are dot-separated version numbers that may carry a pre-release suffix (e.g. `1.7.10`, `1.8-beta2`) — compare numeric components (major.minor.patch) left to right, and treat a pre-release suffix as older than the same numbers without one. Do not compare the raw strings character-by-character (e.g. `"1.7.10" < "1.7.9"` is true as a string but wrong numerically).
   - **Live plugin version == `verifiedUpTo`:** tell the user their BricksExtras skills pack is verified current for the site's plugin version — nothing to do.
   - **Live plugin version > `verifiedUpTo`:** tell the user their skills pack hasn't been verified against this plugin version yet and may be missing settings/behavior for it (this is the same staleness `bricksextras-element-schemas` already falls back to live schema calls for, on a per-element basis — this check just surfaces it up front instead of waiting to hit it mid-task). Tell them to update by running:

     - If `~/.bricksextras/skills/bricksextras-skills` doesn't exist yet: the initial clone command from the repo's README.
     - If it exists and is on a normal branch (not detached HEAD): `git -C ~/.bricksextras/skills/bricksextras-skills pull`.
     - If it exists and is on a detached HEAD (i.e. was checked out at a specific release tag): the README's "checkout latest release tag" step instead — `git pull` will not move it forward from a tag. Check with `git -C ~/.bricksextras/skills/bricksextras-skills symbolic-ref -q HEAD` (empty/error output means detached HEAD).

     Either way, after updating, start a new chat/session — most clients load the skills list once at session start, so the refreshed content won't be picked up mid-session.
   - **Live plugin version < `verifiedUpTo`:** tell the user their skills pack was verified against a newer BricksExtras version than what's installed on this site. This isn't necessarily a problem (most settings are backward compatible), but flag it so they know why a setting the skill mentions might not exist yet on this site.

## Never do

- Do not run the update command on your own initiative just because a check found staleness — this skill's job is to report status. Only run it if the user explicitly asks you to update, not merely to check.
- Do not treat a version mismatch as blocking — tell the user and let them decide whether to update now or continue working.
