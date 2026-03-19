# Crossfire GitHub Page Redesign — Design Spec

**Date:** 2026-03-19
**Status:** Approved
**Audience:** Community showcase + personal archive (dual-purpose)

---

## Overview

Redesign the crossfire GitHub repository page to reflect v1.8 changes (codex-execute absorption, 300-line fix, Windows hardening), update outdated references (gpt-5.3-codex → gpt-5.4, version 1.2.0 → 1.8), and replace images with paperbanana-generated illustrations.

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language strategy | English main + Chinese secondary | README.md (EN) for community, README_CN.md (CN) for personal use |
| Pipeline diagram | Simple in README, detailed in docs/ | Keep README concise (~150-200 lines), deep info accessible via link |
| Design approach | 方案 A: 精简展示型 | Balance between showcase impact and information completeness |

---

## 1. Visual Assets (2 paperbanana images)

### 1.1 Banner Image (`assets/banner.png`)

- **Style:** Fun/conceptual, similar to existing Gemini/Imagen-generated banner
- **Content:** Claude and Codex as two characters in adversarial collaboration — Actor-Critic crossfire concept
- **Size:** 1200 x 600 px (optimal GitHub README ratio)
- **Generation:** paperbanana skill, conceptual illustration mode

### 1.2 Pipeline Diagram (`assets/pipeline.png`)

- **Style:** Clean technical diagram, readable at GitHub scale
- **Content:** Simplified 3-layer architecture showing:
  - Phase 0: PLAN (Claude) → Phase 1: EXECUTE (Codex) → Phase 2: REVIEW (Claude+Codex) → Phase 3: REPORT
  - Claude vs Codex role annotations at each phase
  - L1/L2/L3 branching indication
- **Size:** 1200 x 600 px
- **Generation:** paperbanana skill, method/architecture diagram mode

### 1.3 Detailed Architecture Diagram (`docs/architecture-detailed.png`)

- **Style:** Comprehensive technical flowchart
- **Content:** Full 4-phase pipeline with sub-steps:
  - Phase 0: EXPLORE → PLAN → DEBATE → LOCK
  - Phase 1: Single/multi-step EXECUTE + Windows defense layer
  - Phase 2: Layer 1 (Claude) → Layer 1.5 (deterministic) → Layer 2 (Codex) → Cross-review
  - Phase 3: REPORT + auto-commit
  - L1/L2/L3 branch paths
  - Fix-reaudit loop (3-round cap)
- **Size:** 1200 x 900 px
- **Generation:** paperbanana skill

---

## 2. README.md Structure (English, ~150-200 lines)

```
[banner.png — centered]

# Crossfire

> Claude + Codex Actor-Critic Pipeline for adversarial code review

[badges: version 1.8 | Claude Opus 4.6 | GPT-5.4 | MIT License | Windows]

## Why Crossfire?
- 3-4 sentences: heterogeneous model cross-review reduces confirmation bias
- Key differentiator: different training distributions catch each other's blind spots

## How It Works
[pipeline.png — simplified architecture diagram]
- Brief text explaining 4 phases
- L1/L2/L3 tier table (trigger conditions + what each level does)

## Quick Start
- Prerequisites (Claude Code, Codex CLI, config)
- Installation steps
- First command example: `/crossfire code: ...`

## Templates
- 9-template table: code | bugfix | refactor | test | review | audit | optimize | architect | research
- Each row: template name, default level, one-line description, example

## Configuration
- Codex CLI config (`~/.codex/config.toml`)
- Windows defense layer summary (path normalization, cd injection, blueprint embedding, output integrity)

## Architecture
→ Link to [docs/architecture.md](docs/architecture.md) for detailed pipeline diagram and phase breakdown

## License
MIT
```

## 3. docs/architecture.md

Dedicated architecture documentation containing:
- Detailed architecture diagram (paperbanana-generated)
- Phase 0 sub-steps: EXPLORE → PLAN → DEBATE → LOCK
- Phase 1: Single vs multi-step EXECUTE, blueprint injection, Windows defense layer
- Phase 2: Multi-layer REVIEW with fix-reaudit loop
- Phase 3: Structured report + auto-commit
- Skill delegation strategy table
- Key constraints

## 4. README_CN.md (Chinese)

- Same structure as English README
- Natively written Chinese (not translated)
- Synced content with English version

## 5. Badge Updates

Replace outdated badges:
- Version: ~~1.2.0~~ → **1.8**
- Models: Add Claude Opus 4.6 + GPT-5.4
- Platform: Add Windows badge
- Remove any codex-execute references

## 6. GitHub Repository Settings

Add topics: `claude-code`, `codex`, `actor-critic`, `code-review`, `ai-collaboration`, `claude-skill`

---

## Files to Create/Modify

| File | Action | Description |
|------|--------|-------------|
| `assets/banner.png` | REPLACE | New paperbanana banner |
| `assets/pipeline.png` | CREATE | Simplified pipeline diagram |
| `docs/architecture.md` | CREATE | Detailed architecture documentation |
| `docs/architecture-detailed.png` | CREATE | Detailed pipeline diagram |
| `README.md` | REWRITE | New structure, updated content, v1.8 |
| `README_CN.md` | REWRITE | Chinese version, synced with EN |

## Out of Scope

- GitHub Actions / CI setup
- Wiki pages
- Release tags (can be done separately)
- SKILL.md content changes (already done in v1.8)
