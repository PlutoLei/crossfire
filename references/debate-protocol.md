# Phase 0: EXPLORE → PLAN → DEBATE → LOCK 协议

仅 L2/L3 触发。Claude 全自主完成，用户仅在升级时介入。

---

## Step 0a: EXPLORE

目标：理解当前代码状态，收集实现所需的上下文。

**操作清单：**
1. 用 Grep/Glob/Read 定位目标文件、相关模块、已有实现
2. 若项目已索引 GitNexus，用 `query()` 查找相关执行流，`context()` 查看符号的调用关系
3. 识别可复用的函数、工具类、设计模式
4. 记录约束条件（依赖版本、API 限制、Windows 兼容性等）

**产出：** 上下文摘要（涉及文件列表、现有模式、约束条件）

---

## Step 0b: PLAN

目标：基于探索结果，设计实现方案。

**操作清单：**
1. 提出 2-3 种实现方案，各附权衡分析
2. 选择最优方案（标注选择理由）
3. 草拟架构蓝图，格式如下：

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

**产出：** 架构蓝图草案

---

## Step 0c: DEBATE

目标：让 Codex 以异源视角质疑蓝图，发现盲点。

### Codex 质询命令

```bash
codex exec --full-auto -m gpt-5.3-codex -C "<workdir>" "$(cat /tmp/crossfire_debate.txt)"
```

Claude 先将以下提示词写入 `/tmp/crossfire_debate.txt`：

```
You are an architecture critic. Review the following blueprint and return structured feedback.

## Blueprint
<蓝图内容>

## Instructions
1. OBJECTIVE issues (wrong API usage, math errors, missing deps, race conditions):
   cite specific items, explain the correct approach.
2. SUBJECTIVE concerns (design trade-offs, naming, alternative approaches):
   state your recommendation with rationale.
3. Rate confidence: HIGH / MEDIUM / LOW.

## Output Format
### Objective Issues
- [issue]: [explanation] → [fix]

### Subjective Concerns
- [concern]: [recommendation] | [alternative]

### Confidence: [HIGH/MEDIUM/LOW]
### Summary: [1-2 sentences]
```

### 评估规则

| Codex 批评类型 | Claude 的处理 | 消耗辩论轮次？ |
|---------------|-------------|-------------|
| 客观错误（API 误用、数学错误、依赖缺失） | 直接修正蓝图 | 否 |
| 主观建议，Claude 同意 | 修正蓝图并记录理由 | 是 |
| 主观建议，Claude 不同意 | 保留原设计，附反驳理由 | 是 |
| Confidence = HIGH，零 Objective Issues | 锁定蓝图，进入 Step 0d | N/A |

### 后续轮次

修正蓝图后，再次调用 Codex：
```bash
codex exec --full-auto -m gpt-5.3-codex -C "<workdir>" "$(cat /tmp/crossfire_debate_r2.txt)"
```

提示词中包含：修正后的蓝图 + 上一轮的批评摘要 + Claude 的回应。

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
1. 将最终蓝图写入 `/tmp/crossfire_blueprint.md`
2. Phase 1 的 Codex 提示词中原文引用此蓝图
3. Claude 不得在后续阶段静默偏离蓝图（任何偏离需重新进入 DEBATE 或获得用户批准）
