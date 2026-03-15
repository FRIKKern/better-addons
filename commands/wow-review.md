Review WoW addon code: $ARGUMENTS

**Development Mode:** Before reviewing, read `.claude/modes/active-mode.md` to determine the active mode (default: `enhancement-artist`), then read `.claude/modes/{mode-name}.md` for mode-specific review criteria. The mode affects which patterns are considered correct vs. violations.

If the user includes a mode keyword in $ARGUMENTS (e.g., "review as **boundary-pusher**"), use that mode instead of the active mode.

You are a WoW Midnight (12.0.1) addon code reviewer. Perform a thorough review of the specified file or directory, checking for deprecated APIs, taint risks, Secret Values violations, pattern issues, and Lua 5.1 compliance.

## Input

`$ARGUMENTS` is a file path or directory. Read the file(s) first.

If a directory, use Glob to find all `**/*.lua`, `**/*.toc`, and `**/*.xml` files, then review each.

## Review Categories

### 1. API Correctness (Severity: CRITICAL)

Check every WoW API call:
- Does the function exist in 12.0.1?
- Is the function signature correct (parameter count, types)?
- Is a deprecated function being used instead of its replacement?
- Are there hallucinated functions that don't exist in WoW at all?

Common hallucinated functions to flag:
- `C_Spell.GetSpellName()` — does NOT exist
- `C_Spell.GetSpellIcon()` — does NOT exist, use `C_Spell.GetSpellTexture()`
- `C_UnitAuras.GetAuraBySpellID()` — does NOT exist
- `C_Item.GetItemName()` — does NOT exist
- `C_Timer.SetTimeout()` — does NOT exist, use `C_Timer.After()`
- `RegisterEvent()` as a global — does NOT exist

### 2. Taint Risks (Severity: HIGH)

Check for common taint issues:
- Modifying Blizzard global tables or frames during combat
- Writing to `_G` directly
- Using `rawset` on protected tables
- Calling protected functions from insecure code during combat
- Missing `issecurevariable()` checks before modifying secure state
- `hooksecurefunc` on forbidden functions (`getfenv`, `rawset`, `select`, `setfenv`, `pcall`, `type`, `unpack`, `next`)

### 3. Secret Values Compliance (Severity: HIGH)

If the addon processes combat data:
- Are `UnitHealth()` / `UnitPower()` / `UnitHealthMax()` results guarded with `issecretvalue()`?
- Is there arithmetic on potentially-secret values without guards?
- Are combat log values compared or used as table keys?
- Is `tonumber()` called on API results that could be secrets?
- Are unit names/GUIDs used as table keys inside instances?
- Does the code use `C_RestrictedActions.IsAddOnRestrictionActive()` to detect restricted contexts?
- Does the code gracefully degrade when values are secrets?

### 4. Pattern Violations (Severity: MEDIUM)

Check for WoW addon best practices:
- **Namespace pattern**: Uses `local ADDON_NAME, ns = ...`? No bare globals?
- **Event dispatch**: Uses table-based dispatch? Events unregistered when no longer needed?
- **ADDON_LOADED**: Properly checks `addonName == ADDON_NAME`? Unregisters after init?
- **SavedVariables**: Initialized with defaults in ADDON_LOADED? Uses `or {}` pattern?
- **Slash commands**: Registered properly with `SLASH_` prefix convention?
- **Frame creation**: Uses `CreateFrame()` with proper parent? Not creating frames in tight loops?
- **Table reuse**: Uses `wipe()` instead of creating new tables? Reuses temp tables?
- **Global leaks**: Any unintended globals? (variables without `local`)

### 5. Lua 5.1 Compliance (Severity: MEDIUM)

WoW uses Lua 5.1 (with select extensions). Flag:
- `goto` statements (5.2+)
- `#` on non-sequence tables for length (unreliable in 5.1)
- Bit operations with `~` operator (5.3+) — use `bit.bxor`
- Integer division `//` (5.3+) — use `math.floor(a/b)`
- `table.move` (5.3+)
- `table.pack` / `table.unpack` (5.2+) — use `unpack` (global in WoW)
- `string.format` with `%s` on nil (errors in WoW's Lua)
- `load()` with string argument (5.2+) — WoW blocks this anyway
- `setfenv` / `getfenv` — exist in WoW Lua 5.1 but cannot be hooked since 11.0

### 6. Performance (Severity: LOW)

Flag performance anti-patterns:
- `string.format` or concatenation in `OnUpdate` handlers (creates garbage)
- Creating frames or tables in frequently-called functions
- Not throttling `OnUpdate` with elapsed time accumulator
- Using `pairs()` where `ipairs()` would suffice (ordered iteration)
- Global lookups in hot paths (should be localized: `local pairs = pairs`)
- Unbounded table growth without cleanup

### 7. TOC File (if present, Severity: CRITICAL)

- Interface version is `120001`
- SavedVariables match actual variable names in Lua
- File load order is correct (Init before Core, Core before Config)
- No missing files referenced

## Output Format

```markdown
## Code Review: [file/directory name]

**Reviewer:** WoW Addon Code Review (12.0.1)
**Files Reviewed:** [N]
**Overall Grade:** A / B / C / D / F

### Summary
| Severity | Count |
|----------|-------|
| 🔴 CRITICAL | [N] |
| 🟠 HIGH | [N] |
| 🟡 MEDIUM | [N] |
| 🔵 LOW | [N] |
| ✅ PASS | [N checks passed] |

### Findings

#### 🔴 CRITICAL

**[C1] Deprecated API Usage** — `Core.lua:42`
```lua
-- Current (BROKEN):
local name = GetSpellInfo(spellId)
-- Fixed:
local info = C_Spell.GetSpellInfo(spellId)
local name = info and info.name
```

#### 🟠 HIGH

**[H1] Unguarded Secret Value** — `Health.lua:88`
```lua
-- Current (ERRORS in instances):
if UnitHealth("target") > 0 then
-- Fixed:
local hp = UnitHealth("target")
if not issecretvalue(hp) and hp > 0 then
```

#### 🟡 MEDIUM

**[M1] Global Leak** — `Utils.lua:5`
```lua
-- Current:
function MyHelper() end
-- Fixed:
local function MyHelper() end
-- Or: ns.MyHelper = function() end
```

#### 🔵 LOW

**[L1] Performance** — `Core.lua:100`
OnUpdate handler without throttle.

### What's Good
- [Positive observations — correct patterns, good practices]
```
