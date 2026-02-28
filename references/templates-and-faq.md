# Crossfire 模板与 FAQ

## 模板详情

### code（默认 L1/L2）
- **提示骨架**：`在 {file} 中实现 {function}，满足 {constraints}`
- **Review 重点**：功能正确性、接口一致性
- **示例**：`/crossfire code: 在 src/utils.py 中添加 parse_date 函数，支持 ISO 8601 和中文日期格式`

### bugfix（默认 L2）
- **提示骨架**：`修复 {file} 中的 {issue}，根因是 {analysis}，测试方法 {method}`
- **Review 重点**：根因是否正确、修复是否完整、有无副作用
- **示例**：`/crossfire bugfix: evaluate.py 的 confusion matrix 颜色映射反了`

### refactor（默认 L2）
- **提示骨架**：`重构 {module}，目标 {improvement}，保持 {interfaces} 不变`
- **Review 重点**：接口兼容性、行为一致性
- **示例**：`/crossfire refactor: 将 merge_kg.py 的去重逻辑抽取为独立模块`

### test（默认 L1）
- **提示骨架**：`为 {file} 编写测试，覆盖 {scenarios}，框架 {pytest/jest}`
- **Review 重点**：覆盖率、边界条件
- **示例**：`/crossfire test: 为 extract_fast.py 的 process_one_paper 添加单元测试`

### review（默认 L1）
- **提示骨架**：`审查 {file} 代码质量，输出到 review-report.md`
- **说明**：此模板让 Codex 做独立 code review，不同于 Phase 2 的内部审查流程
- **示例**：`/crossfire review: BS6207/Assignment4/generate_essay_en.py`

### optimize（默认 L2）
- **提示骨架**：`优化 {file} 的 {metric}，当前瓶颈 {description}`
- **Review 重点**：性能基线对比、无功能回退
- **示例**：`/crossfire optimize: fetch_openreview.py 的批量请求太慢，优化为异步并发`

### architect（默认 L3，新增）
- **提示骨架**：`设计 {module/system} 的架构，满足 {requirements}，约束 {constraints}`
- **说明**：DEBATE 是核心价值，蓝图比代码更重要
- **示例**：`/crossfire architect: 设计 paperbanana 的 figure compositor 模块，支持多面板布局`

---

## 调用语法

```
/crossfire <模板>: <描述>
/crossfire L1: <描述>              # 强制 L1 级别
/crossfire L3: <描述>              # 强制 L3 级别
/crossfire --no-debate <模板>: <描述>    # 跳过 Phase 0
/crossfire --no-audit <模板>: <描述>     # 跳过 Codex 终审
```

---

## Codex CLI 调用参考

Codex CLI 基础格式、权限标志、大提示词处理等详见 `codex-execute` skill。

crossfire 各阶段的权限映射：

| 阶段 | 权限标志 | 说明 |
|------|---------|------|
| Phase 0 DEBATE | `--full-auto` | 只读质询，不修改文件 |
| Phase 1 EXECUTE | `--dangerously-bypass-approvals-and-sandbox` | 需要写入文件 |
| Phase 2 REVIEW | `--full-auto` | 只读审计 |

---

## FAQ

| 问题 | 解决 |
|------|------|
| DEBATE 辩论消耗太多 token | 上限 3 轮，客观错误直接修正不消耗轮次 |
| Codex 批评看起来不对 | Claude 区分客观/主观批评，主观的可以反驳 |
| 修复-重审循环卡住 | 最多 3 轮，超限自动升级给用户 |
| 想跳过 DEBATE | 用 `--no-debate` 标志，或强制 L1 级别 |
| 想跳过 Codex 终审 | 用 `--no-audit` 标志 |
| Codex 超时 | 拆分任务，设置 timeout |
| 输出截断 | 指示 Codex 写入文件而非 stdout |
| 错误码 206（Windows） | 单次 Codex 提示 <300 行/15KB，大内容用临时文件 |
| L1 任务太慢 | 显式指定 `/crossfire L1:` 确保走快速通道 |
