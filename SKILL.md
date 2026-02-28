---
name: crossfire
description: Claude + Codex 端到端 Actor-Critic 协作。四阶段 Pipeline：探索规划辩论 → 执行 → 多层审查 → 报告。异源模型互审消除信息茧房。
version: 1.2.0
tags: [Codex, Pipeline, Actor-Critic, Multi-Model, Crossfire]
---

# /crossfire — Claude + Codex 交叉火力 Pipeline

## 核心理念

Claude Code (Opus 4.6) = Actor（架构师/审查员），Codex (GPT-5.3-Codex) = Executor + Critic（执行者/审计员）。异源模型互审避免同源信息茧房（arXiv:2602.03794）。端到端封装从探索到报告的完整流程。

| 角色 | 模型 | 职责 |
|------|------|------|
| 架构师 | Claude Opus 4.6 | 探索、规划、审查、修复、决策 |
| 执行者 | Codex gpt-5.3-codex | 编码实现、文件修改 |
| 审计员 | Codex gpt-5.3-codex `--full-auto` | 架构质询（Phase 0）、代码终审（Phase 2） |

## Skill 委托策略

crossfire 是**编排器**，复用已有 skill 的能力而非重写逻辑。

| 阶段 | L2 委托方式 | L3 委托方式 | 目标 Skill |
|------|-----------|-----------|-----------|
| Phase 0a EXPLORE | **invoke** | **invoke** | `planning-with-files` — 自主状态管理（task_plan.md / findings.md / progress.md） |
| Phase 0a-0b 多方案 | **参考原则** | **invoke** | `brainstorming` — L2 参考多方案+YAGNI 原则；L3 invoke 交互式探索，用户参与方案选择 |
| Phase 0b PLAN 蓝图 | **参考格式** | **invoke** | `writing-plans` — L2 参考蓝图模板格式；L3 invoke 交互式规划，用户确认蓝图 |
| Phase 1 EXECUTE | **复用模式** | **复用模式** | `codex-execute` — 复用 Codex CLI 调用模式（蓝图注入是 crossfire 独有逻辑） |

**独有逻辑**（不委托，crossfire 自身实现）：
- Phase 0c DEBATE — Codex 质询蓝图
- Phase 0d LOCK — 冻结蓝图
- Phase 2 多层 REVIEW — Claude 初审 + Codex 终审 + 交叉审查
- Phase 3 REPORT — 结构化报告 + 自动提交
- 升级机制 — 3 轮上限 → 升级用户

## 何时使用

| 场景 | 使用 |
|------|------|
| 大量代码编写的实现任务 | ✅ |
| 需要异源模型审查提升质量 | ✅ |
| 架构设计需要对抗性验证 | ✅ |
| 简单单文件小修改 | ❌ 直接用 Claude Code |
| 纯数据处理、Notebook 开发 | ❌ |

---

## 四阶段 Pipeline

```
L1:     Phase 1(EXECUTE) → Phase 2(Claude快审) → Phase 3(REPORT)
L2/L3:  Phase 0(EXPLORE→PLAN→DEBATE→LOCK) → Phase 1(EXECUTE) → Phase 2(多层REVIEW) → Phase 3(REPORT)
```

### Phase 0: EXPLORE → PLAN → DEBATE → LOCK（L2/L3）

首先 **invoke `planning-with-files` skill** 初始化状态管理文件。

- **L2（自主模式）：** Claude 全自主完成，用户仅在升级时介入。
- **L3（半交互模式）：** invoke `brainstorming` 和 `writing-plans` 让用户参与方案选择，DEBATE/LOCK 仍自主。

1. **EXPLORE** — 读取相关源文件、GitNexus 查执行流、识别可复用代码。发现即时写入 findings.md（2-action rule）
2. **PLAN** — L2: 参考 brainstorming 原则自主提出多方案 | L3: **invoke `brainstorming`** 交互式探索，再 **invoke `writing-plans`** 生成蓝图
3. **DEBATE** — 提交蓝图给 Codex `--full-auto` xhigh 质询，最多 3 轮辩论
4. **LOCK** — 冻结蓝图写入 task_plan.md，后续阶段不得偏离

详见 [references/debate-protocol.md](references/debate-protocol.md)

### Phase 1: EXECUTE

复用 `codex-execute` 的 Codex CLI 调用模式，增加蓝图注入：

```bash
codex exec --dangerously-bypass-approvals-and-sandbox -m gpt-5.3-codex -C "<dir>" "<task + 蓝图引用>"
```

### Phase 2: REVIEW

| 层级 | 执行者 | 适用级别 |
|------|--------|---------|
| Layer 1 | Claude 初审 | 所有 |
| Layer 2 | Codex `--full-auto` 终审（蓝图 + diff） | L2/L3 |
| 交叉审查 | Codex 审查 Claude 的修正 | L3 必选，L2 可选 |

修复-重审循环：最多 3 轮。退出：✅ Clean pass | 🟡 仅主观建议 | 🛑 3 轮上限 → 升级用户

详见 [references/review-protocol.md](references/review-protocol.md)

### Phase 3: REPORT

输出结构化报告 + 更新 progress.md。✅ 时自动 `git commit`；🛑 时不提交。

---

## 工作流分级

| 级别 | 触发条件 | Phase 0 | Phase 1 | Phase 2 | Phase 3 |
|------|---------|---------|---------|---------|---------|
| L1 | 单文件, <30 行 | 跳过 | 单次 Codex | Claude 快审 | 简要摘要 |
| L2 | 多文件, 需上下文 | 完整 4 步 | 单次 Codex | 多层（3 轮） | 完整报告 |
| L3 | 架构级, 多模块 | 完整 4 步 | 多步 Codex | 多层+交叉审查 | 完整报告 |

## 预设模板

| 模板 | 默认级别 | Phase 0 | 多层 Review |
|------|---------|---------|------------|
| `code` | L1/L2 | L2 时 | L2 时 |
| `bugfix` | L2 | ✓ | ✓ |
| `refactor` | L2 | ✓ | ✓ |
| `test` | L1 | ✗ | ✗ |
| `review` | L1 | ✗ | ✗ |
| `optimize` | L2 | ✓ | ✓ |
| `architect` | L3 | ✓ (核心) | ✓ |

调用：`/crossfire <模板>: <描述>` | `/crossfire L2: <描述>`

可选标志：`--no-debate`（跳过 Phase 0）| `--no-audit`（跳过 Codex 终审）

详见 [references/templates-and-faq.md](references/templates-and-faq.md)

---

## 关键约束

- **模型** — 强制 `gpt-5.3-codex`（`~/.codex/config.toml` 配置 `model_reasoning_effort = "xhigh"`）
- **Windows 路径** — `-C` 参数用正斜杠或引号
- **Windows 大文件** — 单次 Codex >300 行触发错误码 206，用两步法（详见 templates-and-faq.md）
- **并发** — 同一时间只运行一个 Codex 实例
- **蓝图纪律** — Phase 1/2 不得静默偏离锁定蓝图
- **升级机制** — 辩论/审查达 3 轮上限或僵局时，必须升级给用户
