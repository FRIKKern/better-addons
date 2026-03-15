# better-addons

AI-powered WoW addon development toolkit for Claude Code, targeting Patch 12.0+ (Midnight).

## Install

```
/plugin marketplace add FRIKKern/better-addons
/plugin install better-addons
```

## Commands

| Command | Description |
|---------|-------------|
| `/better-addons:wow-create` | Create a complete addon from description |
| `/better-addons:wow-review` | Review addon code for issues |
| `/better-addons:wow-debug` | Diagnose addon problems |
| `/better-addons:wow-migrate` | Migrate pre-12.0 addons to Midnight |
| `/better-addons:wow-api` | Look up WoW API documentation |
| `/better-addons:wow-research` | Verify API claims against real sources |
| `/better-addons:wow-verify` | Fact-check WoW API usage |
| `/better-addons:wow-news` | Latest addon ecosystem news |
| `/better-addons:wow-mode` | Switch development philosophy |

## Development Modes

Switch how the AI writes code with `/better-addons:wow-mode`:

| Mode | Command | Philosophy |
|------|---------|-----------|
| Blizzard Faithful | `/better-addons:wow-mode faithful` | Official APIs only. Zero patch risk. |
| Boundary Pusher | `/better-addons:wow-mode boundary` | ElvUI-class. Metatables, aggressive hooks. |
| Enhancement Artist | `/better-addons:wow-mode enhance` | Skin & extend Blizzard UI. (Default) |
| Performance Zealot | `/better-addons:wow-mode performance` | Zero waste. Object pools. Event-driven. |

## Agents

9 specialized agents that delegate to each other:

- **Generalist** — Entry point, routes to specialists
- **Coder** — Writes complete addon code
- **Reviewer** — Code review (read-only)
- **Scaffold** — Project scaffolding
- **Skin Designer** — Enhancement/skinning addons
- **Researcher** — API documentation research
- **Debugger** — Diagnose addon issues
- **Migrator** — Pre-12.0 to Midnight migration
- **News Desk** — Latest addon news

## Documentation

Full documentation at [better-addons.com](https://better-addons.com)

## License

MIT
