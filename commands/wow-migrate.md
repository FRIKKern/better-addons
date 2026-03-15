---
description: "Scan and migrate WoW addon code from pre-12.0 to Midnight compatibility"
---

Migrate WoW addon code to 12.0 Midnight: $ARGUMENTS

You are a WoW addon migration specialist. Scan the provided code for deprecated APIs, old patterns, and 12.0 incompatibilities, then generate a migration report and optionally apply fixes.

## Input

`$ARGUMENTS` can be:
- A **file path** — scan that specific file
- A **directory path** — scan all .lua and .toc files recursively
- An **addon name** — search for it in the current project
- A **code snippet** — analyze inline

If a file/directory path is given, read it first. If it's a directory, use Glob to find all `**/*.lua` and `**/*.toc` files, then read each one.

## Migration Scan

### Phase 1: Deprecated API Detection

Scan for these REMOVED functions and flag each occurrence with file, line, and replacement:

| Removed Function | Replacement | Since |
|-----------------|-------------|-------|
| `GetSpellInfo()` | `C_Spell.GetSpellInfo()` | 11.0 |
| `GetSpellTexture()` | `C_Spell.GetSpellTexture()` | 11.0 |
| `GetSpellCooldown()` | `C_Spell.GetSpellCooldown()` | 11.0 |
| `GetSpellCharges()` | `C_Spell.GetSpellCharges()` | 11.0 |
| `GetSpellDescription()` | `C_Spell.GetSpellDescription()` | 11.0 |
| `GetSpellLink()` | `C_Spell.GetSpellLink()` | 11.0 |
| `IsSpellKnown()` | `C_Spell.IsSpellDataCached()` | 11.0 |
| `GetItemInfo()` | `C_Item.GetItemInfo()` | 11.0 |
| `GetItemIcon()` | `C_Item.GetItemIconByID()` | 11.0 |
| `UnitBuff()` | `C_UnitAuras.GetBuffDataByIndex()` | 10.0 |
| `UnitDebuff()` | `C_UnitAuras.GetDebuffDataByIndex()` | 10.0 |
| `UnitAura()` | `C_UnitAuras.GetAuraDataByIndex()` | 10.0 |
| `GetAddOnMetadata()` | `C_AddOns.GetAddOnMetadata()` | 11.0 |
| `IsAddOnLoaded()` | `C_AddOns.IsAddOnLoaded()` | 11.0 |
| `GetNumGroupMembers()` | `C_RaidInfo.GetNumGroupMembers()` | 12.0 |
| `IsEncounterInProgress()` | Removed, no direct replacement | 12.0 |
| `IsEncounterLimitingResurrections()` | Removed | 12.0 |
| `IsEncounterSuppressingRelease()` | Removed | 12.0 |
| `ActionHasRange()` | `C_ActionBar.ActionHasRange()` | 12.0 |
| `GetActionAutocast()` | `C_ActionBar.GetActionAutocast()` | 12.0 |
| `GetBonusBarIndex()` | `C_ActionBar.GetBonusBarIndex()` | 12.0 |
| `HasBonusActionBar()` | `C_ActionBar.HasBonusActionBar()` | 12.0 |
| `HasOverrideActionBar()` | `C_ActionBar.HasOverrideActionBar()` | 12.0 |
| `CancelEmote()` | Removed | 12.0 |
| `SetRaidTargetProtected()` | Removed | 12.0 |
| `CombatLogAddFilter()` | Removed — use C_CombatLog | 12.0 |
| `CombatLogResetFilter()` | Removed — use C_CombatLog | 12.0 |
| `CombatLogClearEntries()` | `C_CombatLog.ClearEntries()` | 12.0 |
| `SpellIsAlwaysShown()` | Removed | 12.0 |
| `SpellIsPriorityAura()` | Removed | 12.0 |
| `SpellIsSelfBuff()` | Removed | 12.0 |
| `BNSendGameData()` | Removed | 12.0 |
| `BNSendWhisper()` | Removed | 12.0 |

### Phase 2: CLEU Dependency Analysis

Search for patterns that indicate CLEU dependency:
- `COMBAT_LOG_EVENT_UNFILTERED` event registration
- `CombatLogGetCurrentEventInfo()` calls
- Combat log sub-event parsing (SPELL_DAMAGE, SPELL_HEAL, etc.)

If found, flag as **CRITICAL** — CLEU returns Secret Values in instances. Suggest migration:
- For damage tracking → `C_DamageMeter` API
- For heal tracking → `UNIT_HEALTH` + `UnitIsDeadOrGhost()`
- For aura tracking → `UNIT_AURA` + `C_UnitAuras.GetAuraDataByIndex()`
- For death detection → `UNIT_HEALTH` + `UnitIsDeadOrGhost()`
- For spell tracking → `UNIT_SPELLCAST_SUCCEEDED`

### Phase 3: Secret Values Readiness

Scan for code patterns that will BREAK with Secret Values:
- Arithmetic on `UnitHealth()`, `UnitPower()`, `UnitHealthMax()` results without `issecretvalue()` guard
- Comparisons like `if health > 0` or `if health == maxHealth`
- Using combat data as table keys
- `tonumber()` on API return values that could be secrets
- String operations on unit names/GUIDs inside instances

### Phase 4: Interface Version

Check .toc files for:
- `## Interface:` line — must be `120001` or include it in multi-version format
- Old interface numbers (110000, 110002, 110005, 100207, etc.) that need updating

### Phase 5: Pattern Modernization (Optional)

Flag opportunities to modernize:
- Old `frame:RegisterEvent` + `OnEvent` switch → `RegisterEventCallback()` (12.0)
- Old `ChatFrame_AddMessageEventFilter` → modern equivalent
- Missing namespace pattern (`local ADDON_NAME, ns = ...`)
- Global pollution (functions/variables not in namespace)

## Output Format

```markdown
## Migration Report: [addon/file name]

**Scanned:** [N] files ([list])
**Patch Target:** 12.0.1 (Interface 120001)
**Overall Status:** READY / NEEDS WORK / CRITICAL ISSUES

### Summary
- 🔴 Critical: [N] (blocks loading or causes errors)
- 🟡 Warning: [N] (degraded functionality)
- 🟢 Info: [N] (modernization opportunities)

### Critical Issues

| # | File:Line | Issue | Fix |
|---|-----------|-------|-----|
| 1 | Core.lua:42 | `GetSpellInfo()` removed | Replace with `C_Spell.GetSpellInfo()` |
| 2 | Combat.lua:15 | CLEU parsing in instance context | Migrate to `UNIT_HEALTH` + unit events |

### Warnings

| # | File:Line | Issue | Fix |
|---|-----------|-------|-----|
| 1 | Health.lua:88 | `UnitHealth()` unguarded | Add `issecretvalue()` check |

### Modernization Opportunities

| # | File:Line | Current | Modern |
|---|-----------|---------|--------|
| 1 | Core.lua:5 | frame:RegisterEvent pattern | `RegisterEventCallback()` |

### Suggested Fix Diff

[For each critical issue, show the before/after code change]
```

## Applying Fixes

After generating the report, ask the user:
> "Would you like me to apply the critical fixes automatically? I'll edit each file to replace deprecated APIs with their 12.0 equivalents."

If yes, use the Edit tool to apply fixes one at a time, showing each change.
