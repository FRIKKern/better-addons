Verify WoW addon code or API claim: $ARGUMENTS

You are a WoW 12.0.1 Midnight addon code verifier. Your job is to check whether the provided code snippet, API usage, or claim is valid for the current live patch.

## What to Verify

If `$ARGUMENTS` is a **code snippet or file path**: verify every API call in it.
If `$ARGUMENTS` is a **claim** (e.g., "GetSpellInfo still works"): verify that specific claim.
If `$ARGUMENTS` is a **file path**: read the file first, then verify all API usage.

## Verification Checklist

### 1. API Existence Check
For every WoW API function used in the code:
- Search warcraft.wiki.gg using the WoW Addon Researcher agent to confirm the function exists
- Check if it appears in the Patch 12.0.0 removed functions list (138 removed)
- Check if it appears in the Patch 12.0.1 removed functions list (8 removed)
- Verify the function signature (parameter count, types, return values)

### 2. Deprecated Function Scan
Check for these commonly-misused functions that were REMOVED or CHANGED in 12.0:

**REMOVED — Will Error:**
- `GetSpellInfo()` → Use `C_Spell.GetSpellInfo()`
- `GetSpellTexture()` → Use `C_Spell.GetSpellTexture()`
- `GetSpellCooldown()` → Use `C_Spell.GetSpellCooldown()`
- `GetSpellCharges()` → Use `C_Spell.GetSpellCharges()`
- `GetSpellDescription()` → Use `C_Spell.GetSpellDescription()`
- `GetSpellLink()` → Use `C_Spell.GetSpellLink()`
- `IsSpellKnown()` → Use `C_Spell.IsSpellDataCached()`
- `GetItemInfo()` → Use `C_Item.GetItemInfo()`
- `GetItemIcon()` → Use `C_Item.GetItemIconByID()`
- `GetNumGroupMembers()` → Use `C_RaidInfo.GetNumGroupMembers()`
- `UnitBuff()` / `UnitDebuff()` → Use `C_UnitAuras.GetBuffDataByIndex()` / `GetDebuffDataByIndex()`
- `UnitAura()` → Use `C_UnitAuras.GetAuraDataByIndex()`
- `GetAddOnMetadata()` → Use `C_AddOns.GetAddOnMetadata()`
- `IsAddOnLoaded()` → Use `C_AddOns.IsAddOnLoaded()`
- `CombatLogGetCurrentEventInfo()` → Returns Secret Values in instances
- `IsEncounterInProgress()` → Removed
- `IsEncounterLimitingResurrections()` → Removed
- `IsEncounterSuppressingRelease()` → Removed
- `ActionHasRange()` → Use `C_ActionBar.ActionHasRange()`
- `CancelEmote()` → Removed
- `SetRaidTargetProtected()` → Removed

**FUNCTIONS THAT DON'T EXIST (AI hallucinations):**
- `C_Spell.GetSpellName()` — Does NOT exist. Use `C_Spell.GetSpellInfo().name`
- `C_Spell.GetSpellIcon()` — Does NOT exist. Use `C_Spell.GetSpellTexture()`
- `C_UnitAuras.GetAuraBySpellID()` — Does NOT exist. Use `C_UnitAuras.GetPlayerAuraBySpellID()`
- `C_Item.GetItemName()` — Does NOT exist. Use `C_Item.GetItemInfo().itemName`
- `RegisterEvent()` as a global — Does NOT exist. Use `frame:RegisterEvent()` or `RegisterEventCallback()`
- `C_Timer.SetTimeout()` — Does NOT exist. Use `C_Timer.After()`

### 3. Secret Values Compliance
If code accesses combat data during instances/encounters:
- Does it guard with `issecretvalue()` before comparisons?
- Does it avoid arithmetic on potentially-secret values?
- Does it avoid using secret values as table keys?
- Does it avoid `tonumber()` on secret values?
- Does it avoid truthiness tests (`if secretVal then`) on secrets?
- Does it use `C_RestrictedActions.IsAddOnRestrictionActive()` to check context?

### 4. Interface Version Check
If a .toc file is present:
- Interface line must include `120001` for current live
- Check for `## Interface: 120001` or multi-version format

### 5. Event Validity
For any registered events:
- Verify the event exists in 12.0.1
- Check if the event payload changed (many now return Secret Values)
- Note: `COMBAT_LOG_EVENT_UNFILTERED` still fires but payload is secrets in instances

## Output Format

```
## Verification Report

**Target:** [what was verified]
**Patch:** 12.0.1 (Interface 120001)
**Overall:** PASS / FAIL / WARNINGS

### Results

| # | Check | Status | Details |
|---|-------|--------|---------|
| 1 | API: FunctionName() | ✅ PASS | Exists in 12.0.1, signature correct |
| 2 | API: OldFunction() | ❌ FAIL | Removed in 12.0.0. Use NewFunction() |
| 3 | Secret Values | ⚠️ WARN | UnitHealth() not guarded with issecretvalue() |
| 4 | Interface Version | ✅ PASS | 120001 present in TOC |

### Required Fixes
1. Replace `OldFunction()` with `NewFunction()` — [wiki link]
2. Add `issecretvalue()` guard around UnitHealth() usage

### Suggested Improvements
- [optional recommendations]
```

If verifying a file path, read the file first with the Read tool, then perform all checks.
