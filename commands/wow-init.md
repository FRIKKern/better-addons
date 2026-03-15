---
description: "Initialize a WoW addon project directory for use with the better-addons plugin"
---

Initialize a WoW addon project for use with better-addons: $ARGUMENTS

You set up a WoW addon project directory to work with the better-addons Claude Code plugin.

## What This Does

1. Creates `.claude/modes/active-mode.md` with the default mode ("enhancement-artist")
2. Creates a `CLAUDE.md` file with WoW 12.0+ addon development context (if one doesn't exist)
3. Displays a welcome message with available commands and current mode

## Process

### Step 1: Set Up Mode

Check if `.claude/modes/active-mode.md` exists in the project root.

- **If not:** Create the `.claude/modes/` directory and write `enhancement-artist` to `active-mode.md`
- **If yes:** Read it and note the current mode

If `$ARGUMENTS` contains a recognized mode name (`blizzard-faithful`, `boundary-pusher`, `enhancement-artist`, `performance-zealot`, or their short aliases `faithful`, `boundary`, `enhance`, `performance`), use that as the active mode instead of the default.

### Step 2: Set Up CLAUDE.md

Check if `CLAUDE.md` exists in the project root.

- **If not:** Create a basic `CLAUDE.md` with:

```markdown
# Project: {project name from $ARGUMENTS or directory name}

This is a WoW addon project targeting **Patch 12.0+ (Midnight)**.

## Conventions

- **Language:** Lua 5.1 (WoW embedded runtime)
- **Interface version:** 120001
- **Expansion:** Midnight (12.0)
- **Secret Values:** All combat-related numeric data (health, damage, healing) may be opaque — never do arithmetic on potentially secret values
- **Security model:** Use `hooksecurefunc()` for post-hooks, check `IsForbidden()` on hooked frames, defer secure frame changes out of combat with `InCombatLockdown()` checks

## Plugin Commands

This project uses the `better-addons` Claude Code plugin. Available commands:

- `/better-addons:wow-create` — Create a complete addon from scratch
- `/better-addons:wow-review` — Review addon code for bugs and compatibility
- `/better-addons:wow-debug` — Debug addon issues
- `/better-addons:wow-mode` — Switch development philosophy mode
- `/better-addons:wow-migrate` — Migrate pre-12.0 code to Midnight
- `/better-addons:wow-api` — Look up WoW API documentation
- `/better-addons:wow-news` — Find latest WoW addon ecosystem news
- `/better-addons:wow-research` — Research and verify WoW addon information
- `/better-addons:wow-verify` — Verify addon code or API claims
- `/better-addons:wow-init` — Re-run this initialization
```

- **If yes:** Report that it already exists and skip creation

### Step 3: Display Welcome Message

Output the following welcome message (substitute the actual mode name):

```
better-addons initialized!

Current mode: {mode-name}

Available commands:
  /better-addons:wow-create   — Create a complete addon
  /better-addons:wow-review   — Review addon code
  /better-addons:wow-debug    — Debug addon issues
  /better-addons:wow-mode     — Switch development philosophy
  /better-addons:wow-migrate  — Migrate to Midnight 12.0
  /better-addons:wow-api      — Look up WoW API docs
  /better-addons:wow-news     — Latest addon ecosystem news
  /better-addons:wow-research — Research & verify information
  /better-addons:wow-verify   — Verify code or API claims

Change mode: /better-addons:wow-mode faithful|boundary|enhance|performance
```
