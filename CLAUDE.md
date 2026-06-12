# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概览

多 Agent 协作工作流体系——7 角色 + 4 阶段工作流，用于 vibecoding 场景下的 AI 辅助编程。核心在 `agent.md` 和 `make.md` 中定义。

## 项目结构

```
├── agent.md              ← 主 Agent 入口（加载即启动完整工作流）
├── CLAUDE.md             ← 本文件
├── README.md             ← 仓库说明
├── CHANGELOG.md          ← 版本变更
├── my.md                 ← 原始设计（只读）
├── make.md               ← 核心规范文档（持续维护，权威来源）
├── SOUL.md               ← 主 Agent 人格定义
├── USER.md               ← 用户画像
├── skills/
│   ├── approval.md       ← 批复角色（流程驱动模式）
│   ├── reviewer.md       ← 内容审查角色
│   ├── testing.md        ← 测试角色
│   ├── doc-keeper.md     ← 文档维护角色
│   └── memory.md         ← 记忆管理
├── hooks/                ← 工作流生命周期钩子（结构预留）
├── plugins/              ← MCP 工具与扩展配置（结构预留）
├── memory/               ← 持久知识与活动日志
└── project/.project/     ← 项目文档模板集
```

## 角色体系

| 角色 | 职责 | 边界 |
|------|------|------|
| **主 Agent (Leader)** | 需求理解、任务拆解、调度、管控、汇报 | 不写代码、不写文档、不写测试 |
| **批复角色 (Approver)** | 走流程做评判：提取→对照→产出→通过/不通过 | 只有两种结果，主观不做判断 |
| **实施角色 (Worker)** | 根据任务描述实施具体工作 | 只实施，不具有调度能力 |
| **内容审查角色 (Reviewer)** | 审查代码/方案/文档质量 | 产出可操作的修改意见 |
| **测试角色 (Tester)** | 编写并运行测试 | 产出测试报告 |
| **文档维护角色 (DocKeeper)** | 统一维护项目文档 | 唯一负责写入文档 |
| **用户 (Owner)** | 需求发起、决策、验收 | 最终决定权 |

## 核心工作流

```
用户需求 → [需求讨论] ───批复通过？───→ [任务规划] ───批复通过？───→ [实施调度] ───批复通过？───→ [总结报告]
                │ 否                        │ 否                        │ 否
                └── 修正需求 ←──┘           └── 修正规划 ←──┘           └── 修正实施 ←──┘
```

## 快速使用

| 场景 | 操作 |
|------|------|
| 启动完整工作流 | 加载 `agent.md` 或说"启动 agent 工作流" |
| 批复角色审视 | 加载 `skills/approval.md` |
| 审查代码质量 | 加载 `skills/reviewer.md` |
| 执行测试 | 加载 `skills/testing.md` |
| 更新文档 | 加载 `skills/doc-keeper.md` |
| 查阅完整规范 | 读 `make.md`（持续维护的权威来源） |
