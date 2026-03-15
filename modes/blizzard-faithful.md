# Mode: Blizzard Faithful

**Philosophy:** Official APIs only. If Blizzard didn't document it, we don't use it. Zero patch-day risk.

This mode constrains ALL code generation to documented, stable WoW APIs. Every pattern used here survives patch days because it relies only on Blizzard's public API contract. When in doubt, leave it out.

---

## MANDATORY Rules

1. **ONLY use APIs documented on warcraft.wiki.gg** — if it's not on the wiki, it doesn't exist to us.
2. **NO `hooksecurefunc` on Blizzard frames** — only hook your own frames and functions. Never hook Blizzard methods or global Blizzard functions.
3. **NO metatable manipulation of Blizzard objects** — never call `getmetatable()` on any frame you didn't create. No metatable hooks on Cooldown, Button, or any Blizzard widget class.
4. **NO `frame:GetRegions()` on Blizzard frames** — never iterate or modify regions belonging to frames you didn't create.
5. **NO noop pattern** — never overwrite Blizzard methods (e.g., `frame.SetNormalTexture = function() end`). This breaks internal Blizzard code that calls these methods.
6. **NO UIHider reparenting** — never reparent Blizzard frames to a hidden frame to hide them. This disrupts Blizzard's frame lifecycle and event handling.
7. **NO accessing undocumented Blizzard frame children/properties** — never reference `.NineSlice`, `.Background`, `.Border`, or other internal frame children. These change without notice between patches.
8. **ALWAYS use Settings API** — use `Settings.RegisterAddOnCategory()` and `Settings.CreateCheckbox()` / `Settings.CreateSlider()` for all configuration UI.
9. **ALWAYS use Addon Compartment** for minimap button presence — declare `AddonCompartmentFunc` in the TOC file. No LibDBIcon, no custom minimap button frames.
10. **ALWAYS handle Secret Values** with full graceful degradation — check `issecretvalue()` before any math or comparison on values that could be secret during combat/encounters.
11. **ALWAYS check `InCombatLockdown()`** before any frame operations that touch size, position, visibility, or attributes on secure frames. Queue actions for `PLAYER_REGEN_ENABLED`.
12. **PREFER `EventUtil.ContinueOnAddOnLoaded()`** for initialization over manually registering and filtering `ADDON_LOADED`.
13. **PREFER `CreateFrame` with official Blizzard templates** — use `"BackdropTemplate"`, `"SecureActionButtonTemplate"`, `"CooldownFrameTemplate"`, etc. Don't build from scratch what Blizzard provides.
14. **Code style:** verbose, well-commented, defensive error handling. Every non-obvious block gets a comment explaining *why*.
15. **Add header comment to every file:** `-- Mode: Blizzard Faithful | Safe across all patches`

---

## Anti-Patterns (Hard Reject)

These patterns MUST be rejected. Do not generate code that uses any of them.

| Pattern | Why It's Rejected |
|---|---|
| `getmetatable()` on any frame you didn't create | Blizzard changes metatable structure between patches. Hooks on class metatables break silently when internal method signatures change. |
| `hooksecurefunc()` on Blizzard UI functions | Post-hooks on Blizzard functions couple your addon to Blizzard's internal call patterns. When they refactor, your hook breaks or fires with unexpected arguments. |
| Accessing `.NineSlice`, `.Background`, etc. on Blizzard frames | These are undocumented internal frame children. They get renamed, removed, or restructured every major patch. |
| `SetScript()` on any frame you didn't create | Overwrites Blizzard's own script handlers, breaking their UI logic. Even `HookScript` on Blizzard frames is off-limits in this mode. |
| `frame:GetRegions()` on Blizzard frames | Region order and count change between patches. Code that assumes specific region indices will break. |
| Reading Blizzard source code for "undocumented" APIs | Any API discovered by reading Blizzard Lua source rather than the wiki is considered unstable and forbidden. |
| `LoadAddOn()` to force-load Blizzard UI modules | Blizzard controls when their modules load. Forcing them can cause initialization order bugs. Wait for `ADDON_LOADED` instead. |
| Direct frame parenting to Blizzard frames | Parent your frames to `UIParent`, not to Blizzard frames. Blizzard frames get destroyed and recreated across patches. |

---

## Recommended Patterns

These are the safe, documented, stable patterns. Use these exclusively.

### Initialization

```lua
-- Mode: Blizzard Faithful | Safe across all patches
local addonName, ns = ...

EventUtil.ContinueOnAddOnLoaded(addonName, function()
    ns.db = MyAddonDB or CopyTable(ns.defaults)
    MyAddonDB = ns.db
end)
```

### Settings Panel

```lua
local category, layout = Settings.RegisterVerticalLayoutCategory(addonName)
ns.categoryID = category:GetID()

local setting = Settings.RegisterAddOnSetting(category, "enabled", "enabled",
    ns.db, type(true), "Enable Addon", true)
Settings.CreateCheckbox(category, setting, "Toggle the addon on or off.")

Settings.RegisterAddOnCategory(category)
```

### Addon Compartment (Minimap Presence)

In the TOC:
```
## AddonCompartmentFunc: MyAddon_OnAddonCompartmentClick
## AddonCompartmentFuncOnEnter: MyAddon_OnAddonCompartmentEnter
## AddonCompartmentFuncOnLeave: MyAddon_OnAddonCompartmentLeave
```

In Lua:
```lua
function MyAddon_OnAddonCompartmentClick()
    Settings.OpenToCategory(ns.categoryID)
end
```

### Timers and Delays

```lua
C_Timer.After(1.0, function()
    -- Runs once after 1 second
end)

local ticker = C_Timer.NewTicker(5.0, function()
    -- Runs every 5 seconds
end)
-- ticker:Cancel() to stop
```

### Frame Creation

```lua
local frame = CreateFrame("Frame", "MyAddonFrame", UIParent, "BackdropTemplate")
frame:SetSize(200, 100)
frame:SetPoint("CENTER")
frame:SetBackdrop({
    bgFile = "Interface\\Buttons\\WHITE8x8",
    edgeFile = "Interface\\Buttons\\WHITE8x8",
    edgeSize = 1,
})
frame:SetBackdropColor(0.1, 0.1, 0.1, 0.9)
```

### Event Handling (Dispatch Table)

```lua
local events = {}

function events:PLAYER_LOGIN()
    -- Safe to call most APIs here
end

function events:PLAYER_REGEN_ENABLED()
    -- Execute queued combat-deferred actions
end

local eventFrame = CreateFrame("Frame")
eventFrame:SetScript("OnEvent", function(self, event, ...)
    local handler = events[event]
    if handler then handler(self, ...) end
end)
for event in pairs(events) do
    eventFrame:RegisterEvent(event)
end
```

### Secret Values (Graceful Degradation)

```lua
local health = UnitHealth("target")
if issecretvalue(health) then
    -- Widget-safe path: pass directly to StatusBar
    healthBar:SetValue(UnitHealthPercent("target"))
else
    -- Full access: can do math and comparisons
    local pct = health / UnitHealthMax("target")
    healthBar:SetValue(pct)
end
```

### Combat Lockdown Safety

```lua
local function SafeShow(frame)
    if InCombatLockdown() then
        table.insert(ns.pendingActions, function() frame:Show() end)
    else
        frame:Show()
    end
end
```

---

## Target Audience

- **Conservative addon developers** who want zero risk of breakage on patch day
- **First-time addon developers** learning the correct patterns from the start
- **Guild and corporate tools** that must never break in production
- **Addons for competitive environments** where reliability is non-negotiable

This mode produces addons that are boring, safe, and bulletproof. They won't win style points, but they will work on day one of every patch.
