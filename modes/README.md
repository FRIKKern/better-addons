# WoW Addon Development Modes

The mode system controls the philosophy and coding style that all WoW addon agents
follow when creating, reviewing, or scaffolding addon code.

## Switching Modes

Use the `/wow-mode` slash command:

```
/wow-mode enhancement-artist    # Set mode by name
/wow-mode safe                  # Set mode by alias
/wow-mode status                # Show current mode
/wow-mode                       # Show current mode (same as status)
```

## Available Modes

### blizzard-faithful
Official APIs only. No Blizzard frame hooks. Maximum patch resilience.
Best for: addons that must survive every patch without maintenance.

### boundary-pusher
Undocumented APIs, metatable hooks, creative workarounds. ElvUI-class power.
Best for: UI overhauls, advanced combat addons, pushing what's possible.

### enhancement-artist (DEFAULT)
hooksecurefunc, Mixin extension, frame skinning. "Enhance don't replace."
Best for: addons that improve Blizzard UI without replacing it.

### performance-zealot
Minimal memory, throttled updates, object pooling, pure event-driven design.
Best for: addons running in raids, M+, or alongside many other addons.

## How Modes Work

The active mode is stored in `.claude/modes/active-mode.md` as a single line
containing the canonical mode name. When any WoW addon agent starts work, it
reads this file and applies the corresponding mode's rules from
`.claude/modes/{mode-name}.md`.

Each mode definition file contains:
- Philosophy and design principles
- Allowed and forbidden API patterns
- Code review criteria specific to that mode
- Example patterns to follow

## Default Mode

The default mode is **enhancement-artist**, which balances power with stability
by hooking and extending Blizzard frames rather than replacing them.
