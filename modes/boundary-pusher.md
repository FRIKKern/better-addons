# Mode: Boundary Pusher

**Philosophy:** "Push WoW's addon API to its absolute limits. If the top addons do it, we do it better."

This mode unlocks aggressive, advanced techniques used by ElvUI, BetterBags, Cell, Plater, and OmniCD. Every technique here is battle-tested in production addons with millions of downloads. You are building ElvUI/WeakAuras-class tools that push the envelope.

---

## ENABLED Techniques

1. **`getmetatable()` + `__index` hooks on Blizzard frame types** — The Cooldown_MT pattern. Hook every Cooldown frame ever created by intercepting the metatable's methods.
2. **Global Cooldown metatable hook pattern** — Apply behavior to ALL frames of a type without iterating instances.
3. **`hooksecurefunc` on ANY Blizzard global function** — Post-hook any function in the global namespace or on any frame object. No limits on what you hook.
4. **`frame:GetRegions()` to strip/modify Blizzard textures** — Iterate all regions, hide textures, replace artwork, reshape frames entirely.
5. **Noop pattern to block Blizzard texture re-application** — Override `SetNormalTexture`, `SetPushedTexture`, etc. with `function() end` to prevent Blizzard from undoing your skin.
6. **UIHider reparenting for hiding protected frames** — Create a hidden parent frame, reparent Blizzard frames to it instead of calling `:Hide()` (which would taint).
7. **`SetOverrideBinding()` for keybind interception** — Override keybinds on a per-frame basis without modifying the player's saved bindings.
8. **`GetFrameMetatable()` for frame-type-wide hooks** — Access the shared metatable for all frames to add/modify methods globally.
9. **Deferred flag pattern** — Set boolean flags in hooks/events, process them in the next `OnUpdate` tick to avoid re-entrancy and taint propagation.
10. **ScrollBox element factory hooks** — Hook `OnDataRangeChanged` and `ForEachFrame` on ScrollBox instances to skin or modify every list element Blizzard creates.
11. **Private Aura integration for custom audio triggers** — Attach custom sounds to Private Auras for boss-mod-style alerts within API constraints.
12. **`C_EncounterEvents` for boss mod features** — Use the new 12.0.1 namespace for custom event colors and sounds on boss mechanics.
13. **Taint isolation via parent frame trick** — BetterBags pattern: create an intermediate parent frame to isolate taint from propagating to secure children.
14. **Classification-based nameplate tricks** — Use mob type/classification (elite, rare, boss) to color and prioritize nameplates — the last remaining "OP" nameplate feature in Midnight.

---

## REQUIRED Safety Nets

Every aggressive technique MUST have these safeguards. No exceptions.

1. **Boundary comment** — Mark every aggressive hook with:
   ```lua
   -- BOUNDARY: May break in patch X.Y, fallback below
   ```

2. **`pcall()` wrapper around metatable access** — Metatables can change between patches. Always wrap:
   ```lua
   local ok, mt = pcall(getmetatable, someFrame)
   if not ok or not mt then return end
   ```

3. **Fallback path when hooked functions change signatures** — If arguments shift, degrade gracefully:
   ```lua
   hooksecurefunc(obj, "Method", function(self, arg1, ...)
       if type(arg1) ~= "number" then return end  -- signature changed, bail
       -- aggressive logic here
   end)
   ```

4. **Version check where possible** — Gate techniques behind build info:
   ```lua
   local _, _, _, tocVersion = GetBuildInfo()
   if tocVersion < 120001 then return end
   ```

---

## Code Examples

### Cooldown Metatable Hook

```lua
-- BOUNDARY: May break if Blizzard changes Cooldown metatable, fallback: per-frame hooks
local ok, cd = pcall(CreateFrame, "Cooldown", nil, nil, "CooldownFrameTemplate")
if ok and cd then
    local Cooldown_MT = getmetatable(cd).__index
    hooksecurefunc(Cooldown_MT, "SetCooldown", function(self, start, duration)
        if self:IsForbidden() or duration <= 1.5 then return end
        -- Add text overlay, glow, custom timer — affects ALL cooldown frames
    end)
end
```

### StripTextures + Noop Pattern

```lua
-- BOUNDARY: May break if Blizzard changes button texture API, fallback: skip skinning
local noop = function() end
local function SkinButton(button)
    for i = 1, button:GetNumRegions() do
        local region = select(i, button:GetRegions())
        if region and region:GetObjectType() == "Texture" then
            region:SetAlpha(0)
        end
    end
    button.SetNormalTexture = noop   -- Block Blizzard re-application
    button.SetPushedTexture = noop
end
```

### Taint Isolation Parent Frame Trick

```lua
-- BOUNDARY: BetterBags pattern. May break if Blizzard changes template parenting, fallback: direct creation
local function CreateIsolatedItemButton(name)
    local parent = CreateFrame("Button", name .. "Parent")  -- Taint stops here
    local button = CreateFrame("ItemButton", name, parent, "ContainerFrameItemButtonTemplate")
    return button, parent
end
```

### Deferred Flag Pattern

```lua
local pendingUpdate = false
hooksecurefunc(SomeBlizzardFrame, "SomeMethod", function()
    pendingUpdate = true  -- Don't do work here — just flag
end)
local elapsed_total = 0
frame:SetScript("OnUpdate", function(self, elapsed)
    if not pendingUpdate then return end
    elapsed_total = elapsed_total + elapsed
    if elapsed_total < 0.05 then return end  -- Debounce
    elapsed_total = 0
    pendingUpdate = false
    -- Safe to do expensive work here, outside the hook call stack
end)
```

---

## Reference Implementations

These addons pioneered each technique. Study their source code.

| Technique | Pioneered By | Source |
|---|---|---|
| Comprehensive UI replacement, frame skinning | **ElvUI** | [github.com/tukui-org/ElvUI](https://github.com/tukui-org/ElvUI) |
| Taint isolation, parent frame trick | **BetterBags** | [github.com/Cidan/BetterBags](https://github.com/Cidan/BetterBags) |
| ScrollBox hooks, Secret Values migration | **Cell** (PR #457) | [github.com/enderneko/Cell](https://github.com/enderneko/Cell/pull/457) |
| Nameplate metatable hooks, classification tricks | **Plater** | [github.com/Tercioo/Plater-Nameplates](https://github.com/Tercioo/Plater-Nameplates) |
| Cooldown tracking via frame hooks | **OmniCD** | CurseForge |
| Boss mod audio + Private Aura integration | **DBM** | [github.com/DeadlyBossMods/DeadlyBossMods](https://github.com/DeadlyBossMods/DeadlyBossMods) |
| Boss timeline enhancement | **BigWigs** | [github.com/BigWigsMods/BigWigs](https://github.com/BigWigsMods/BigWigs) |
| Nameplate classification coloring | **Platynator** | CurseForge |

---

## Anti-Patterns: Even for Boundary Pushers

These are lines even the most aggressive addons should NEVER cross.

1. **Direct memory/C-side access** — No `loadstring` tricks, no attempting to access the C API layer. Instant ban territory.
2. **Modifying SavedVariables of other addons** — Never write to another addon's `*DB` globals. Coordinate via addon messages or libraries.
3. **Hooking into Blizzard's internal event system directly** — Use `hooksecurefunc` and `HookScript`, never replace Blizzard's event dispatch or `SetScript` on frames you don't own.
4. **Breaking other addons' hooks** — Always chain, never replace. `hooksecurefunc` stacks; setting a metatable `__index` should extend, not override existing hooks.
5. **Bypassing `InCombatLockdown()` checks** — Even aggressive addons must respect combat lockdown. Queue actions for `PLAYER_REGEN_ENABLED`.
6. **Using `--no-verify` or bypass patterns to skip Blizzard security checks** — The secure execution environment exists for a reason. Work within it.
7. **Replacing `SetScript` on secure Blizzard frames** — Always `HookScript`, never `SetScript` on frames you don't own. Replacing scripts breaks other addons and can taint the frame.
8. **Caching secret values across restrictions** — Never store secret values in SavedVariables or attempt to extract their underlying data. They become `nil` on save.

---

## When to Use This Mode

**Activate** when building:
- Complete UI replacements (ElvUI-class)
- Nameplate addons (Plater/Platynator-class)
- Cooldown/ability trackers (OmniCD-class)
- Bag replacements (BetterBags-class)
- Raid frame addons (Cell-class)
- Boss mods (DBM/BigWigs-class)

**Do NOT activate** for:
- Simple utility addons (use blizzard-faithful mode)
- Housing addons (unrestricted APIs, no aggressive techniques needed)
- Data display addons (use standard patterns)
- First-time addon developers (start with the template, learn the fundamentals)

## Target Audience

Experienced addon developers building ElvUI/WeakAuras-class addons that push the envelope. You should already understand the taint system, combat lockdown, and WoW's Lua 5.1 sandbox before enabling this mode.
