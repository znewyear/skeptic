---
name: memory
description: 记忆管理——管理 FACT.md 持久知识、JOURNAL.jsonl 活动日志、六论知识图谱的读写和流转
---

# 记忆管理

> 管理项目的记忆体系：持久知识读写、活动日志记录、六论知识图谱维护和自动流转。

## 会话初始化（Leader 每次会话开始时执行）

```
读取顺序:
  1. project/.project/requirement.md  → 需求列表和状态
  2. project/.project/progress.md     → 进度百分比和状态
  3. memory/FACT.md                   → 关键事实摘要
  4. project/.project/learned.md "反复出现的问题" → 避免重复犯错
  5. .project/knowledge/index.json    → 高权重知识节点摘要（权重≥3.0）
```

## 六论知识图谱

### 存储结构

```
.project/
├── knowledge/
│   ├── graph.json     ← 主图谱（所有节点 + 关系）
│   ├── schema.json    ← 六论定义（节点类型、关系类型、属性）
│   ├── archive.json   ← 归档节点（冻结 90 天以上）
│   └── index.json     ← 查询索引（按标签、权重排序）
└── memory/
    ├── FACT.md        ← 关键事实摘要（SessionStart 加载用）
    └── JOURNAL.jsonl  ← 活动日志（图谱的输入源）
```

### 六论定义

| 论域 | 节点类型 | 关系 | 生命周期 |
|------|---------|------|---------|
| 需求论 | 需求(Requirement)、功能点(Feature)、约束(Constraint) | DEPENDS_ON、IMPLEMENTS、CONSTRAINS | proposed → confirmed → implemented → deprecated |
| 经验论 | 模式(Pattern)、教训(Lesson)、最佳实践(BestPractice) | APPLIES_TO、PREVENTS、IMPROVES | observed → validated → established → archived |
| 错误论 | 错误(Error)、根因(RootCause)、修复方案(Fix) | CAUSED_BY、FIXED_BY、APPLIES_TO | detected → diagnosed → fixed → monitored |
| 技术栈论 | 技术(Technology)、依赖(Dependency)、配置(Config) | COMPATIBLE_WITH、REQUIRES、CONFIGURES | introduced → active → deprecated → removed |
| 架构论 | 模块(Module)、接口(Interface)、数据流(DataFlow) | CONTAINS、COMMUNICATES_WITH、FLOWS | designed → implemented → refactored → removed |
| 工具论 | MCP工具(MCPTool)、Skill、CLI工具(CLITool) | USED_FOR、INTEGRATES_WITH、REQUIRES | discovered → configured → adopted → archived |

### 生命周期管理

每个节点和关系经历以下阶段：

| 阶段 | 条件 | 行为 |
|------|------|------|
| 创建 | 首次出现的新知识 | 创建节点，初始权重 1.0 |
| 活跃 | 30 天内被查询 ≥2 次 | 权重 +0.1/次查询，上限 5.0 |
| 冻结 | 60 天未被查询 | 标记冻结，不参与自动加载 |
| 归档 | 90 天未被查询 | 移到 archive.json，仅手动可查 |
| 删除 | 180 天未被查询且无关联 | 清理 |
| 成熟 | 权重 ≥3.0 且验证次数 ≥5 | SessionStart 自动注入摘要到 index.json |

### 图存储格式（graph.json）

```json
{
  "nodes": [
    {
      "id": "req-001",
      "type": "Requirement",
      "domain": "需求论",
      "properties": {
        "title": "用户注册",
        "description": "支持邮箱+密码注册",
        "priority": "P0",
        "status": "confirmed"
      },
      "weight": 2.5,
      "created_at": "2026-06-12T10:00:00Z",
      "last_accessed": "2026-06-12T10:30:00Z"
    }
  ],
  "edges": [
    {
      "source": "req-001",
      "target": "feat-001",
      "relation": "IMPLEMENTS",
      "weight": 1.0,
      "created_at": "2026-06-12T10:00:00Z"
    }
  ]
}
```

### schema.json 格式

```json
{
  "domains": ["需求论", "经验论", "错误论", "技术栈论", "架构论", "工具论"],
  "node_types": {
    "Requirement": {
      "domain": "需求论",
      "properties": ["title", "description", "priority", "status"],
      "lifecycle": ["proposed", "confirmed", "implemented", "deprecated"]
    }
  },
  "relation_types": {
    "DEPENDS_ON": { "source_types": ["Requirement"], "target_types": ["Requirement"] },
    "IMPLEMENTS": { "source_types": ["Feature"], "target_types": ["Requirement"] }
  }
}
```

## 记忆流转规则

### 提升条件（learned.md → FACT.md）
- 同一问题描述本质相同的条目累计出现 ≥3 次
- 复制到 FACT.md，标注来源编号和首次出现时间
- 同时在知识图谱中创建对应节点和关系

### 降级条件（FACT.md → learned.md 归档区）
- FACT.md 中某条目超过 6 个月未被任何任务/进度/需求文档引用
- 搜索所有项目文档无匹配后移回

### 跨论关联发现
- 当同一关键词出现在不同论域中时，自动创建跨论关联
- 例如"bcrypt"同时出现在"技术栈论"（依赖）和"经验论"（解决方案记录中）→ 自动创建关联

## 日志记录格式（JOURNAL.jsonl）

每行一个 JSON 对象：

```json
{"ts":"2026-06-12T10:00:00Z","tags":["requirement","confirmed"],"text":"需求R001确认：用户注册功能"}
{"ts":"2026-06-12T10:30:00Z","tags":["task","status-change"],"text":"任务T001状态变更为进行中"}
{"ts":"2026-06-12T11:00:00Z","tags":["error","diagnosed"],"text":"错误E001诊断：密码hash未做，根因为直接存明文"}
```

### 必须记录的事件
- 需求变更（创建/修改/确认/废弃）
- 任务状态变更（开始/完成/阻塞/降级）
- 验收结果（通过/不通过 + 关键问题）
- 重要决策（技术选型/方案变更/架构调整）
- 实施路线变更（需求重排/优先级调整）
- 错误诊断和解决方案
- 新发现的模式和最佳实践

## 更新时机

| 事件 | 更新内容 | 执行者 |
|------|---------|--------|
| 需求确认后 | 写入 requirement.md + 创建需求论节点 | DocKeeper |
| 任务规划后 | 写入 task.md + progress.md | DocKeeper |
| 任务完成时 | 更新进度 + 写入实施经验 | DocKeeper |
| 验收通过时 | 记录验收结论到 learned.md | DocKeeper |
| 总结报告时 | 更新 memory/FACT.md + knowledge/ 图谱 | Memory Skill |
