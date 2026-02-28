# Phase 2: 多层 REVIEW 协议

L1 仅执行 Layer 1。L2/L3 执行完整多层审查。

---

## Layer 1: Claude 初审（所有级别）

Codex 写完代码后，Claude 立即执行：

1. **检查 exit code** — 非零则判断是重新执行还是升级
2. **读取所有修改文件** — 用 Read 工具逐一检查
3. **审查要点：** 安全、风格、逻辑、蓝图合规（L2/L3）
4. **决策：** 小问题直接 Edit 修正 | 大问题重新 EXECUTE | 通过进入下一层

**L1 到此结束，直接进入 Phase 3 REPORT。**

---

## Layer 2: Codex 终审（L2/L3）

新的 Codex 会话，接收两个输入：锁定蓝图 + git diff。

### 审计命令

```bash
git diff HEAD > /tmp/crossfire_review_diff.patch
codex exec --full-auto -m gpt-5.3-codex -C "<workdir>" "$(cat /tmp/crossfire_audit.txt)"
```

Claude 先将以下提示词写入 `/tmp/crossfire_audit.txt`：

```
You are a senior code auditor. You have two inputs:

## 1. Approved Architecture Blueprint
(Read from /tmp/crossfire_blueprint.md)

## 2. Implementation Diff
(Read from /tmp/crossfire_review_diff.patch)

## Task
Review implementation against blueprint:
1. Blueprint Compliance: deviations from approved architecture?
2. Correctness: logic errors, off-by-one, null handling, race conditions?
3. Security: injection, unsafe deserialization, credential exposure?
4. Performance: unnecessary allocations, N+1 queries, missing caching?
5. Completeness: acceptance criteria not met?

## Output Format
### Verdict: [PASS / FAIL]

### Objective Issues (must fix)
- [file:line] [issue] → [suggested fix]

### Subjective Suggestions (optional)
- [file:line] [suggestion]

### Blueprint Compliance: [FULL / PARTIAL / DEVIATION]
### Summary: [1-2 sentences]
```

大 diff（>300 行）时，指示 Codex 从文件读取而非内嵌。

---

## 修复-重审循环（L2/L3）

```
Codex 终审 FAIL
  ├─ Claude 读取 Objective Issues → Edit 修复
  ├─ 生成新 git diff → 再次调用 Codex 终审
  ├─ PASS → Phase 3 ✅
  ├─ 轮次 < 3 → 继续循环
  └─ 轮次 = 3 → 升级给用户 🛑
```

---

## 交叉审查（L3 必选，L2 可选）

Claude 修正代码后，额外调用 Codex 审查 Claude 的修正本身：

```bash
git diff HEAD~1..HEAD > /tmp/crossfire_fixes.patch
codex exec --full-auto -m gpt-5.3-codex -C "<workdir>" "$(cat /tmp/crossfire_crossreview.txt)"
```

输出：每个修正标记为 RESOLVED / PARTIALLY_RESOLVED / NEW_ISSUE_INTRODUCED

---

## 退出条件

| 条件 | 结果 | 符号 |
|------|------|------|
| Layer 2 PASS，无客观问题 | 成功，进入 Phase 3 | ✅ |
| 仅剩主观建议 | 记录建议，进入 Phase 3 | 🟡 |
| 3 轮重审仍 FAIL | 升级给用户 | 🛑 |
| Claude 和 Codex 对同一问题意见矛盾 | 升级给用户 | 🛑 |
