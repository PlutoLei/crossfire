# Crossfire Architecture

> Detailed pipeline breakdown for the crossfire Actor-Critic collaboration skill.

<p align="center">
  <img src="architecture-detailed.png" alt="crossfire detailed architecture" />
</p>

---

## Pipeline Overview

```
L1:     Phase 1(EXECUTE) → Phase 2(Claude quick review) → Phase 3(REPORT)
L2/L3:  Phase 0(EXPLORE→PLAN→DEBATE→LOCK) → Phase 1(EXECUTE) → Phase 2(multi-layer REVIEW) → Phase 3(REPORT)
```

## Phase 0: EXPLORE → PLAN → DEBATE → LOCK

Claude drives all four sub-steps. L1 tasks skip this phase entirely.

### EXPLORE

Invoke `planning-with-files` skill for autonomous state management. Claude reads relevant source files, queries execution flow (via GitNexus if available), and identifies reusable code. Findings are written to `findings.md` immediately (2-action rule).

### PLAN

- **L2 (autonomous):** Claude generates a blueprint referencing `brainstorming` principles (YAGNI, multi-option comparison) without user interaction.
- **L3 (semi-interactive):** Invokes `brainstorming` for interactive exploration, then `writing-plans` for structured blueprint with user confirmation.

### DEBATE

The blueprint is submitted to Codex (`--full-auto`, reasoning effort `xhigh`) for adversarial review. Up to 3 rounds:

| Finding Type | Action | Costs a Round? |
|-------------|--------|:--------------:|
| Objective error (bug, incorrect API usage) | Fix immediately | No |
| Subjective suggestion (style, naming) | Claude can rebut | Yes |

Exit conditions: converged (no more findings) → LOCK | 3-round cap → escalate to user.

### LOCK

Freeze the final blueprint into `task_plan.md`. All subsequent phases must follow this blueprint — no silent deviations.

---

## Phase 1: EXECUTE

Codex implements from the locked blueprint via Codex CLI.

### Windows Defense Layer (mandatory on Windows)

Every Codex call goes through four pre-processing steps:

| Step | Rule | Purpose |
|------|------|---------|
| **A** — Path normalization | Convert all paths to forward slashes | Prevent Windows path escaping issues |
| **B** — `cd` injection | First line of prompt: `cd <normalized-path>` | Redundant with `-C` flag for shell-level safety |
| **C** — Blueprint embedding | ≤200 lines: inline in prompt; >200 lines: instruct Codex to read `.crossfire/blueprint.md` | Prevent prompt bloat |
| **D** — Output integrity check | After execution: check for missing/truncated files | Catch timeout, network, or model anomalies |

### Execution Modes

| Level | Mode | Details |
|-------|------|---------|
| L1/L2 | Single Codex call | One `codex exec` with full blueprint |
| L3 | Multi-step Codex calls | Split by module boundary, dependency-ordered. Each step checked before proceeding. Same step fails twice → escalate to user |

### Step D: Output Integrity Fallback

When Codex output is incomplete (exit code non-zero, or files missing/truncated):

1. Check Codex stdout for implementation intent
2. Claude uses Write tool to complete the file
3. Continue with remaining files
4. Each file judged independently

---

## Phase 2: REVIEW

Multi-layer review with a shared 3-round budget for fix-reaudit cycles.

### Review Layers

| Layer | Executor | Scope | Required |
|-------|----------|-------|:--------:|
| **Layer 1** | Claude | Review against locked blueprint | All levels |
| **Layer 1.5** | Deterministic (pytest, linter) | Syntax, imports, test pass | L2/L3 |
| **Layer 2** | Codex `--full-auto` | Blueprint + diff audit | L2/L3 |
| **Cross-review** | Codex reviews Claude's fixes | Verify fix quality | L3 required, L2 optional |

### Fix-Reaudit Loop

Layer 2 and cross-review share a global 3-round budget:

```
Issue found → Claude fixes → Re-review (round N+1) → ...
```

Exit conditions:
- ✅ **Clean pass** — no issues remaining
- 🟡 **Subjective only** — only style/naming suggestions, not worth fixing
- 🛑 **3-round cap** — escalate to user with summary of remaining issues

### Standalone Audit Pipeline

The `audit` template runs a separate pipeline (Phase A-F) with dual-source parallel review (Claude L1 + code-reviewer agent), merged findings, and objective final ruling. See `references/review-protocol.md` for details.

---

## Phase 3: REPORT

1. Generate structured report (summary, changes made, review findings, remaining items)
2. Update `progress.md`
3. If all clear → automatic `git commit`; if issues remain → do not commit

---

## Skill Delegation Strategy

crossfire is an **orchestrator** — it reuses existing skills rather than reimplementing their logic.

| Phase | L2 Delegation | L3 Delegation | Target Skill |
|-------|:------------:|:------------:|-------------|
| Phase 0a EXPLORE | **invoke** | **invoke** | `planning-with-files` — autonomous state management |
| Phase 0a-0b Options | **reference principles** | **invoke** | `brainstorming` — YAGNI, multi-option comparison |
| Phase 0b PLAN Blueprint | **reference format** | **invoke** | `writing-plans` — structured blueprint template |
| Phase 1 EXECUTE | **self-contained** | **self-contained** | Codex CLI + blueprint injection + Windows defense (crossfire-own logic) |

**Crossfire-only logic** (never delegated):
- Phase 0c DEBATE — Codex adversarial review of blueprint
- Phase 0d LOCK — freeze blueprint
- Phase 2 multi-layer REVIEW — Claude + Codex + cross-review
- Phase 3 REPORT — structured output + auto-commit
- Escalation — 3-round cap → surface to user

---

## Key Constraints

| Constraint | Rule |
|-----------|------|
| Model | Codex must use `gpt-5.4` (configured in `~/.codex/config.toml`) |
| Concurrency | One Codex instance at a time |
| Blueprint discipline | Phase 1/2 must not silently deviate from locked blueprint |
| Escalation | Debate/review hitting 3-round cap → must escalate to user |
| Temp files | Use project-level `.crossfire/` directory (gitignored), cleaned after Phase 3 |
| External dependencies | `planning-with-files`, `brainstorming`, `writing-plans` from superpowers plugin (tested v4.3.1+) |
