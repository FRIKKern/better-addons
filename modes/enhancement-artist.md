# Mode: Enhancement Artist

**Philosophy:** "Skin, hook, extend — never replace. Make Blizzard's UI beautiful without breaking it."

This is the **DEFAULT** mode. It encodes the "Better Addon" philosophy that dominates the Midnight addon ecosystem.

---

## Core Principle

Enhance Blizzard's own UI elements rather than replacing them. Work WITH the default UI, not against it. Users keep their familiar layouts, Edit Mode settings, and muscle memory — you just make it look and work better.

Reference implementations: **BetterBlizzFrames**, **BetterBlizzPlates**, **ActionBarsEnhanced**, **Platynator**.

Target audience: Addon developers who love Blizzard's UI but want it polished. "BetterX" addon creators. UI enthusiasts.

---

## MANDATORY Rules

Every piece of code written in this mode MUST follow these rules:

1. **ALWAYS use `hooksecurefunc()`** — never pre-hook or override Blizzard functions
2. **ALWAYS check `IsForbidden()`** before modifying any hooked frame
3. **ALWAYS use recursion guards** when hooks trigger methods that fire hooks
4. **ALWAYS store original state** before modification (`ogPoint`, `ogParent`, `ogWidth`, etc.)
5. **ALWAYS defer operations out of combat** via the `RunAfterCombat` queue pattern
6. **ALWAYS use `HookScript()`** — never `SetScript()` on Blizzard frames
7. **USE `StripTextures()` + `ApplyBackdrop()`** for frame reskinning
8. **USE `Mixin()`** to extend Blizzard frame instances with your methods
9. **USE Edit Mode awareness** — hook `EventRegistry` for layout changes
10. **USE ScrollBox skinning** via `OnDataRangeChanged` callback
11. **SUPPORT Masque compatibility** for button-based addons
12. **Add header comment:** `-- Mode: Enhancement Artist | Enhance don't replace`

---

## FORBIDDEN Patterns

- `SetScript()` on any Blizzard frame — use `HookScript()` instead
- `:Hide()` on protected frames — use hidden frame reparenting
- Pre-hooking or overriding Blizzard functions — only post-hook
- Raw math on health/power values — they may be Secret Values
- `CreateFrame()` replacements for existing Blizzard frames
- Direct calls to `EditModeManagerFrame:RegisterSystemFrame()` — causes taint

---

## Standard Boilerplate Patterns

These reusable patterns form the foundation of every Enhancement Artist addon.

### Hidden Frame (for reparenting stripped textures)

```lua
local hiddenFrame = CreateFrame("Frame")
hiddenFrame:Hide()
```

### Combat Queue

```lua
local combatQueue = {}

local function RunAfterCombat(fn)
    if InCombatLockdown() then
        combatQueue[#combatQueue + 1] = fn
    else
        fn()
    end
end

-- Flush on combat end
local combatFrame = CreateFrame("Frame")
combatFrame:RegisterEvent("PLAYER_REGEN_ENABLED")
combatFrame:SetScript("OnEvent", function()
    for i = 1, #combatQueue do
        combatQueue[i]()
        combatQueue[i] = nil
    end
end)
```

### StripTextures

```lua
local function StripTextures(frame)
    if frame.GetRegions then
        for _, region in pairs({frame:GetRegions()}) do
            if region.SetTexture then
                region:SetParent(hiddenFrame)
            end
        end
    end
end
```

### ApplyBackdrop

```lua
local function ApplyBackdrop(frame, bgColor, borderColor)
    if not frame.backdropApplied then
        Mixin(frame, BackdropTemplateMixin)
        frame.backdropApplied = true
    end
    frame:SetBackdrop({
        bgFile = "Interface\\Buttons\\WHITE8x8",
        edgeFile = "Interface\\Buttons\\WHITE8x8",
        edgeSize = 1,
    })
    frame:SetBackdropColor(bgColor.r, bgColor.g, bgColor.b, bgColor.a or 0.9)
    frame:SetBackdropBorderColor(borderColor.r, borderColor.g, borderColor.b, borderColor.a or 1)
end
```

### Safe Hook Pattern

```lua
local isProcessing = false
hooksecurefunc(SomeFrame, "SomeMethod", function(self, ...)
    if isProcessing then return end
    if self.IsForbidden and self:IsForbidden() then return end
    isProcessing = true
    -- Your enhancement code here
    isProcessing = false
end)
```

### Original State Preservation

```lua
-- Save before modifying
frame.ogPoint = {frame:GetPoint()}
frame.ogParent = frame:GetParent()
frame.ogWidth = frame:GetWidth()
frame.ogHeight = frame:GetHeight()

-- Restore when needed
local function RestoreFrame(frame)
    if frame.ogParent then frame:SetParent(frame.ogParent) end
    if frame.ogPoint then frame:SetPoint(unpack(frame.ogPoint)) end
    if frame.ogWidth then frame:SetWidth(frame.ogWidth) end
    if frame.ogHeight then frame:SetHeight(frame.ogHeight) end
end
```

### Edit Mode Awareness

```lua
EventRegistry:RegisterCallback("EditMode.Enter", function()
    -- Temporarily disable position overrides
    ns:DisablePositionOverrides()
end)

EventRegistry:RegisterCallback("EditMode.Exit", function()
    -- Re-apply your modifications
    ns:EnablePositionOverrides()
end)
```

### Secret Values-Safe StatusBar Hook

```lua
hooksecurefunc(healthBar, "SetValue", function(self, value)
    if self:IsForbidden() then return end
    -- Apply class color — safe regardless of secret status
    local unit = self.unit
    if unit and UnitExists(unit) and UnitIsPlayer(unit) then
        local _, class = UnitClass(unit)
        local color = RAID_CLASS_COLORS[class]
        if color then
            self:SetStatusBarColor(color.r, color.g, color.b)
        end
    end
    -- DON'T do math on 'value' — it might be secret
end)
```

---

## Code Review Criteria

When reviewing code in this mode, check for:

1. Any use of `SetScript()` on Blizzard frames (must be `HookScript()`)
2. Any direct `:Hide()` calls on protected frames (must reparent)
3. Missing `IsForbidden()` checks in hooks
4. Missing recursion guards where hooks call hooked methods
5. Missing combat lockdown checks before secure frame modifications
6. Arithmetic on potentially secret health/power values
7. Missing original state preservation before modifications
8. Missing Edit Mode awareness for repositioned frames
9. Missing `-- Mode: Enhancement Artist | Enhance don't replace` header

---

## Reference Files

| Topic | File |
|-------|------|
| Skin Designer agent | `.claude/agents/wow-addon-skin-designer.md` |
| Better addon ecosystem | `docs-site/docs/better-addons.md` |
| Midnight coding patterns | `docs-site/docs/midnight-patterns.md` |
| Security model | `docs-site/docs/security.md` |
| Frames & widgets | `docs-site/docs/frames-widgets.md` |
| Code templates | `docs-site/docs/code-templates.md` |
