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
  <img alt="Claude" src="https://img.shields.io/badge/Claude-Opus%204.6-2B6CB0?style=flat-square" />
  <img alt="Codex" src="https://img.shields.io/badge/Codex-GPT--5.4-10A37F?style=flat-square" />
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Windows-0078D4?style=flat-square" />
  <img alt="MIT License" src="https://img.shields.io/badge/License-MIT-black?style=flat-square" />
</p>

<p align="center">
  <strong>English</strong> | <a href="README_CN.md">中文</a>
</p>

## Why Crossfire?

What happens when you lock Claude and Codex in a room and tell them to agree on your architecture? Surprisingly good code.

`crossfire` is a Claude Code skill that runs an end-to-end **Actor-Critic pipeline** pairing two AI models from different training distributions. Claude Code (Opus 4.6) plays Architect and Reviewer — the strategic thinker with the whiteboard. Codex CLI (GPT-5.4) plays Executor and Auditor — the heavy lifter who codes fast and double-checks everything like a suspicious airport inspector.

The core insight: heterogeneous models catch each other's blind spots better than homogeneous teams ([arXiv:2602.03794](https://arxiv.org/abs/2602.03794)). Phase 0 is couples therapy for AIs. Phase 2 is trust issues as a service. Your codebase is the winner.

## How It Works

<p align="center">
  <img src="assets/pipeline.png" alt="crossfire pipeline" width="800" />
</p>

The pipeline has four phases — plan, execute, review, report:

1. **Phase 0 — PLAN** (Claude): Explore codebase → draft blueprint → adversarial debate with Codex → lock final plan
2. **Phase 1 — EXECUTE** (Codex): Implement exactly from the locked blueprint, with Windows defense layer pre-processing
3. **Phase 2 — REVIEW** (Both): Claude initial review → deterministic checks (tests/linters) → Codex final audit → cross-review
4. **Phase 3 — REPORT**: Structured report + automatic git commit

### Workflow Tiers

| Level | Pipeline | Best For | Review Depth |
|-------|----------|----------|:------------:|
| **L1** | Skip Phase 0 → EXECUTE → quick review | Small edits, known fixes, low ambiguity | Basic |
| **L2** | Full 4-phase pipeline | Default for most development work | Strong (3-round cap) |
| **L3** | Full pipeline + mandatory cross-review | Architecture changes, risky refactors, critical releases | Maximum |

When file count and line count point to different levels, the higher level wins.

## Quick Start

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) v1.0+
- [Codex CLI](https://github.com/openai/codex) v0.115.0+ — `npm install -g @openai/codex`
- Configure Codex model in `~/.codex/config.toml`:
  ```toml
  model = "gpt-5.4"
  model_reasoning_effort = "xhigh"
  ```

### Installation

```bash
git clone https://github.com/PlutoLei/crossfire.git
mkdir -p ~/.claude/skills/crossfire/references
cp crossfire/SKILL.md ~/.claude/skills/crossfire/
cp crossfire/references/*.md ~/.claude/skills/crossfire/references/
```

### First Command

```bash
/crossfire code: Add a parse_date function to src/utils.py supporting ISO 8601 and Chinese date formats
```

## Templates

Nine templates for common task types:

| Template | Level | Description | Example |
|----------|:-----:|-------------|---------|
| `code` | L1/L2 | New feature implementation | `/crossfire code: Add parse_date to utils.py` |
| `bugfix` | L2 | Root-cause fix + regression guard | `/crossfire bugfix: Fix confusion matrix colors` |
| `refactor` | L2 | Improve structure, preserve behavior | `/crossfire refactor: Extract dedup logic` |
| `test` | L1 | Add or harden automated tests | `/crossfire test: Unit tests for process_one_paper` |
| `review` | L1 | Focused Codex code review | `/crossfire review: Review generate_essay_en.py` |
| `audit` | L2 | Dual-source parallel audit (Claude + code-reviewer agent) | `/crossfire audit: Audit TextMamba3D changes` |
| `optimize` | L2 | Performance or resource optimization | `/crossfire optimize: Async batch requests` |
| `architect` | L3 | System-level architecture design | `/crossfire architect: Design figure compositor` |
| `research` | L2 | Implement from research architecture (auto inject-plan) | `/crossfire research: Implement per architecture_proposal.md` |

**Syntax:** `/crossfire <template>: <description>` or `/crossfire L2: <description>`

**Flags:** `--no-debate` (skip Phase 0 debate) · `--no-audit` (skip Codex final audit) · `--inject-plan <dir>` (inject research outputs)

## Configuration

### Codex CLI

Key settings in `~/.codex/config.toml`:

```toml
model = "gpt-5.4"
model_reasoning_effort = "xhigh"

[windows]
sandbox = "elevated"
```

### Windows Defense Layer

All Codex calls go through mandatory pre-processing on Windows:

| Protection | Rule | Phases |
|-----------|------|:------:|
| Path normalization | All paths converted to forward slashes | All |
| `cd` injection | First line of every Codex prompt: `cd <normalized-abs-path>` | All |
| Blueprint embedding | ≤200 lines inline in prompt; >200 lines via file reference | Phase 1 |
| Output integrity fallback | Detect missing/truncated files → Claude completes via Write tool | Phase 1 |

## Architecture Details

For the complete pipeline breakdown — including Phase 0 sub-steps, multi-layer review, fix-reaudit loops, and skill delegation strategy — see [Architecture Documentation](docs/architecture.md).

<p align="center">
  <img src="docs/architecture-detailed.png" alt="detailed architecture" width="600" />
</p>

## Credits

- Actor-Critic pipeline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) × [Codex CLI](https://github.com/openai/codex)
- Inspired by [heterogeneous multi-agent debate](https://arxiv.org/abs/2602.03794)
- Banner & diagrams generated with [PaperBanana](https://github.com/PlutoLei/paperbanana) + Google Gemini
- MIT Licensed
