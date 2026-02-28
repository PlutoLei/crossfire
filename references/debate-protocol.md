# Phase 0: EXPLORE → PLAN → DEBATE → LOCK 协议

仅 L2/L3 触发。**首先 invoke `planning-with-files` skill**，创建 task_plan.md / findings.md / progress.md 进行状态管理。

- **L2（自主模式）：** Claude 全自主完成，用户仅在升级时介入。
- **L3（半交互模式）：** Step 0a/0b invoke `brainstorming` 和 `writing-plans` 让用户参与方案选择，Step 0c/0d 仍自主。

---

## Step 0a: EXPLORE

目标：理解当前代码状态，收集实现所需的上下文。

**操作清单：**
1. 用 Grep/Glob/Read 定位目标文件、相关模块、已有实现
2. 若项目已索引 GitNexus，用 `query()` 查找相关执行流，`context()` 查看符号的调用关系
3. 识别可复用的函数、工具类、设计模式
4. 记录约束条件（依赖版本、API 限制、Windows 兼容性等）
5. **每 2 次查询后将发现写入 findings.md**（planning-with-files 的 2-action rule）

**产出：** 上下文摘要（涉及文件列表、现有模式、约束条件）

---

## Step 0a/0b: L3 交互路径

**仅 L3 触发。** L2 跳过此段，直接执行下方的自主 Step 0b。

1. **invoke `brainstorming` skill** — 交互式探索多方案，用户逐步参与选择
2. **invoke `writing-plans` skill** — 交互式生成结构化蓝图，用户确认后进入 DEBATE

invoke 完成后，将产出写入 task_plan.md，然后跳到 Step 0c DEBATE。

---

## Step 0b: PLAN

目标：基于探索结果，设计实现方案。**（L2 自主模式，L3 已在上方完成）**

**参考 `brainstorming` skill 的多方案原则：**
- 提出 2-3 种实现方案，各附权衡分析
- YAGNI：砍掉非必要功能
- 选择最优方案（标注选择理由）

**参考 `writing-plans` skill 的蓝图格式**草拟架构蓝图：

```markdown
## Architecture Blueprint

### Objective
[一句话目标]

### File Changes
| File | Action | Description |
|------|--------|-------------|
| path/to/file.py | CREATE/MODIFY/DELETE | 具体说明 |

### Design Decisions
1. [决策]: [理由]

### Constraints
- [约束 1]

### Acceptance Criteria
- [ ] [验收标准 1]
```

**产出：** 架构蓝图草案，同步更新 task_plan.md

---

## Step 0c: DEBATE

目标：让 Codex 以异源视角质疑蓝图，发现盲点。

### Codex 质询命令

```bash
codex exec --full-auto -m gpt-5.3-codex -C "<workdir>" "$(cat /tmp/crossfire_debate.txt)"
```

Claude 先将以下提示词写入 `/tmp/crossfire_debate.txt`：

```
You are a senior architect reviewing a proposed design. You have one input:

## Architecture Blueprint
(Read from /tmp/crossfire_blueprint.md)

## Task
Challenge this blueprint from a different perspective:
1. Objective Issues: API misuse, math errors, missing dependencies, incorrect assumptions?
2. Subjective Concerns: alternative approaches, over-engineering, under-engineering?
3. Confidence: HIGH (ready to implement) / MEDIUM (minor concerns) / LOW (major rethink needed)

## Output Format
### Objective Issues
- [issue] → [suggested fix]

### Subjective Concerns
- [concern] → [alternative approach]

### Confidence: [HIGH / MEDIUM / LOW]
### Summary: [1-2 sentences]
```

### 评估规则

| Codex 批评类型 | Claude 的处理 | 消耗辩论轮次？ |
|---------------|-------------|-------------|
| 客观错误（API 误用、数学错误、依赖缺失） | 直接修正蓝图 | 否 |
| 主观建议，Claude 同意 | 修正蓝图并记录理由 | 是 |
| 主观建议，Claude 不同意 | 保留原设计，附反驳理由 | 是 |
| Confidence = HIGH，零 Objective Issues | 锁定蓝图，进入 Step 0d | N/A |

### 退出条件

| 条件 | 结果 |
|------|------|
| Codex 返回 HIGH，无客观问题 | → Step 0d LOCK |
| 完成 3 轮辩论仍无共识 | → 升级给用户 🛑 |
| 同一客观问题重复出现 | → 升级给用户 🛑 |

---

## Step 0d: LOCK

目标：冻结蓝图，确保后续阶段严格执行。

**操作：**
1. 将最终蓝图写入 task_plan.md（利用 planning-with-files 的持久化机制）
2. 同时写入 `/tmp/crossfire_blueprint.md` 供 Codex 引用
3. Phase 1 的 Codex 提示词中原文引用此蓝图
4. Claude 不得在后续阶段静默偏离蓝图（任何偏离需重新进入 DEBATE 或获得用户批准）
