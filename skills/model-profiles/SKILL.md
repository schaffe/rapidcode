---
name: model-profiles
description: Reference for model capability tiers and step-kind-to-model mapping. Use when assigning models in plan-as-folder or understanding dispatch behavior.
---

# Model Profiles

## Tiers

### Deep Reasoning
Best-in-class reasoning, multi-step chains, multi-file coordination.
**Struggles with:** speed, cost.

### Balanced
Solid all-rounder, good logic, reasonable speed.
**Struggles with:** deepest reasoning chains.

### Fast/Cheap
Low latency, minimal token cost, good for terminal actions and simple file gen.
**Struggles with:** nuanced logic, multi-step reasoning.

## Environment Mappings

| Tier | opencode | Claude Code | Gemini CLI |
|------|----------|-------------|------------|
| Deep Reasoning | Big Pickle (GLM-5.2) / DeepSeek V4 Flash | Claude Opus 4.5 | Gemini 2.5 Pro |
| Balanced | DeepSeek V4 Flash | Claude Sonnet 4.5 | Gemini 2.5 Flash |
| Fast/Cheap | North Mini Code / MiMo 2.5 | Claude Haiku 4.5 | Gemini 2.5 Flash-Lite |

## Step-kind-to-tier Mapping

| Kind | Complexity | Recommended Tier |
|------|-----------|-----------------|
| code (complex, >2 files, cross-module) | high | Deep Reasoning |
| code (simple, 1 file) | medium | Balanced |
| author-test (contract-based) | medium | Balanced |
| run-test (command execution) | low | Fast/Cheap |
| bootstrap (scaffolding) | low | Fast/Cheap |
| Phase 2 reviewer | high | Deep Reasoning |
| Phase 2 fixer | medium | Balanced |

## Verification Guidance

When testing tier→model resolution for a given environment:

1. Check that each tier (Deep Reasoning, Balanced, Fast/Cheap) maps to a concrete model ID for the target environment in the table above.
2. For a given step kind (code, author-test, run-test, bootstrap, reviewer, fixer), verify that the assigned tier matches the recommended tier in the step-kind-to-tier mapping.
3. If a step file uses a tier label (e.g., `Model: Fast/Cheap`) instead of a concrete model ID, the executor must resolve it via this table before dispatching.
4. To automate resolution checks:
   ```bash
   # Verify all tier labels in step files resolve to concrete models
   for env in opencode claude-code gemini-cli; do
     for tier in "Deep Reasoning" Balanced Fast/Cheap; do
       grep -q "$tier.*$env" skills/model-profiles/SKILL.md && \
         echo "OK: $tier → $env mapping found" || \
         echo "MISSING: $tier → $env mapping"
     done
   done
   ```

## Per-executor Model Profiles

### opencode models

- **DeepSeek V4 Flash (Free):** strongest all-rounder, 1M context, high output token volume
- **Big Pickle / GLM-5.2:** raw intelligence rivaling frontier, unpredictable/rogue behavior risk
- **MiMo 2.5 (Free):** fast, sliding-window attention, good for multi-agent orchestration, weak nuanced logic
- **North Mini Code (Free):** speed king, lowest TTFT, lacks depth for architectural patterns

### Claude Code models

- **Claude Opus 4.5:** most capable for deep reasoning, time-insensitive tasks
- **Claude Sonnet 4.5:** balanced capability, good all-rounder
- **Claude Haiku 4.5:** fast/cheap, terminal actions, simple file gen

### Gemini CLI models

- **Gemini 2.5 Pro:** deep reasoning
- **Gemini 2.5 Flash:** balanced
- **Gemini 2.5 Flash-Lite:** fast/cheap
