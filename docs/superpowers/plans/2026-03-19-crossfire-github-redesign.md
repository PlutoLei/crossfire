# Crossfire GitHub Page Redesign — Implementation Plan

> **For agentic workers:** Use superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking. Image generation tasks require `/paperbanana` skill invocation.

**Goal:** Redesign crossfire GitHub repo page with updated content (v1.8), new paperbanana-generated images, and restructured README.

**Architecture:** Two paperbanana images (banner + pipeline), restructured README.md (EN) + README_CN.md (CN), new docs/architecture.md for detailed pipeline. All work in `E:/tmp/crossfire-sync2/` (clone of `PlutoLei/crossfire`, master branch).

**Tech Stack:** paperbanana (Google Gemini API), Markdown, GitHub CLI (`gh`)

**Spec:** [docs/superpowers/specs/2026-03-19-crossfire-github-redesign-design.md](../specs/2026-03-19-crossfire-github-redesign-design.md)

---

## File Structure

| File | Action | Responsibility |
|------|--------|---------------|
| `assets/banner.png` | REPLACE | Fun conceptual banner (Claude + Codex collaboration) |
| `assets/pipeline.png` | CREATE | Simplified 4-phase pipeline diagram |
| `docs/architecture.md` | CREATE | Detailed architecture documentation with full pipeline breakdown |
| `docs/architecture-detailed.png` | CREATE | Detailed pipeline flowchart with sub-steps |
| `README.md` | REWRITE | English README, ~150-200 lines, v1.8 content |
| `README_CN.md` | REWRITE | Chinese README, synced structure, native writing |

---

## Chunk 1: Image Generation

### Task 1: Generate Banner Image

**Files:**
- Create: `assets/banner.png` (replaces existing)

- [ ] **Step 1: Invoke paperbanana to generate banner**

Use `/paperbanana` with the following brief:
- **Type:** Conceptual illustration
- **Subject:** Two AI characters (Claude = blue/architect with blueprint, Codex = green/executor with toolbox) in adversarial collaboration, "crossfire" concept — debating over code
- **Style:** Fun, slightly whimsical, similar energy to current Gemini/Imagen banner. Clean background, tech-meets-cartoon aesthetic
- **Size:** 1200 x 600 px
- **Output:** `E:/tmp/crossfire-sync2/assets/banner.png`

- [ ] **Step 2: Verify banner image**

Open the generated image to verify quality and style. If unsatisfactory, regenerate with adjusted prompt.

- [ ] **Step 3: Commit banner**

```bash
cd E:/tmp/crossfire-sync2
git add assets/banner.png
git commit -m "art: replace banner with paperbanana-generated illustration"
```

### Task 2: Generate Pipeline Diagram (Simple)

**Files:**
- Create: `assets/pipeline.png`

- [ ] **Step 1: Invoke paperbanana to generate simplified pipeline diagram**

Use `/paperbanana` with the following brief:
- **Type:** Architecture/method diagram
- **Subject:** Crossfire 4-phase pipeline — simplified view:
  ```
  Phase 0: PLAN        → Claude (Architect)
  Phase 1: EXECUTE     → Codex (Executor)
  Phase 2: REVIEW      → Claude + Codex (Cross-review)
  Phase 3: REPORT      → Auto-commit
  ```
  Show L1 (skip Phase 0) vs L2/L3 (full pipeline) branching.
  Annotate Claude (blue) vs Codex (green) roles at each phase.
- **Style:** Clean technical diagram, white background, readable at GitHub scale, color-coded by actor
- **Size:** 1200 x 600 px
- **Output:** `E:/tmp/crossfire-sync2/assets/pipeline.png`

- [ ] **Step 2: Verify pipeline diagram**

Check readability at typical GitHub README width (~800px rendered).

- [ ] **Step 3: Commit pipeline diagram**

```bash
cd E:/tmp/crossfire-sync2
git add assets/pipeline.png
git commit -m "art: add simplified pipeline diagram via paperbanana"
```

### Task 3: Generate Detailed Architecture Diagram

**Files:**
- Create: `docs/architecture-detailed.png`

- [ ] **Step 1: Create docs directory if needed**

```bash
mkdir -p E:/tmp/crossfire-sync2/docs
```

- [ ] **Step 2: Invoke paperbanana to generate detailed pipeline diagram**

Use `/paperbanana` with the following brief:
- **Type:** Architecture/method diagram (detailed)
- **Subject:** Full crossfire pipeline with all sub-steps:
  - Phase 0: EXPLORE → PLAN → DEBATE (≤3 rounds) → LOCK
  - Phase 1: Blueprint injection → Codex EXECUTE (single or multi-step for L3) → Step D integrity check
  - Phase 2: Layer 1 (Claude review) → Layer 1.5 (pytest/linter) → Layer 2 (Codex audit) → Cross-review (L3)
  - Phase 3: REPORT + git commit
  - Show fix-reaudit loop (≤3 rounds) in Phase 2
  - L1/L2/L3 branching paths
  - Windows defense layer annotation on Phase 1
- **Style:** Comprehensive flowchart, white background, color-coded actors (blue=Claude, green=Codex, gray=deterministic), connecting arrows with labels
- **Size:** 1200 x 900 px
- **Output:** `E:/tmp/crossfire-sync2/docs/architecture-detailed.png`

- [ ] **Step 3: Commit detailed diagram**

```bash
cd E:/tmp/crossfire-sync2
git add docs/architecture-detailed.png
git commit -m "art: add detailed architecture diagram via paperbanana"
```

---

## Chunk 2: README.md Rewrite (English)

### Task 4: Rewrite README.md

**Files:**
- Modify: `README.md` (full rewrite)

- [ ] **Step 1: Write the new README.md**

Replace entire contents of `README.md` with the following structure (~150-200 lines):

```markdown
<p align="center">
  <img src="assets/banner.png" alt="crossfire banner" />
</p>

<h1 align="center">crossfire</h1>
<p align="center">
  <strong>Actor-Critic collaboration for Claude + Codex</strong><br/>
  Two AI partners walk into your repo. One draws the map, one carries the toolbox, both argue like a championship debate team.
</p>
<p align="center">
  <img alt="Version" src="https://img.shields.io/badge/version-1.8-orange?style=flat-square" />
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude-Opus%204.6-2B6CB0?style=flat-square" />
  <img alt="Codex CLI" src="https://img.shields.io/badge/Codex-GPT--5.4-10A37F?style=flat-square" />
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Windows-0078D4?style=flat-square" />
  <img alt="MIT License" src="https://img.shields.io/badge/License-MIT-black?style=flat-square" />
</p>

<p align="center">
  <strong>English</strong> | <a href="README_CN.md">中文</a>
</p>

## Why Crossfire?

[3-4 sentences explaining core value proposition:
- Heterogeneous model cross-review reduces confirmation bias
- Different training distributions catch each other's blind spots
- arXiv:2602.03794 reference
- Claude as Architect/Reviewer, Codex as Executor/Auditor]

## How It Works

<p align="center">
  <img src="assets/pipeline.png" alt="crossfire pipeline" />
</p>

[Brief text explaining:
- Phase 0: Claude explores, plans, debates with Codex, locks blueprint
- Phase 1: Codex executes from locked blueprint
- Phase 2: Multi-layer review (Claude → deterministic → Codex → cross-review)
- Phase 3: Structured report + auto-commit]

### Workflow Tiers

| Level | Pipeline | Best For | Review Depth |
|-------|----------|----------|-------------|
| L1 | Skip Phase 0 → EXECUTE → Quick review | Small edits, known fixes | Basic |
| L2 | Full 4-phase pipeline | Default for most development | Strong (3-round cap) |
| L3 | Full pipeline + mandatory cross-review | Architecture changes, high-impact | Maximum |

## Quick Start

### Prerequisites
- Claude Code v1.0+
- Codex CLI v0.115.0+ (`npm install -g @openai/codex`)
- Configure `~/.codex/config.toml`: model = "gpt-5.4"

### Installation
[git clone + copy commands]

### First Command
[Example: /crossfire code: ...]

## Templates

9 templates for common task types:

| Template | Default Level | Description | Example |
|----------|:---:|-------------|---------|
| `code` | L1/L2 | New feature implementation | `/crossfire code: Add parse_date to utils.py` |
| `bugfix` | L2 | Root-cause fix + regression guard | `/crossfire bugfix: Fix confusion matrix colors` |
| `refactor` | L2 | Improve structure, preserve behavior | `/crossfire refactor: Extract dedup logic` |
| `test` | L1 | Add/harden automated tests | `/crossfire test: Unit tests for process_one_paper` |
| `review` | L1 | Focused Codex code review | `/crossfire review: Review generate_essay_en.py` |
| `audit` | L2 | Dual-source parallel audit (Claude + agent) | `/crossfire audit: Audit all TextMamba3D changes` |
| `optimize` | L2 | Performance/resource optimization | `/crossfire optimize: Async batch requests` |
| `architect` | L3 | System-level architecture design | `/crossfire architect: Design figure compositor` |
| `research` | L2 | Implement from research architecture | `/crossfire research: Implement per architecture_proposal.md` |

## Configuration

### Codex CLI
[config.toml key settings: model, reasoning_effort, windows sandbox]

### Windows Defense Layer
[Brief summary table of 4 protections: path normalization, cd injection, blueprint embedding, output integrity fallback]

## Architecture Details

For detailed pipeline breakdown including sub-phases, review layers, and fix-reaudit loops, see [Architecture Documentation](docs/architecture.md).

## Credits

- Actor-Critic pipeline for Claude Code × Codex CLI
- Inspired by [heterogeneous multi-agent debate](https://arxiv.org/abs/2602.03794)
- Banner & diagrams generated with [PaperBanana](https://github.com/PlutoLei/paperbanana) + Google Gemini
- MIT Licensed
```

- [ ] **Step 2: Verify README renders correctly**

Check that:
- Banner image displays
- Pipeline diagram displays
- All badges show correct values (1.8, Opus 4.6, GPT-5.4, Windows, MIT)
- 9 templates listed (not 7)
- No references to codex-execute or gpt-5.3-codex
- Language switcher links work

- [ ] **Step 3: Commit README.md**

```bash
cd E:/tmp/crossfire-sync2
git add README.md
git commit -m "docs: rewrite README.md for v1.8 with new structure and templates"
```

---

## Chunk 3: README_CN.md + docs/architecture.md

### Task 5: Rewrite README_CN.md

**Files:**
- Modify: `README_CN.md` (full rewrite)

- [ ] **Step 1: Write the new README_CN.md**

Same structure as English README, but natively written Chinese (not translated). Key changes from current version:
- Badges: version 1.8, GPT-5.4, add Windows badge
- Templates: 9 templates (add audit, architect, research)
- Remove codex-execute references in delegation table
- Update Phase 1 delegation to "自有逻辑"（Codex CLI + 蓝图注入 + Windows 防护层）
- Add Configuration section with Windows defense layer
- Add link to docs/architecture.md

- [ ] **Step 2: Verify Chinese README**

Check that:
- Structure matches English version
- Chinese reads naturally (not translated)
- All technical terms consistent with SKILL.md terminology
- No outdated references

- [ ] **Step 3: Commit README_CN.md**

```bash
cd E:/tmp/crossfire-sync2
git add README_CN.md
git commit -m "docs: rewrite README_CN.md for v1.8, sync with English structure"
```

### Task 6: Create docs/architecture.md

**Files:**
- Create: `docs/architecture.md`

- [ ] **Step 1: Write docs/architecture.md**

Content should include:
1. Detailed architecture diagram image reference (`architecture-detailed.png`)
2. Phase 0 breakdown: EXPLORE → PLAN → DEBATE (≤3 rounds, objective/subjective distinction) → LOCK
3. Phase 1 breakdown: Blueprint injection, single vs multi-step EXECUTE (L3), Windows defense layer (Step A-D)
4. Phase 2 breakdown: Layer 1 (Claude) → Layer 1.5 (deterministic: pytest/linter) → Layer 2 (Codex audit) → Cross-review
5. Fix-reaudit loop: ≤3 rounds, exit conditions (clean pass / subjective only / 3-round cap → escalate)
6. Phase 3: Structured report + auto-commit rules
7. Skill delegation strategy table (updated for v1.8)
8. Key constraints summary

- [ ] **Step 2: Commit docs/architecture.md**

```bash
cd E:/tmp/crossfire-sync2
git add docs/architecture.md
git commit -m "docs: add detailed architecture documentation"
```

---

## Chunk 4: GitHub Settings + Push

### Task 7: Update GitHub Repository Settings

- [ ] **Step 1: Add repository topics**

```bash
cd E:/tmp/crossfire-sync2
gh repo edit PlutoLei/crossfire --add-topic claude-code --add-topic codex --add-topic actor-critic --add-topic code-review --add-topic ai-collaboration --add-topic claude-skill
```

- [ ] **Step 2: Update repo description**

```bash
gh repo edit PlutoLei/crossfire --description "Claude + Codex Actor-Critic Pipeline — heterogeneous AI cross-review for end-to-end code implementation"
```

### Task 8: Push All Changes

- [ ] **Step 1: Review all commits before pushing**

```bash
cd E:/tmp/crossfire-sync2
git log --oneline master
```

Verify commits are:
1. Design spec
2. Banner image
3. Pipeline diagram
4. Detailed architecture diagram
5. README.md rewrite
6. README_CN.md rewrite
7. docs/architecture.md

- [ ] **Step 2: Push to remote**

```bash
cd E:/tmp/crossfire-sync2
git push origin master
```

- [ ] **Step 3: Verify GitHub page**

Open `https://github.com/PlutoLei/crossfire` and verify:
- Banner displays correctly
- Pipeline diagram shows in "How It Works"
- Badges show v1.8, Opus 4.6, GPT-5.4, Windows
- 9 templates listed
- Language switcher works
- Topics visible
- Description updated
