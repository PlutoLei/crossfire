<p align="center">
  <img src="assets/banner.png" alt="crossfire banner" />
</p>

<h1 align="center">crossfire</h1>
<p align="center">
  <strong>Actor-Critic collaboration for Claude + Codex</strong><br/>
  Two AI partners walk into your repo. One draws the map, one carries the toolbox, both argue like a championship debate team.
</p>
<p align="center">
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-Opus%204.6-2B6CB0?style=flat-square" />
  <img alt="Codex CLI" src="https://img.shields.io/badge/Codex%20CLI-GPT--5.3--Codex-10A37F?style=flat-square" />
  <img alt="MIT License" src="https://img.shields.io/badge/License-MIT-black?style=flat-square" />
  <img alt="Version" src="https://img.shields.io/badge/version-1.2.0-orange?style=flat-square" />
</p>

<p align="center">
  <strong>English</strong> | <a href="README_CN.md">中文</a>
</p>

## What is this?
What happens when you lock Claude and Codex in a room and tell them to agree on your architecture? Surprisingly good code.

`crossfire` is a Claude Code skill that runs an end-to-end Actor-Critic pipeline:
- Claude Code (Opus 4.6) plays Architect/Reviewer: the detective with the whiteboard and conspiracy strings.
- Codex CLI (GPT-5.3-Codex) plays Executor/Auditor: the heavy lifter who codes fast and double-checks everything like a suspicious airport inspector.

Phase 0 is basically couples therapy for AIs. Phase 2 is trust issues as a service. Your codebase is the winner.

Inspired by [heterogeneous multi-agent debate](https://arxiv.org/abs/2602.03794) — diverse models catch each other's blind spots better than homogeneous teams.

## Pipeline Overview
The workflow is strict enough for production, with just enough comedy to keep standup meetings survivable.

```text
Task
  |
  v
+-------------------------------------------------------------+
| Phase 0 (Architect Loop)                                    |
| Claude: EXPLORE -> PLAN -> DEBATE(with Codex) -> LOCK       |
+-------------------------------------------------------------+
  |
  v
+-------------------------------------------------------------+
| Phase 1 (Execution)                                         |
| Codex: IMPLEMENT exactly from locked blueprint              |
+-------------------------------------------------------------+
  |
  v
+-------------------------------------------------------------+
| Phase 2 (Review)                                            |
| Claude initial review -> Codex final audit -> cross-review* |
+-------------------------------------------------------------+
  |
  v
+-------------------------------------------------------------+
| Phase 3 (Report)                                            |
| Structured report + auto-commit                             |
+-------------------------------------------------------------+

* cross-review is required at L3, optional at L2
```

## Level Routing
Choose how much process you want versus how much uncertainty you can tolerate.

| Level | Route | Best For | Review Depth |
| --- | --- | --- | --- |
| L1 | Quick route, skip Phase 0, go straight to EXECUTE | Small edits, known fixes, low ambiguity tasks | Basic |
| L2 | Full pipeline: Phase 0 -> 1 -> 2 -> 3 | Default mode for most development work | Strong |
| L3 | Full pipeline + mandatory cross-review in Phase 2 | Architecture changes, risky refactors, high-impact releases | Maximum |

## Prerequisites
- Claude Code `v1.0+`
- Codex CLI `v0.98.0+`
- Patience for AI debates that sound like a cooking show finale, but end with cleaner PRs

## Installation
Clone and copy skill files:

```bash
git clone https://github.com/PlutoLei/crossfire.git
mkdir -p ~/.claude/skills/crossfire/references
cp crossfire/SKILL.md ~/.claude/skills/crossfire/
cp crossfire/references/*.md ~/.claude/skills/crossfire/references/
```

## Usage

```bash
/crossfire code: Add a parse_date function to src/utils.py
/crossfire bugfix: Fix the confusion matrix color mapping in evaluate.py
/crossfire L2: Refactor the ingestion module for async support
/crossfire --no-debate optimize: Speed up batch API requests in fetch.py
```

## Templates Quick Reference
`crossfire` ships seven templates for common task types.

| Template | Use It When You Need To | Typical Output |
| --- | --- | --- |
| `code` | Build a new feature or module | New implementation following locked blueprint |
| `bugfix` | Reproduce and fix a defect | Root-cause patch + regression guard |
| `refactor` | Improve structure without changing behavior | Safer architecture and cleaner boundaries |
| `test` | Add or harden automated tests | Coverage for critical paths and edge cases |
| `review` | Perform focused code review/audit | Structured findings and risk-ranked issues |
| `optimize` | Improve performance or resource usage | Measured bottleneck reductions |
| `architect` | Design system-level changes | Architecture plan + implementation route |

## Skill Delegation (v1.1)
`crossfire` is an **orchestrator**, not a monolith. Instead of reimplementing other skills' logic (a recipe for semantic drift and maintenance nightmares), it delegates to existing Claude Code skills where possible:

| Phase | Delegation | Target Skill | Why |
| --- | --- | --- | --- |
| Phase 0a EXPLORE | **invoke** | `planning-with-files` | Autonomous state management (task_plan.md, findings.md, progress.md) |
| Phase 0a-0b | **reference principles** | `brainstorming` | YAGNI, multi-option comparison (can't invoke — interactive workflow conflicts with autonomous Phase 0) |
| Phase 0b PLAN | **reference format** | `writing-plans` | Structured blueprint template (can't invoke — asks user to choose execution approach) |
| Phase 1 EXECUTE | **reuse patterns** | `codex-execute` | Codex CLI calling conventions (doesn't invoke — blueprint injection is crossfire-unique) |

Three delegation modes because not all skills play nice with autonomous pipelines. Some want to chat with the user (looking at you, `brainstorming`). We respect their boundaries while borrowing their wisdom.

## Credits
- Repository: `PlutoLei/crossfire`
- Collaboration model: Actor-Critic pipeline for Claude Code and Codex CLI
- Runtime roles: Claude as Architect/Reviewer, Codex as Executor/Auditor
- Banner artwork: AI-generated with Gemini (Imagen 4)

Banner generated by Gemini (Imagen 4). All skill content MIT licensed.
