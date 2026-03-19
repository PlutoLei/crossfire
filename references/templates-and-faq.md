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

### audit（默认 L2，双源并行审核）
- **触发词**：`审核代码`、`audit`
- **说明**：Claude L1 审查 + code-reviewer agent **并行执行**，合并发现后由 Claude 终审裁定。跳过 Phase 0/1，直接进入双源 Phase 2。适用于已完成代码变更后的质量把关
- **Pipeline**：见 [review-protocol.md](review-protocol.md) 的 "Standalone Audit Pipeline" 章节
- **示例**：`/crossfire audit: 审核 TextMamba3D 所有代码变更`

### optimize（默认 L2）
- **提示骨架**：`优化 {file} 的 {metric}，当前瓶颈 {description}`
- **Review 重点**：性能基线对比、无功能回退
- **示例**：`/crossfire optimize: fetch_openreview.py 的批量请求太慢，优化为异步并发`

### architect（默认 L3，新增）
- **提示骨架**：`设计 {module/system} 的架构，满足 {requirements}，约束 {constraints}`
- **说明**：DEBATE 是核心价值，蓝图比代码更重要
- **示例**：`/crossfire architect: 设计 paperbanana 的 figure compositor 模块，支持多面板布局`

### research（默认 L2 + inject-plan）
- **提示骨架**：`按照 architecture_proposal.md 实现 {description}`
- **说明**：专为 `idea2plan` → `crossfire` 串联设计。自动检测项目目录中的研究产出（`architecture_proposal.md` + `dev_plan.md` + `research_summary.md`），Phase 0 DEBATE 对比战术蓝图与研究架构的一致性
- **示例**：`/crossfire research: 按照 architecture_proposal.md 实现 TextMamba3D 文本注入修复`

---

## 调用语法

```
/crossfire <模板>: <描述>
/crossfire L1: <描述>              # 强制 L1 级别
/crossfire L3: <描述>              # 强制 L3 级别
/crossfire --no-debate <模板>: <描述>    # 跳过 Phase 0
/crossfire --no-audit <模板>: <描述>     # 跳过 Codex 终审
/crossfire --inject-plan . L2: <描述>   # 注入研究产出作为蓝图基础
/crossfire research: <描述>             # research 模板（自动 inject-plan）
```

---

## Codex CLI 调用参考

Codex CLI 基础调用格式和权限标志如下：

crossfire 各阶段的权限映射：

| 阶段 | 权限标志 | 说明 |
|------|---------|------|
| Phase 0 DEBATE | `--full-auto` | 只读质询，不修改文件 |
| Phase 1 EXECUTE | `--dangerously-bypass-approvals-and-sandbox` | 需要写入文件 |
| Phase 2 REVIEW | `--full-auto` | 只读审计 |

### Windows 防护层（强制执行）

所有 Codex 调用（DEBATE / EXECUTE / REVIEW）构建 prompt 前必须执行：

**Step A — 路径归一化：** 所有注入 prompt 的路径统一转正斜杠。

```
-C "E:\VSCode_Project\TextMamba3D"  →  -C "E:/VSCode_Project/TextMamba3D"
.crossfire\blueprint.md             →  .crossfire/blueprint.md
```

**Step B — `cd` 注入：** 每个 Codex prompt 第一行固定为：

```
首先执行 cd E:/VSCode_Project/TextMamba3D，后续所有文件操作使用绝对路径。
```

路径取自 `-C` 参数，已经过 Step A 归一化。`cd` 与 `-C` 是有意冗余——`-C` 是 CLI 层面，`cd` 是 shell 层面保底，两者必须同时存在。

**Step C — 蓝图内嵌（仅 Phase 1 EXECUTE）：**

| 蓝图大小 | 策略 |
|---------|------|
| ≤ 200 行 | 内容直接嵌入 prompt（减少 Codex 文件 I/O） |
| > 200 行 | 指示 Codex 读 `.crossfire/blueprint.md` |

200 行阈值是 prompt 管理的经验值——超过此长度直接嵌入会膨胀 prompt、降低 Codex 对任务指令的注意力。

**Step D — 产出完整性兜底（仅 Phase 1 EXECUTE）：**

Codex EXECUTE 完成后，对照蓝图 File Changes 表逐文件检查：

| 检测 | 判定方式 |
|------|---------|
| 文件缺失 | 蓝图中 CREATE 的文件在磁盘上不存在 |
| 文件截断 | 缺少闭合语法（Python 缩进 / JS `}`）或末尾语句不完整 |

触发条件：exit code 非零，或上述检测发现缺失/截断。可能原因包括超时、网络中断、模型异常等。处理：检查 Codex stdout 提取实现意图 → Claude 用 Write 工具完整覆盖该文件 → 继续剩余文件。每个文件独立判定，L3 多步 EXECUTE 中仅接管失败文件。

> **Shell 上下文：** crossfire 所有 Codex 调用通过 Claude Code 的 Bash 工具执行（bash shell，非 PowerShell），`$(cat ...)` 命令替换可用。

---

## L3 多步 EXECUTE 规则

L3 级别的 Phase 1 支持将蓝图拆分为多步 Codex 调用：

1. **拆分规则** — 按蓝图 File Changes 表的模块边界拆分，每步对应一个逻辑模块（如"创建数据模型" → "实现业务逻辑" → "添加测试"）
2. **步骤排序** — 按依赖关系排序：被依赖的模块先执行，依赖方后执行
3. **中间产物** — 每步完成后 Claude 快速检查 exit code 和文件完整性，确认后再执行下一步
4. **中间失败** — 某步 exit code 非零或产出不完整时：
   - 小问题（缺少 import、语法错误）→ Claude 直接 Edit 修正后继续
   - 大问题（方向性错误）→ 仅重新执行该步，不回退已完成的步骤
   - 同一步失败 2 次 → 升级给用户

---

## FAQ

| 问题 | 解决 |
|------|------|
| DEBATE 辩论消耗太多 token | 上限 3 轮，客观错误直接修正不消耗轮次 |
| Codex 批评看起来不对 | Claude 区分客观/主观批评，主观的可以反驳 |
| 修复-重审循环卡住 | 最多 3 轮，超限自动升级给用户 |
| 想跳过 DEBATE | 用 `--no-debate` 标志，或强制 L1 级别 |
| 想跳过 Codex 终审 | 用 `--no-audit` 标志 |
| Codex 超时 | 不消耗修复轮次。Claude 拆分任务后重试；同一子任务超时 2 次则升级用户。用 Bash `timeout` 命令控制 |
| 输出截断 | 指示 Codex 写入文件而非 stdout |
| Codex 产出不完整 | 超时、网络中断或异常退出均可能导致。Step D（产出完整性兜底）自动检测并由 Claude 接管修复。详见上方「Windows 防护层」 |
| L1 任务太慢 | 显式指定 `/crossfire L1:` 确保走快速通道 |
| idea2plan 产出如何传给 crossfire？ | 确保 `architecture_proposal.md` + `dev_plan.md` 在项目目录，crossfire 自动检测；或用 `--inject-plan <dir>` 显式指定；或用 `research` 模板 |
| 仅有 research_summary.md 怎么办？ | crossfire 将其作为 EXPLORE/DEBATE 的额外上下文，Phase 0 正常完整运行 |
| inject-plan 模式下 DEBATE 有什么不同？ | Codex 额外检查战术蓝图与研究架构的一致性（Architecture Alignment），偏离需有理由 |
| audit 和 review 有什么区别？ | `review` 是 L1 单源 Codex 快审；`audit` 是 L2 双源并行（Claude + code-reviewer agent）+ 客观终审 |
| audit 终审降级了很多发现怎么办？ | 正常，终审的目的就是过滤过度谨慎的建议，只修真正有价值的问题 |
| 想跳过 audit 终审直接全修？ | 不推荐。终审防止过度工程化。如确需全修，用标准 Phase 2 流程 |
| 各级别典型耗时？ | L1: ~1-2 min（1 次 Codex）；L2: ~5-10 min（DEBATE 1-2 轮 + EXECUTE + REVIEW 1-2 轮）；L3: ~10-20 min（含多步 EXECUTE + 交叉审查） |
| 为什么不用单轮 DEBATE？ | 实际执行数据显示 DEBATE 通常 1-2 轮收敛：第 1 轮发现问题 → 修复 → 第 2 轮确认。3 轮上限是安全网，极少触发 |
| 项目没有测试套件，audit Phase E 怎么办？ | 降级为手动验证：检查修改文件语法（`python -c "import file"`）、import 正确性、关键函数可调用。报告中标注"未通过自动测试验证" |
