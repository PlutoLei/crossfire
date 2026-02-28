---
name: crossfire
description: Claude + Codex 端到端 Actor-Critic 协作。四阶段 Pipeline：探索规划辩论 → 执行 → 多层审查 → 报告。异源模型互审消除信息茧房。
version: 1.0.0
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

Claude 全自主完成，用户仅在升级时介入。

1. **EXPLORE** — 读取相关源文件、GitNexus 查执行流、识别可复用代码
2. **PLAN** — 提出 2-3 种方案并比较，选最优方案，草拟架构蓝图
3. **DEBATE** — 提交蓝图给 Codex `--full-auto` xhigh 质询，最多 3 轮辩论
   - 客观错误 → 直接改，不消耗轮次
   - 主观分歧 → 辩论或反驳
   - 退出：共识 → LOCK | 3 轮上限/僵局 → 升级用户 🛑
4. **LOCK** — 冻结蓝图写入临时文件，后续阶段不得偏离

详见 [references/debate-protocol.md](references/debate-protocol.md)

### Phase 1: EXECUTE

```bash
codex exec --dangerously-bypass-approvals-and-sandbox -m gpt-5.3-codex -C "<dir>" "<task>"
```

- L2/L3：Codex 提示词中原文引用锁定蓝图
- L1：直接构造 Codex 提示词（无蓝图）
- Windows 限制：单次 <300 行/15KB，大内容用 `$(cat /tmp/prompt.txt)`

### Phase 2: REVIEW

| 层级 | 执行者 | 适用级别 |
|------|--------|---------|
| Layer 1 | Claude 初审（安全/风格/逻辑/蓝图合规） | 所有 |
| Layer 2 | Codex `--full-auto` 终审（蓝图 + diff） | L2/L3 |
| 交叉审查 | Codex 审查 Claude 的修正 | L3 可选 |

修复-重审循环：Codex FAIL → Claude 修 → Codex 重审 → 最多 3 轮

退出：✅ Clean pass | 🟡 仅主观建议 | 🛑 3 轮上限/意见矛盾 → 升级用户

详见 [references/review-protocol.md](references/review-protocol.md)

### Phase 3: REPORT

输出结构化报告：

```
最终状态: [✅ / 🟡 / 🛑]
Phase 0: [辩论轮数, 关键决策]
Phase 1: [Codex 调用次数, 修改文件, 变更行数]
Phase 2: [Layer 1 结果, Layer 2 结果, 重审轮数]
剩余事项: [未解决的主观建议]
```

**自动提交**：✅ 时自动 `git commit`（仅暂存任务相关文件）；🛑 时不提交，等用户决定。

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

## 模型策略

**强制 `gpt-5.3-codex`**（`~/.codex/config.toml` 配置 `model_reasoning_effort = "xhigh"`）。ChatGPT 账户下 o3/o4-mini 不可用。

## 关键约束

- **Windows 路径** — `-C` 参数用正斜杠或引号
- **Windows 大文件** — 单次 Codex >300 行触发错误码 206，用两步法（详见 templates-and-faq.md）
- **并发** — 同一时间只运行一个 Codex 实例
- **蓝图纪律** — Phase 1/2 不得静默偏离锁定蓝图
- **升级机制** — 辩论/审查达 3 轮上限或僵局时，必须升级给用户
