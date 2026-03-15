# Mode: Performance Zealot

**Philosophy:** Every allocation counts. Zero waste. Your addon should be invisible to `/eventtrace`.

This mode prioritizes absolute minimum overhead. Every table creation, every global lookup, every unthrottled handler is a tax on the player's framerate. Code generated in this mode is lean, pre-allocated, and event-driven. If it can be cached, it is cached. If it can be pooled, it is pooled. If it can be avoided, it is avoided.

---

## MANDATORY Rules

1. **NEVER** create tables inside functions called more than once per second
2. **ALWAYS** cache globals as locals at file scope:
   ```lua
   local pairs, ipairs = pairs, ipairs
   local format = string.format
   local GetTime = GetTime
   local InCombatLockdown = InCombatLockdown
   ```
3. **ALWAYS** throttle OnUpdate with elapsed accumulator (minimum 0.1s interval)
4. **ALWAYS** use events over polling — register for specific events only
5. **ALWAYS** unregister events when data is no longer needed
6. **ALWAYS** use `wipe(t)` over `t = {}` for table reuse (avoids GC pressure)
7. **PREFER** `C_Timer.After()` over OnUpdate for one-shot delays
8. **PREFER** `BAG_UPDATE_DELAYED` over `BAG_UPDATE`
9. **PREFER** `RegisterUnitEvent("EVENT", "unit")` over `RegisterEvent("EVENT")`
10. **PREFER** a single centralized OnUpdate frame for the entire addon
11. **USE** object pooling for dynamically created/destroyed frames
12. **USE** pre-built format strings over runtime string concatenation
13. **AVOID** closures in hot paths — use upvalues instead
14. **AVOID** metatables in hot paths — use direct table access
15. **AVOID** string operations in OnUpdate — pre-compute all strings
16. **Add header comment:** `-- Mode: Performance Zealot | Zero-waste addon`

---

## Required Code Patterns

### Local Caching Block (file header)

Every file MUST begin with a caching block after the namespace line:

```lua
local addonName, ns = ...

-- Performance: cached globals
local pairs, ipairs, next = pairs, ipairs, next
local format, gsub = string.format, string.gsub
local wipe, tinsert, tremove = table.wipe, table.insert, table.remove
local GetTime = GetTime
local UnitName, UnitHealth, UnitHealthMax = UnitName, UnitHealth, UnitHealthMax
local InCombatLockdown = InCombatLockdown
local C_Timer_After = C_Timer.After
```

Cache every WoW API function used more than once in the file. Group by category.

### Throttled OnUpdate

The ONLY acceptable OnUpdate pattern:

```lua
local elapsed_acc = 0
local THROTTLE = 0.1

frame:SetScript("OnUpdate", function(self, elapsed)
    elapsed_acc = elapsed_acc + elapsed
    if elapsed_acc < THROTTLE then return end
    elapsed_acc = 0
    -- actual work here
end)
```

For multiple periodic tasks, use ONE frame with multiple accumulators:

```lua
local timers = { fast = 0, slow = 0 }
local FAST_INTERVAL, SLOW_INTERVAL = 0.1, 1.0

ns.tickFrame:SetScript("OnUpdate", function(self, elapsed)
    timers.fast = timers.fast + elapsed
    timers.slow = timers.slow + elapsed

    if timers.fast >= FAST_INTERVAL then
        timers.fast = 0
        ns:FastUpdate()
    end

    if timers.slow >= SLOW_INTERVAL then
        timers.slow = 0
        ns:SlowUpdate()
    end
end)
```

### Object Pool

REQUIRED for any addon that creates and destroys frames dynamically:

```lua
local pool = {}

local function Acquire()
    local obj = table.remove(pool)
    if not obj then
        obj = CreateFrame("Frame")
        -- one-time setup here
    end
    obj:Show()
    return obj
end

local function Release(obj)
    obj:Hide()
    obj:ClearAllPoints()
    pool[#pool + 1] = obj
end
```

### Event-Driven Architecture

Register only what you need. Unregister when you don't.

```lua
local frame = CreateFrame("Frame")
local events = {}

function events:PLAYER_ENTERING_WORLD(isLogin, isReload)
    -- Initialize only what's needed
end

function events:PLAYER_REGEN_DISABLED()
    -- Combat started: disable expensive non-essential features
    self:UnregisterEvent("BAG_UPDATE_DELAYED")
end

function events:PLAYER_REGEN_ENABLED()
    -- Combat ended: re-enable
    self:RegisterEvent("BAG_UPDATE_DELAYED")
end

frame:SetScript("OnEvent", function(self, event, ...)
    events[event](self, ...)
end)

for event in pairs(events) do
    frame:RegisterEvent(event)
end
```

### Table Reuse

Never allocate in a hot path. Pre-allocate and wipe:

```lua
-- Pre-allocate at file scope
local scratchPos = { x = 0, y = 0 }
local scratchUnits = {}

local function UpdatePositions()
    scratchPos.x, scratchPos.y = GetPlayerMapPosition()
    -- use scratchPos, never {x=..., y=...}
end

local function GatherUnits(maxCount)
    wipe(scratchUnits)
    for i = 1, maxCount do
        scratchUnits[i] = "raid" .. i
    end
    return scratchUnits  -- caller must NOT store a reference long-term
end
```

### String Pre-computation

```lua
-- Pre-build format strings at load time
local FMT_HEALTH = "%s: %.1f%%"
local FMT_DPS = "%.1f K"

-- In hot path, use format() with pre-built template
local function UpdateText(name, pct)
    label:SetText(format(FMT_HEALTH, name, pct))
end
```

---

## Measurement (Required in every addon)

Every Performance Zealot addon MUST include a `/perf` diagnostic:

```lua
SLASH_MYADDON_PERF1 = "/myaddonperf"
SlashCmdList["MYADDON_PERF"] = function()
    UpdateAddOnMemoryUsage()
    local mem = GetAddOnMemoryUsage("MyAddon")
    print(format("|cff00ff00MyAddon|r: %.1f KB | %d frames pooled",
        mem, #pool))
end
```

---

## Anti-Patterns (Hard Reject)

Any of these in generated code is an automatic rejection:

| Anti-Pattern | Why | Fix |
|---|---|---|
| `{}` or `{...}` inside any function called >1/sec | GC pressure, allocation stalls | `wipe()` + reuse pre-allocated table |
| `a .. b .. c` in OnUpdate | Creates intermediate strings every frame | `format()` with pre-built format string |
| Unthrottled OnUpdate handler | Burns CPU every frame (60-240+ Hz) | Elapsed accumulator, minimum 0.1s |
| Event handler without early-return for irrelevant units | Processes every unit change in range | Check `unit` arg first, return early |
| `pairs()` in hot paths with integer-keyed tables | Hash traversal is slower than array | `ipairs()` or numeric `for` loop |
| Creating closures `function() end` inside frequently-called functions | New closure object = new allocation | Use upvalues or pre-defined functions |
| `RegisterEvent` when `RegisterUnitEvent` would suffice | Fires for ALL units, not just yours | `RegisterUnitEvent("EVENT", "player")` |
| Multiple OnUpdate frames across files | Scattered tick sources, hard to profile | One centralized tick frame for entire addon |
| String keys in inner loops | Hash lookup per iteration | Numeric keys or direct field access |
| Unused event registrations left active | Handler fires for nothing | `UnregisterEvent` when feature is inactive |

---

## Target Audience

Addon developers building for:
- **Raid environments** (20+ players, dozens of auras, high event throughput)
- **Low-spec machines** (every KB and every ms matters)
- **Always-on addons** (damage meters, threat plates, nameplates, unit frames)
- **High-frequency data** (combat parsing, health tracking, nameplate updates)

---

## Quick Reference

```
DO:  wipe(t)              DON'T:  t = {}
DO:  local GetTime = GetTime     DON'T:  call GetTime as global in loop
DO:  format(FMT, ...)    DON'T:  a .. " " .. b .. " " .. c
DO:  RegisterUnitEvent    DON'T:  RegisterEvent for unit events
DO:  ipairs / numeric for DON'T:  pairs on sequential tables
DO:  pool:Acquire()       DON'T:  CreateFrame() in a loop
DO:  C_Timer.After()      DON'T:  OnUpdate for one-shot delay
DO:  elapsed accumulator  DON'T:  raw OnUpdate at frame rate
```
