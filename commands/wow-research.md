---
description: "Research and verify WoW addon API claims, patterns, and information"
---

Research and verify information about: $ARGUMENTS

You are a WoW addon research coordinator. Your job is to launch a parallel agent swarm to research the given topic, then synthesize verified findings.

## Process

### Step 1: Launch 3 Parallel Research Agents

Use the Agent tool to spawn all 3 agents simultaneously in a single message:

**Agent 1 — Wiki Specialist** (subagent_type: "WoW Addon Researcher")
```
Search warcraft.wiki.gg for: $ARGUMENTS

Instructions:
- Search warcraft.wiki.gg using WebSearch with "site:warcraft.wiki.gg $ARGUMENTS"
- For each result, use WebFetch to verify the URL loads and contains relevant content
- Extract: function signatures, parameters, return values, events, code examples
- Note the page's "Patch changed" or "Added in" version info
- Check both the API page AND the Patch 12.0.0/API_changes page for context
- If the topic relates to a C_ namespace, also check Patch_12.0.0/Planned_API_changes
- Return structured findings with exact URLs that you verified exist
```

**Agent 2 — Community Specialist** (subagent_type: "general-purpose")
```
Search Reddit, Wowhead, and community sites for: $ARGUMENTS

Instructions:
- Search "site:reddit.com/r/wowaddons OR site:reddit.com/r/WowUI $ARGUMENTS"
- Search "site:wowhead.com $ARGUMENTS addon API"
- Search "site:us.forums.blizzard.com $ARGUMENTS addon"
- Search "site:icy-veins.com $ARGUMENTS addon midnight"
- For each promising result, use WebFetch to verify it loads
- Extract: community advice, known issues, workarounds, developer statements
- Note dates — prioritize post-January 2026 content (12.0 era)
- Return findings with verified URLs and dates
```

**Agent 3 — Code Specialist** (subagent_type: "general-purpose")
```
Search GitHub for real-world code examples of: $ARGUMENTS

Instructions:
- Search GitHub: "$ARGUMENTS language:lua path:*.lua" via WebSearch
- Check these known-good repos for relevant code:
  - github.com/enderneko/Cell (healing frames, 12.0 adaptation)
  - github.com/BigWigsMods/BigWigs (boss mods)
  - github.com/Stanzilla/WoWUIBugs (bug reports)
  - github.com/Gethe/wow-ui-source (Blizzard UI source)
  - github.com/Ketho/BlizzardInterfaceResources
- For each result, use WebFetch to verify the code exists
- Extract: usage patterns, function calls in context, error handling approaches
- Note which WoW version/patch the code targets
- Return code snippets with source URLs
```

### Step 2: Synthesize Findings

After all 3 agents return, compile a unified report:

1. **Cross-reference** — Where do sources agree? Where do they conflict?
2. **Tag confidence levels:**
   - ✅ **CONFIRMED** — Multiple independent sources verify this (wiki + code + community)
   - ⚠️ **LIKELY** — Single credible source (official wiki, major addon codebase)
   - ❓ **UNVERIFIED** — Single community post, extrapolation, or no source found
3. **Flag conflicts** — If wiki says one thing and code does another, call it out
4. **Note 12.0 status** — Is this API current for Midnight? Was it changed/removed/added?

### Step 3: Output Format

```markdown
# Research Report: [topic]

**Date:** [today's date]
**Patch:** 12.0.1 (Midnight)
**Confidence:** [overall CONFIRMED/LIKELY/UNVERIFIED]

## Summary
[2-3 sentence overview]

## Findings

### From warcraft.wiki.gg
[Structured API info, signatures, examples]
Source: [verified URL]

### From Community
[Community advice, known issues, workarounds]
Source: [verified URLs with dates]

### From GitHub
[Real-world code examples]
Source: [verified URLs]

## 12.0 Status
[Was this changed in Midnight? Is it safe to use?]

## Conflicts / Gaps
[Any disagreements between sources, or info that couldn't be found]
```
