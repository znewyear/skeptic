# skeptics 插件设计文档

> 多 Agent 协作工作流体系——7 角色 + 4 阶段工作流，封装为 Claude Code Plugin

## 概述

**插件名**：skeptics（怀疑论者）

**核心理念**：以批复对抗为核心的多 Agent 协作体系。每个角色的产出都会受到其他角色的审视和质疑，通过流程驱动保证质量。

**设计来源**：[make.md](../../make.md) 核心规范 + `my.md` 原始设计 + 用户实践经验

## 架构

### 角色体系

| 角色 | 类型 | 职责 | 加载方式 |
|------|------|------|---------|
| 主 Agent (Leader) | 主会话 | 需求理解、任务拆解、调度、管控、汇报 | SessionStart 自动注入 |
| 批复角色 (Approval) | Skill | 流程驱动对抗型审视 | `/skeptics:approval` |
| 内容审查 (Reviewer) | Skill | 代码/方案/文档质量审查 | `/skeptics:reviewer` |
| 测试角色 (Tester) | Subagent | 编写并运行测试（独立环境） | Leader 派发 |
| 文档维护 (DocKeeper) | Skill | 统一维护项目文档 | `/skeptics:doc-keeper` |
| 记忆管理 (Memory) | Skill | 管理 FACT.md + JOURNAL + 知识图谱 | `/skeptics:memory` |
| 实施角色 (Worker) | Subagent | 根据任务描述实施具体工作 | Leader 派发 |
| 用户 (Owner) | — | 需求发起、决策、验收 | 对话 |

### 插件结构

```
skeptics/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── package.json
├── README.md
├── skills/
│   ├── using-skeptics/SKILL.md (SessionStart 自动注入)
│   ├── workflow/SKILL.md (四阶段工作流定义)
│   ├── approval/SKILL.md (批复角色)
│   ├── reviewer/SKILL.md (内容审查)
│   ├── tester/SKILL.md (测试角色 Skill 定义)
│   ├── doc-keeper/SKILL.md (文档维护)
│   ├── memory/SKILL.md (记忆管理)
│   ├── tdd/SKILL.md (TDD 方法论)
│   └── init/SKILL.md (项目初始化)
├── hooks/
│   ├── hooks.json (SessionStart hook)
│   └── session-start (bash script)
├── prompts/
│   ├── implementer.md (Worker subagent prompt)
│   └── tester-agent.md (Tester subagent prompt)
└── gemini-extension.json
```

### 四阶段工作流

```
需求讨论 → 任务规划 → 实施调度 → 总结报告
   │           │           │           │
   ▼           ▼           ▼           ▼
Approval    Approval   三重 loop    DocKeeper
loop        loop       (审查/测试/ + Memory
(需求修正）  (规划修正)   批复拦截)   + 图谱更新
```

### 三重 Loop 机制（实施调度阶段）

1. **Reviewer Loop**：审查发现问题 → Worker 修复 → 再审查
2. **Tester Loop**：测试失败 → Worker 修复 → 新代码重新跑
3. **Approval Loop**：批复质疑 → 对应角色修正 → 再审视

### 六论知识图谱

| 论域 | 节点类型 | 生命周期 | 用途 |
|------|---------|---------|------|
| 需求论 | 需求、功能点、约束 | proposed → deprecated | 记录所有需求及其状态 |
| 经验论 | 模式、教训、最佳实践 | observed → archived | 归纳实施经验和解决方案 |
| 错误论 | 错误、根因、修复方案 | detected → monitored | 记录错误及其根因 |
| 技术栈论 | 技术、依赖、配置 | introduced → removed | 记录技术栈和依赖关系 |
| 架构论 | 模块、接口、数据流 | designed → removed | 记录系统架构和模块关系 |
| 工具论 | MCP工具、Skill、CLI工具 | discovered → archived | 记录可用的工具和技能 |

### 全局规则

1. **文件锁**：并发 Subagent 禁止修改同一文件，派发前做文件冲突检查
2. **验证新鲜**：所有测试/审查必须重新运行，git HEAD + 时间戳证明
3. **根因优先**：实施问题先定位根因，不胡乱修改
4. **ASCII 可视化**：方案设计必须使用 ASCII 图表
5. **明确交接**：每个步骤的输入/输出/动作/回退条件必须明确

## 与 superpowers 的差异

| 方面 | superpowers | skeptics |
|------|------------|----------|
| 核心 | Skill 链自动触发 | 批复对抗驱动流程 |
| 注入 | "检查是否有匹配 skill" | "你是 Leader，执行工作流" |
| Subagent | 通用 implementer | Tester(受限) + Worker(全工具) |
| 质量 | 两层审查（spec+code） | 三层（审查+测试+批复拦截） |
| 记忆 | FACT.md | 六论知识图谱 |
