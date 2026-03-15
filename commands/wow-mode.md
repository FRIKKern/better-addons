---
description: "Set the active development mode (Blizzard Faithful, Boundary Pusher, Enhancement Artist, Performance Zealot)"
---

Set WoW addon development mode: $ARGUMENTS

> **Plugin context:** This command is part of the better-addons plugin. Mode definitions (blizzard-faithful.md, etc.) are in the plugin's own `modes/` directory. The active mode state (`active-mode.md`) is stored in the project's `.claude/modes/` directory.

You manage the addon development mode for this project. The mode system fundamentally changes how all WoW addon agents write, review, and scaffold code.

## Available Modes

| Mode | Aliases | Philosophy |
|------|---------|-----------|
| `blizzard-faithful` | faithful, blizzard, safe, conservative | Official APIs only. No Blizzard frame hooks. Patch-proof. |
| `boundary-pusher` | boundary, pusher, aggressive, advanced, elvui | Undocumented APIs, metatable hooks, creative workarounds. ElvUI-class. |
| `enhancement-artist` | enhance, artist, better, skin, hook | hooksecurefunc, Mixin, frame skinning. "Enhance don't replace." DEFAULT. |
| `performance-zealot` | performance, perf, zealot, fast, lean | Minimal memory, throttled updates, object pooling, event-driven. |

## Process

1. Parse `$ARGUMENTS` for a mode name or alias
2. If no argument or "status": Read the project's `.claude/modes/active-mode.md` and display current mode with full description
3. If valid mode name or alias:
   - Resolve alias to canonical name (e.g., "safe" → "blizzard-faithful")
   - If the project's `.claude/modes/` directory doesn't exist, create it before writing
   - Write ONLY the canonical mode name to the project's `.claude/modes/active-mode.md` (overwrite entire file)
   - Read the mode definition from the plugin's `modes/{canonical-name}.md` directory, and display a summary of the mode's philosophy and key rules
4. If invalid argument: Show available modes and aliases

**Path resolution:**
- **Mode definitions** (read-only): Look in this plugin's `modes/` directory (e.g., `modes/blizzard-faithful.md`)
- **Active mode state** (read/write): Always use the project's `.claude/modes/active-mode.md`

## Output Format

When setting a mode, display:
- Mode name and icon
- 3-5 key philosophy points
- "All /wow-create, /wow-review, and agent interactions will now follow {mode} rules."

When showing status, also show brief descriptions of all 4 modes so the user knows their options.
