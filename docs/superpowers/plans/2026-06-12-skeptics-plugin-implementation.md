# skeptics Plugin 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use skeptics:tdd for each task. Steps use checkbox (`- [ ]`) syntax.

**Goal:** 将现有 agent 协作工作流体系封装为标准 Claude Code Plugin，新增 TDD/知识图谱/三重 loop 机制

**Architecture:** 直接在当前项目 `/mnt/f/projects/AI/agent/` 中改造。现有 skills/ 目录的 .md 文件转为 skills/<name>/SKILL.md 格式。新增 hooks/、prompts/、六论知识图谱结构。

**Tech Stack:** Markdown (SKILL.md format), JSON (plugin manifest), Bash (hooks), JSON (knowledge graph)

---

### Task 1: 插件清单文件

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`
- Create: `package.json`

- [ ] **Step 1: 创建 .claude-plugin/plugin.json**

写入：
```json
{
  "name": "skeptics",
  "description": "多 Agent 协作工作流体系——批复对抗驱动，7角色+4阶段+三重Loop",
  "version": "0.1.0",
  "author": {
    "name": "NewYear"
  },
  "homepage": "https://github.com/NewYear/skeptics",
  "license": "MIT",
  "keywords": [
    "multi-agent",
    "workflow",
    "approval",
    "review",
    "collaboration"
  ]
}
```

- [ ] **Step 2: 创建 .claude-plugin/marketplace.json**

```json
{
  "name": "skeptics-dev",
  "description": "Development marketplace for skeptics workflow plugin",
  "plugins": [
    {
      "name": "skeptics",
      "description": "多 Agent 协作工作流体系——批复对抗驱动",
      "version": "0.1.0",
      "source": "./",
      "author": { "name": "NewYear" }
    }
  ]
}
```

- [ ] **Step 3: 创建 package.json**

```json
{
  "name": "skeptics",
  "version": "0.1.0"
}
```

- [ ] **Step 4: 验证**

```bash
ls -la .claude-plugin/plugin.json .claude-plugin/marketplace.json package.json
```
预期：3个文件都存在

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/ package.json
git commit -m "feat: skeptics 插件清单文件初始化"
```

---

### Task 2: SessionStart Hook 系统（自动注入核心）

**Files:**
- Create: `hooks/hooks.json`
- Create: `hooks/session-start`

- [ ] **Step 1: 创建 hooks/hooks.json**

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/session-start\"",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: 创建 hooks/session-start（bash 注入脚本）**

功能：读取 skills/using-skeptics/SKILL.md，以 JSON 格式注入到系统提示

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"

using_skeptics_content=$(cat "${PLUGIN_ROOT}/skills/using-skeptics/SKILL.md" 2>&1 || echo "")

escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"
    s="${s//\"/\\\"}"
    s="${s//$'\n'/\\n}"
    s="${s//$'\r'/\\r}"
    s="${s//$'\t'/\\t}"
    printf '%s' "$s"
}

escaped=$(escape_for_json "$using_skeptics_content")

session_context="<EXTREMELY_IMPORTANT>\nYou have skeptics superpowers.\n\n**Below is the full content of your 'skeptics:using-skeptics' skill:**\n\n${escaped}\n</EXTREMELY_IMPORTANT>"

if [ -n "${CLAUDE_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$session_context"
else
  printf '{\n  "additionalContext": "%s"\n}\n' "$session_context"
fi
exit 0
```

- [ ] **Step 3: 设置为可执行**

```bash
chmod +x hooks/session-start
```

- [ ] **Step 4: 验证**

```bash
ls -la hooks/hooks.json hooks/session-start
file hooks/session-start | grep "shell script"
```
预期：hooks.json 存在，session-start 是可执行 shell 脚本

- [ ] **Step 5: Commit**

```bash
git add hooks/
git commit -m "feat: 添加 SessionStart hook 自动注入机制"
```

---

### Task 3: using-skeptics 引导 Skill（核心注入内容）

**Files:**
- Create: `skills/using-skeptics/SKILL.md`

- [ ] **Step 1: 创建 skills/using-skeptics 目录**

```bash
mkdir -p skills/using-skeptics
```

- [ ] **Step 2: 创建 SKILL.md**

内容包含：
1. 声明：当前 Claude 会话拥有 skeptics 超能力
2. 检查项目是否初始化了 skeptics（检测 .project/ 目录或 CLAUDE.md 中的 skeptics 段落）
3. 如已初始化：激活 Leader 模式，遵循四阶段工作流
4. 指令优先级：用户指令 > skeptics 技能 > 默认行为
5. 全局规则：文件锁、验证新鲜、根因优先、ASCII 可视化、明确交接
6. 技能调用方式：使用 Skill 工具加载对应角色 skill

```markdown
---
name: using-skeptics
description: 会话启动时自动注入——激活 skeptics 多 Agent 协作工作流体系。检测项目是否初始化，决定是否进入 Leader 模式
---

<EXTREMELY_IMPORTANT>
你有 skeptics 超能力。

如果你是作为 Subagent 被派发的，跳过本技能。
</EXTREMELY_IMPORTANT>

## skeptics 工作流系统

你是一套多 Agent 协作工作流体系的主 Agent（Leader）。你的核心职责是调度和管控，不直接实施任何具体工作。

### 初始化检测

检查当前项目是否已初始化 skeptics：
1. 是否存在 `.project/` 目录？
2. `CLAUDE.md` 中是否包含 "skeptics" 段落？

两个条件任一成立 → 激活 Leader 模式
都不成立 → 行为正常（不干预）

### Leader 模式行为

当 Leader 模式激活时：
- 你不再直接写代码、写文档、写测试
- 你按照四阶段工作流推进：需求讨论 → 任务规划 → 实施调度 → 总结报告
- 每个阶段需要调度对应角色 skill 执行审查/批复
- 实施和测试工作派发给 Subagent 执行

### 指令优先级

1. 用户的明确指令（最高）
2. skeptics 技能指令
3. 默认系统提示

### 全局规则

1. **文件锁**：并发 Subagent 禁止修改同一文件。派发 Worker 前检查任务涉及的文件是否有冲突
2. **验证新鲜**：所有测试/审查结果必须包含 git HEAD hash + 时间戳。代码变更后过往结果作废
3. **根因优先**：实施遇到问题先定位根因，不胡乱修改。定位不了则搜索 GitHub/Google/Stack Overflow
4. **ASCII 可视化**：所有方案设计、流程说明必须使用 ASCII 图表
5. **明确交接**：每个步骤必须有明确的输入、输出、动作、回退条件

### 技能调度

| 场景 | 调度方式 |
|------|---------|
| 需要对抗型审视 | 加载 skill `skills/approval/SKILL.md` |
| 需要代码/方案审查 | 加载 skill `skills/reviewer/SKILL.md` |
| 需要编写执行测试 | 派发 Tester Subagent（使用 prompts/tester-agent.md） |
| 需要更新文档 | 加载 skill `skills/doc-keeper/SKILL.md` |
| 需要记忆管理 | 加载 skill `skills/memory/SKILL.md` |
| 需要实施任务 | 派发 Worker Subagent（使用 prompts/implementer.md） |
| 需要初始化项目 | 加载 skill `skills/init/SKILL.md` |
```

- [ ] **Step 3: 验证**

```bash
ls -la skills/using-skeptics/SKILL.md
head -5 skills/using-skeptics/SKILL.md
```
预期：文件存在，包含 frontmatter

- [ ] **Step 4: Commit**

```bash
git add skills/using-skeptics/
git commit -m "feat: 添加 using-skeptics 引导技能（SessionStart 注入内容）"
```

---

### Task 4: workflow 工作流 Skill

**Files:**
- Create: `skills/workflow/SKILL.md`

- [ ] **Step 1: 创建目录**

```bash
mkdir -p skills/workflow
```

- [ ] **Step 2: 创建 SKILL.md**

从现有 `agent.md` 迁移并增强，包含：
1. 四阶段工作流完整定义
2. 每个阶段的执行步骤、输入输出、回退条件
3. 三重 Loop 机制（Reviewer ↔ Worker / Tester ↔ Worker / Approval 拦截）
4. 全局规则的执行方式
5. ASCII 流程图展示总体工作流

```markdown
---
name: workflow
description: 四阶段工作流完整定义——需求讨论→任务规划→实施调度→总结报告。包含三重Loop机制和全局规则
---

# 四阶段工作流

> 你是主 Agent（Leader），不直接实施，你的职责是调度、管控和汇报。

## 工作流总览

```
用户需求 → [需求讨论] → Approval审视 → 通过？ → [任务规划] → Approval审视 → 通过？ → [实施调度] → 三重Loop → 通过？ → [总结报告]
                │ 否                     │ 否                        │ 否
                └── 修正需求 ←──┘        └── 修正规划 ←──┘           └── 返回修正 ←──┘
```

## 第一阶段：需求讨论

### 流程
1. 用户提出需求（自然语言）
2. Leader 分析需求，产出结构化需求理解文档
   - 提取关键词和功能点
   - 识别模糊点和风险
3. **调度 Approval Skill** 审视需求理解
   - Approval 执行6项检查：覆盖度/歧义/可执行性/隐含需求/技术可行性
   - 产出批复报告
4. 有 ❌ 项 → Leader 修正理解 → 再调 Approval（loop）
5. 无 ❌ 项 → 需求确认 → 调度 DocKeeper 写 requirement.md → 向用户汇报

### 交接物
| 步骤 | 执行者 | 输入 | 动作 | 产出 |
|------|--------|------|------|------|
| 1.1 | Leader | 用户需求文本 | 提取分析 | 需求理解文档 |
| 1.2 | Approval | 需求理解+用户原文 | 6项检查清单 | 批复报告 |
| 1.3 | DocKeeper | 确认的需求 | 写 requirement.md | 更新后的文档 |

### 回退条件
批复报告有 ❌ 项 → 回退步骤 1.1，Leader 修正需求理解

## 第二阶段：任务规划

### 流程
1. 将每个需求拆分为 1-N 个可执行任务
2. 每个任务标注：ID/标题/目标/分类/优先级/难度/涉及文件/前置依赖/技术方案/不做什么/验证标准
3. ≥⭐⭐⭐ 的难题任务必须附带突破计划（调研→POC→实施）
4. 检查文件冲突：确保并行任务不修改同一文件
5. **调度 Approval Skill** 审视任务规划
6. 有 ❌ 项 → Leader 修正 → 再调 Approval（loop）
7. 无 ❌ 项 → 调度 DocKeeper 写 task.md + progress.md → 向用户汇报

### 任务描述模板
```
任务ID: T001
标题: [功能名称]
实现目标: [一句话描述]
分类: 新增/修复/优化/重构/配置/文档
优先级: P0/P1/P2/P3
难度: ⭐~⭐⭐⭐⭐⭐
涉及文件:
  创建: [路径]
  修改: [路径:行号]
  不修改: [路径]
前置依赖: [依赖的任务ID]
技术方案: [步骤化的实施方案]
不做什么: [明确排除的范围]
验证标准: [可观测的验收条件]
```

### 回退条件
批复报告有 ❌ 项 → 回退步骤 1，Leader 修正任务规划

## 第三阶段：实施调度

### 核心规则

**规则1 - 文件锁**：并发 Subagent 禁止修改同一文件
```
派发 Worker 前：
  1. 列出本任务的所有文件
  2. 与当前运行中的 Worker 文件清单交叉检查
  3. 有冲突 → 串行化；无冲突 → 注册文件锁
  释放条件：Worker 完成（成功/失败）
```

**规则2 - 验证新鲜**：测试/审查结果必须包含 git HEAD + 时间戳
```
代码变更后 → 过往结果全部作废 → 必须重新运行
不接受"之前跑过没问题"的表述
```

**规则3 - 根因优先**：实施问题先定位根因
```
问题出现 → 收集完整错误信息 → 分析类型(逻辑/集成/环境)
  → 能定位根因？→ 修复 + 记录
  → 不能定位？→ 搜索 GitHub/Google/Stack Overflow
  → 还不行？→ 上报 Leader（含全部调查过程）
禁止无头绪地反复修改
```

### 实施调度流程

```
┌─ 按 parallel_groups 分批并发 ──────────────────────┐
│                                                       │
│  ┌─ Worker Subagent 实施 ───────────────────────┐     │
│  │ 1. 确认任务理解                                │     │
│  │ 2. 检查环境就绪                                │     │
│  │ 3. 执行 TDD（RED→GREEN→REFACTOR）              │     │
│  │ 4. 遇到问题找根因                              │     │
│  │ 5. 返回实施结果 + 测试报告 + git HEAD          │     │
│  └──────────────────┬─────────────────────────┘     │
│                      │                                │
│  ┌─ Reviewer 审查 ←──有问题→Worker修复→再审查 (loop) ┐│
│  │ 检查: 代码质量/设计/完整性/安全/性能/可维护性/边界  ││
│  └──────────────────┬─────────────────────────┘     │
│                      │ 通过                            │
│  ┌─ Tester 测试 ←───有问题→Worker修复→再测试 (loop)  ┐│
│  │ 测试类型: 单元/集成/边界                          ││
│  │ 必须新鲜运行（git HEAD + 时间戳）                  ││
│  └──────────────────┬─────────────────────────┘     │
│                      │ 通过                            │
│  ┌─ Approval 拦截 ←──质疑→对应角色修正→再审视 (loop) ┐│
│  │ 审视: 审查结论可信度/测试结论可信度/冰山追问        ││
│  │ 连续2次❌→难度+1星→Leader重新评估                 ││
│  └──────────────────┬─────────────────────────┘     │
│                      │ 批准                            │
│  还有下一批？→ 是 → 回到分批并发 (loop)                 │
│  全部完成 → 进入第四阶段                              │
└─────────────────────────────────────────────────────┘
```

### 并行分组规则
1. 无文件冲突的任务可以并行
2. 有前置依赖的任务必须等依赖完成
3. 同模块的任务建议串行（避免逻辑冲突）

### 回退条件
- Reviewer 审查 FAIL → 回 Worker 修复 → 再审查
- Tester 测试 FAIL → 回 Worker 修复 → 再测试（旧结果作废）
- Approval 质疑 → 对应角色补充/修正 → 再审视
- 连续 2 次 ❌ → 难度 +1，Leader 重新评估方案

## 第四阶段：总结报告

### 流程
1. Leader 汇总所有任务结果
2. 生成总结报告（任务概览/问题记录/技术方案/知识图谱更新）
3. 调度 DocKeeper 更新 task.md → progress.md → learned.md → requirement.md
4. 调度 Memory Skill 更新知识图谱
5. 向用户汇报完成

### 总结报告模板
```
### 任务概览
| 任务ID | 标题 | 状态 | 实施者 | 审查结果 | 测试结果 | 问题记录 |

### 问题记录
| 类型 | 描述 | 阶段 | 根因 | 方案 | 关联任务 |

### 技术方案
| 方案 | 任务 | 选择理由 | 替代方案 |

### 知识图谱更新
| 论域 | 新增节点 | 关系更新 | 生命周期变更 |
```
```

- [ ] **Step 3: 验证**

```bash
ls -la skills/workflow/SKILL.md
```
预期：文件存在

- [ ] **Step 4: Commit**

```bash
git add skills/workflow/
git commit -m "feat: 添加 workflow 工作流 Skill（含三重Loop和全局规则）"
```

---

### Task 5: approval 批复 Skill（增强版）

**Files:**
- Modify: `skills/approval.md` → 作为参考保留
- Create: `skills/approval/SKILL.md`

- [ ] **Step 1: 创建目录**

```bash
mkdir -p skills/approval
```

- [ ] **Step 2: 创建 SKILL.md**

从现有 `skills/approval.md` 迁移并增强：
- 统一执行流程（提取→对照→产出三段式）
- 需求正确性批复（覆盖度/歧义/可执行性/隐含需求）
- 需求完整性批复（错误处理/校验/权限/并发/持久化/日志）
- 代码规范批复（命名/轮子/空代码/硬编码/注释）
- 技术方案批复（过度设计/设计不足/框架适配/安全/性能/边界/错误处理/冗余/SOLID）
- 测试验证批复（覆盖真实/结果可信）
- 实施结果批复（实际执行/方案一致）
- 新增：PUA 对抗追问（冰山追问/逼问依据/偷懒识别/反转假设）
- 新增：难度升级机制（L0-L4）

- [ ] **Step 3: 验证**

```bash
ls -la skills/approval/SKILL.md
grep -c "不通过条件" skills/approval/SKILL.md
```
预期：文件存在，包含多个"不通过条件"定义

- [ ] **Step 4: Commit**

```bash
git add skills/approval/
git commit -m "feat: 增强 approval 批复 Skill（新增PUA对抗追问+难度升级）"
```

---

### Task 6: reviewer 审查 + tester 测试 + doc-keeper 文档 Skill

**Files:**
- Create: `skills/reviewer/SKILL.md`, `skills/tester/SKILL.md`, `skills/doc-keeper/SKILL.md`

- [ ] **Step 1: 创建目录**

```bash
mkdir -p skills/reviewer skills/tester skills/doc-keeper
```

- [ ] **Step 2: 创建 skills/reviewer/SKILL.md**

从 `skills/reviewer.md` 迁移，增强检查维度明细：
- 代码质量：空代码/死代码/命名可读性
- 设计合理性：过度设计/设计不足/框架适配
- 完整性：任务覆盖/未完成标记
- 安全性：SQL注入/XSS/鉴权/敏感信息
- 性能：N+1/全量查询/资源泄漏
- 可维护性：函数长度/嵌套深度
- 审查报告格式

- [ ] **Step 3: 创建 skills/tester/SKILL.md**

从 `skills/testing.md` 迁移：
- 测试类型与覆盖标准（单元/集成/E2E/边界/回归）
- 必须覆盖的用例分类
- 测试执行原则（新鲜运行、断言≥2、无依赖、mock合理）
- 测试报告模板
- 新增：验证新鲜的规则（git HEAD + 时间戳）

- [ ] **Step 4: 创建 skills/doc-keeper/SKILL.md**

从 `skills/doc-keeper.md` 迁移：
- 文档分类（需求/进度/任务/经验）
- 写作规范（中文优先/表格优先/ASCII图表）
- 4种文档模板
- 更新流程规范

- [ ] **Step 5: Commit**

```bash
git add skills/reviewer/ skills/tester/ skills/doc-keeper/
git commit -m "feat: 迁移 reviewer/tester/doc-keeper 为 SKILL.md 格式"
```

---

### Task 7: memory 记忆管理 + tdd 方法论 Skill

**Files:**
- Create: `skills/memory/SKILL.md`
- Create: `skills/tdd/SKILL.md`

- [ ] **Step 1: 创建目录**

```bash
mkdir -p skills/memory skills/tdd
```

- [ ] **Step 2: 创建 skills/memory/SKILL.md**

增强版记忆管理，新增六论知识图谱定义：
- 记忆体系表（需求/进度/任务/纠错/报错）
- 会话初始化流程
- 六论知识图谱定义（schema + 节点 + 关系 + 生命周期）
- 记忆流转规则（提升/降级）
- JOURNAL.jsonl 日志记录

- [ ] **Step 3: 创建 skills/tdd/SKILL.md**

TDD 方法论定义：
- 红-绿-重构循环
- 核心铁律：无失败测试不写代码
- 验证 RED（看它失败）→ GREEN（最小实现）→ REFACTOR（测试保护下优化）
- 常见借口与事实对照表
- 与 skeptics 三层验证体系的关系

- [ ] **Step 4: Commit**

```bash
git add skills/memory/ skills/tdd/
git commit -m "feat: 添加 memory（六论图谱）和 tdd 方法论 Skill"
```

---

### Task 8: init 项目初始化 + prompts Subagent 模板

**Files:**
- Create: `skills/init/SKILL.md`
- Create: `prompts/implementer.md`
- Create: `prompts/tester-agent.md`

- [ ] **Step 1: 创建目录**

```bash
mkdir -p skills/init prompts
```

- [ ] **Step 2: 创建 skills/init/SKILL.md**

功能：在目标项目中初始化 skeptics 体系
1. 检测 CLAUDE.md 是否已有 skeptics 记录
2. 有 → 验证完整性，缺失则补充
3. 无 → 在原文件末尾追加 skeptics 段落（不覆盖现有内容）
4. 创建 .project/ 目录结构（requirement.md / progress.md / task.md / learned.md）
5. 创建 knowledge/ 目录结构（graph.json / schema.json / archive.json / index.json）
6. 创建 memory/ 目录结构（FACT.md / JOURNAL.jsonl）
7. 通知用户初始化完成

- [ ] **Step 3: 创建 prompts/implementer.md**

Worker Subagent 的完整指令模板：
- 任务理解确认
- 使用 TDD 方式开发
- 文件锁和文件范围约束
- 问题根因分析流程（收集→分析→搜索→上报）
- 产出格式（实施结果、测试报告、git HEAD）
- 自审查要求

- [ ] **Step 4: 创建 prompts/tester-agent.md**

Tester Subagent 的完整指令模板：
- 验证代码是最新版（git HEAD 检查）
- 先跑已有测试
- 补充测试用例（正常/异常/边界/集成）
- 所有测试新鲜运行
- 产出测试报告（总/通过/失败/覆盖/时间戳）

- [ ] **Step 5: Commit**

```bash
git add skills/init/ prompts/
git commit -m "feat: 添加 init 初始化和 prompts Subagent 模板"
```

---

### Task 9: 更新 README 和 CLAUDE.md

**Files:**
- Modify: `README.md`
- Modify: `CLAUDE.md`（添加插件安装说明）

- [ ] **Step 1: 更新 README.md**

新增插件安装说明、角色技能速查表、与旧版本的兼容性说明

- [ ] **Step 2: 更新 CLAUDE.md**

添加插件结构说明和开发说明

- [ ] **Step 3: Commit**

```bash
git add README.md CLAUDE.md
git commit -m "docs: 更新 README 和 CLAUDE.md 为 skeptics 插件格式"
```

---

### Task 10: 添加 gemini-extension.json（跨平台支持）

**Files:**
- Create: `gemini-extension.json`

- [ ] **Step 1: 创建 gemini-extension.json**

```json
{
  "name": "skeptics",
  "description": "多 Agent 协作工作流体系——批复对抗驱动",
  "version": "0.1.0",
  "contextFileName": "CLAUDE.md"
}
```

- [ ] **Step 2: Commit**

```bash
git add gemini-extension.json
git commit -m "feat: 添加 Gemini CLI 跨平台支持"
```
