---
name: init
description: 项目初始化——CLAUDE.md 追加 + .project/ 脚手架 + memory/ + knowledge/ 目录创建。仅首次使用 skeptics 时执行
---

# 项目初始化

> 一键初始化 skeptics 工作流环境。仅当项目尚未植入 skeptics 时执行一次。

## 执行流程

```
检查 .claude-plugin/plugin.json → 存在则跳过
  │
  ▼
步骤1: CLAUDE.md 追加（不覆盖）
  检查 CLAUDE.md 末尾是否有 skeptics 标记
  无 → 追加 skeptics 段落
  有 → 跳过
  │
  ▼
步骤2: 创建 .project/ 目录结构
  │
  ▼
步骤3: 创建 memory/ + knowledge/ 目录
  │
  ▼
步骤4: 写入初始知识图谱文件
  │
  ▼
完成
```

## 步骤详情

### 步骤1: CLAUDE.md 追加

检查 CLAUDE.md 末尾。若没有 `<!-- skeptics-init -->` 标记则追加：

```markdown
<!-- skeptics-init -->
## skeptics 工作流

本仓库使用 skeptics 插件管理多 Agent 协作工作流。

| 场景 | 操作 |
|------|------|
| 启动完整工作流 | 加载 `agent.md` 或说"启动 agent 工作流" |
| 加载批复角色 | `/skill skeptics:approval` |
| 审查代码质量 | `/skill skeptics:reviewer` |
| 执行测试 | `/skill skeptics:tester` |
| 更新文档 | `/skill skeptics:doc-keeper` |
| 记忆管理 | `/skill skeptics:memory` |

### 工作流文件

```
skills/               ← 角色技能定义（SKILL.md 格式）
├── approval/         ← 批复角色
├── reviewer/         ← 内容审查角色
├── tester/           ← 测试角色
├── doc-keeper/       ← 文档维护角色
├── workflow/         ← 工作流定义
├── memory/           ← 记忆管理
├── tdd/              ← 测试驱动开发
├── init/             ← 项目初始化
└── using-skeptics/   ← 入口启动
prompts/              ← Subagent 调度模板
hooks/                ← 生命周期钩子
.claude-plugin/       ← 插件元数据
```
```

### 步骤2: 创建 .project/ 目录结构

```
.project/
├── requirement.md    ← 需求文档（按 doc-keeper 模板）
├── progress.md       ← 进度文档（按 doc-keeper 模板）
├── task.md           ← 任务文档（按 doc-keeper 模板）
└── learned.md        ← 实施经验文档（按 doc-keeper 模板）
```

四个文件全部使用 `skills/doc-keeper/SKILL.md` 中的对应模板创建空壳。

### 步骤3: 创建 knowledge/ 目录

```
.project/
└── knowledge/
    ├── schema.json    ← 六论 schema 定义
    ├── graph.json     ← 主图谱（空）
    ├── index.json     ← 查询索引（空）
    └── archive.json   ← 归档节点（空）
```

### 步骤4: 写入初始 schema.json

```json
{
  "domains": ["需求论", "经验论", "错误论", "技术栈论", "架构论", "工具论"],
  "node_types": {
    "Requirement": {
      "domain": "需求论",
      "properties": ["title", "description", "priority", "status"],
      "lifecycle": ["proposed", "confirmed", "implemented", "deprecated"]
    },
    "Feature": {
      "domain": "需求论",
      "properties": ["title", "description", "status"],
      "lifecycle": ["proposed", "confirmed", "implemented", "deprecated"]
    },
    "Constraint": {
      "domain": "需求论",
      "properties": ["description", "type"],
      "lifecycle": ["proposed", "confirmed", "deprecated"]
    },
    "Pattern": {
      "domain": "经验论",
      "properties": ["title", "description", "category"],
      "lifecycle": ["observed", "validated", "established", "archived"]
    },
    "Lesson": {
      "domain": "经验论",
      "properties": ["description", "severity"],
      "lifecycle": ["observed", "validated", "established", "archived"]
    },
    "BestPractice": {
      "domain": "经验论",
      "properties": ["title", "description", "category"],
      "lifecycle": ["observed", "validated", "established", "archived"]
    },
    "Error": {
      "domain": "错误论",
      "properties": ["message", "severity", "frequency"],
      "lifecycle": ["detected", "diagnosed", "fixed", "monitored"]
    },
    "RootCause": {
      "domain": "错误论",
      "properties": ["description", "category"],
      "lifecycle": ["detected", "diagnosed", "fixed", "monitored"]
    },
    "Fix": {
      "domain": "错误论",
      "properties": ["description", "verified"],
      "lifecycle": ["detected", "diagnosed", "fixed", "monitored"]
    },
    "Technology": {
      "domain": "技术栈论",
      "properties": ["name", "version", "purpose"],
      "lifecycle": ["introduced", "active", "deprecated", "removed"]
    },
    "Dependency": {
      "domain": "技术栈论",
      "properties": ["name", "version", "type"],
      "lifecycle": ["introduced", "active", "deprecated", "removed"]
    },
    "Config": {
      "domain": "技术栈论",
      "properties": ["key", "value", "scope"],
      "lifecycle": ["introduced", "active", "deprecated", "removed"]
    },
    "Module": {
      "domain": "架构论",
      "properties": ["name", "purpose", "layer"],
      "lifecycle": ["designed", "implemented", "refactored", "removed"]
    },
    "Interface": {
      "domain": "架构论",
      "properties": ["name", "protocol", "version"],
      "lifecycle": ["designed", "implemented", "refactored", "removed"]
    },
    "DataFlow": {
      "domain": "架构论",
      "properties": ["description", "direction", "format"],
      "lifecycle": ["designed", "implemented", "refactored", "removed"]
    },
    "MCPTool": {
      "domain": "工具论",
      "properties": ["name", "purpose", "server"],
      "lifecycle": ["discovered", "configured", "adopted", "archived"]
    },
    "Skill": {
      "domain": "工具论",
      "properties": ["name", "purpose", "plugin"],
      "lifecycle": ["discovered", "configured", "adopted", "archived"]
    },
    "CLITool": {
      "domain": "工具论",
      "properties": ["name", "purpose", "command"],
      "lifecycle": ["discovered", "configured", "adopted", "archived"]
    }
  },
  "relation_types": {
    "DEPENDS_ON": { "source_types": ["Requirement", "Feature"], "target_types": ["Requirement", "Feature"] },
    "IMPLEMENTS": { "source_types": ["Feature"], "target_types": ["Requirement"] },
    "CONSTRAINS": { "source_types": ["Constraint"], "target_types": ["Requirement", "Feature"] },
    "APPLIES_TO": { "source_types": ["Pattern", "Lesson", "BestPractice"], "target_types": ["Module", "Feature"] },
    "PREVENTS": { "source_types": ["Pattern", "Lesson", "BestPractice"], "target_types": ["Error"] },
    "IMPROVES": { "source_types": ["Pattern", "BestPractice"], "target_types": ["Module", "Interface"] },
    "CAUSED_BY": { "source_types": ["Error"], "target_types": ["RootCause"] },
    "FIXED_BY": { "source_types": ["Error"], "target_types": ["Fix"] },
    "COMPATIBLE_WITH": { "source_types": ["Technology", "Dependency"], "target_types": ["Technology", "Dependency"] },
    "REQUIRES": { "source_types": ["Technology", "Module"], "target_types": ["Dependency", "Technology"] },
    "CONFIGURES": { "source_types": ["Config"], "target_types": ["Technology", "Module"] },
    "CONTAINS": { "source_types": ["Module"], "target_types": ["Module", "Interface", "DataFlow"] },
    "COMMUNICATES_WITH": { "source_types": ["Module", "Interface"], "target_types": ["Module", "Interface"] },
    "FLOWS": { "source_types": ["DataFlow"], "target_types": ["Module", "Interface"] },
    "USED_FOR": { "source_types": ["MCPTool", "Skill", "CLITool"], "target_types": ["Task", "Feature"] },
    "INTEGRATES_WITH": { "source_types": ["MCPTool", "Skill"], "target_types": ["MCPTool", "Platform"] }
  }
}
```

## 幂等性

- 检测到 `.claude-plugin/plugin.json` 已存在 → 跳过全部步骤
- 检测到 CLAUDE.md 中已有 `<!-- skeptics-init -->` → 跳过步骤1
- 检测到 `.project/requirement.md` 已存在 → 跳过步骤2 对应文件
- 检测到 `.project/knowledge/schema.json` 已存在 → 跳过步骤4 对应文件

## 验证清单

- [ ] CLAUDE.md 末尾有 `<!-- skeptics-init -->` 标记
- [ ] `.project/` 下四个文档文件存在
- [ ] `.project/knowledge/` 下四个 JSON 文件存在
- [ ] `schema.json` 包含全部 6 论 18 种节点类型
- [ ] `schema.json` 包含全部 15 种关系类型
