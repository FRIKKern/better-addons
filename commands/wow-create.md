---
description: "Create a complete, ready-to-install WoW addon for Midnight 12.0+ from a description"
---

Create a WoW addon: $ARGUMENTS

**Development Mode:** Before creating the addon, read the project's `.claude/modes/active-mode.md` to determine the active mode (default: `enhancement-artist`), then read the mode definition from this plugin's `modes/{mode-name}.md` for mode-specific rules. If the project's `.claude/modes/` directory doesn't exist, use the default mode. Pass the mode context to the Coder agent. The mode affects which patterns, APIs, and techniques are used in the generated code.

If the user includes a mode keyword in $ARGUMENTS (e.g., "create a **faithful** damage meter" or "create a **boundary** nameplate addon"), use that mode instead of the active mode. Keywords: faithful/blizzard/safe, boundary/aggressive/advanced, enhance/better/skin, performance/perf/fast.

You are a WoW Midnight (12.0.1) addon generator. Create a complete, ready-to-install addon based on the user's description.

## Process

### Step 1: Understand the Request

Parse `$ARGUMENTS` for:
- **Core functionality** — What does the addon do?
- **Addon name** — Extract or generate a PascalCase name (e.g., "RareTracker", "HousingHelper")
- **UI needs** — Does it need frames, buttons, minimap icons, slash commands?
- **Data needs** — Does it need SavedVariables, combat data, unit info, housing APIs?
- **Combat relevance** — Will it operate during encounters? (Secret Values implications)

### Step 2: Read the Template

Read these template files for patterns:
- `template/MyAddon.toc` — TOC format and metadata
- `template/Init.lua` — Namespace and initialization pattern
- `template/Core.lua` — Event dispatch and main logic
- `template/Config.lua` — Settings and SavedVariables
- `template/CLAUDE.md` — API reference and conventions

### Step 3: Research Required APIs

Use the WoW Addon Researcher agent to verify every API you plan to use:
- Confirm functions exist in 12.0.1
- Get correct signatures
- Check for Secret Values implications
- Find real-world usage patterns

### Step 4: Generate the Addon

Create all files in a new directory named after the addon. Follow these MANDATORY patterns:

**TOC File Requirements:**
```toc
## Interface: 120001
## Title: AddonName
## Notes: Description
## Author: [user]
## Version: 1.0.0
## SavedVariables: AddonNameDB
## IconTexture: Interface\Icons\[appropriate-icon]

Init.lua
Core.lua
Config.lua
```

**Init.lua — Namespace Pattern (MANDATORY):**
```lua
local ADDON_NAME, ns = ...
ns.ADDON_NAME = ADDON_NAME

-- Shared state
ns.db = {}
ns.isLoaded = false
```

**Core.lua — Event Dispatch (MANDATORY):**
```lua
local ADDON_NAME, ns = ...

local frame = CreateFrame("Frame")
local events = {}

function events:ADDON_LOADED(addonName)
    if addonName ~= ADDON_NAME then return end
    -- Initialize SavedVariables
    AddonNameDB = AddonNameDB or {}
    ns.db = AddonNameDB
    ns.isLoaded = true
    frame:UnregisterEvent("ADDON_LOADED")
end

-- Register and dispatch
for event in pairs(events) do
    frame:RegisterEvent(event)
end
frame:SetScript("OnEvent", function(self, event, ...)
    if events[event] then
        events[event](self, ...)
    end
end)
```

**Combat Data — Secret Values Guard (if applicable):**
```lua
local function SafeGetHealth(unit)
    local health = UnitHealth(unit)
    if issecretvalue(health) then
        return nil -- Cannot process, degrade gracefully
    end
    return health
end
```

**Communication Lockdown (if using addon comms):**
```lua
local pendingMessages = {}

local function SafeSendMessage(prefix, text, channel, target)
    if C_ChatInfo.InChatMessagingLockdown() then
        table.insert(pendingMessages, {prefix, text, channel, target})
        return false
    end
    C_ChatInfo.SendAddonMessage(prefix, text, channel, target)
    return true
end

-- Flush on encounter end
function events:ENCOUNTER_END()
    for _, msg in ipairs(pendingMessages) do
        C_ChatInfo.SendAddonMessage(unpack(msg))
    end
    wipe(pendingMessages)
end
```

### Step 5: Verify Generated Code

After generating all files, run verification:
1. Check every API call against the deprecated list
2. Ensure no hallucinated functions (C_Spell.GetSpellName, C_Timer.SetTimeout, etc.)
3. Verify Secret Values handling if combat-relevant
4. Confirm Interface version is 120001
5. Check namespace pattern is used (no bare globals)
6. Verify event dispatch table pattern

### Step 6: Output

Write all files to a directory. Provide a summary:

```markdown
## Addon Created: [Name]

**Files:**
- `[Name]/[Name].toc` — Interface 120001
- `[Name]/Init.lua` — Namespace setup
- `[Name]/Core.lua` — Main logic
- `[Name]/Config.lua` — Settings (if needed)

**Features:**
- [bullet list of what the addon does]

**APIs Used:**
- [list of verified WoW APIs with wiki links]

**Installation:**
Copy the `[Name]` folder to `World of Warcraft/_retail_/Interface/AddOns/`

**Secret Values:** [Yes/No — does it handle combat data?]
**Addon Comms:** [Yes/No — does it communicate?]
```

## CRITICAL RULES

- NEVER use deprecated functions. Always use the C_ namespace replacements.
- NEVER hallucinate API functions. If unsure, research first.
- ALWAYS use the namespace pattern (`local ADDON_NAME, ns = ...`)
- ALWAYS use the event dispatch table pattern
- ALWAYS guard combat data with `issecretvalue()` if the addon operates during encounters
- ALWAYS set Interface to `120001`
- NEVER use globals without declaring them in the TOC or a shared namespace
- PREFER `RegisterEventCallback()` for simple event handling (new in 12.0)
