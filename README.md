# skeptics — 多 Agent 协作工作流体系

> 一套结构化多 AI Agent 协作方案——7 角色 + 4 阶段工作流 + 三重 Loop 验证，用于 vibecoding 场景下的 AI 辅助编程。
>
> 现已封装为 Claude Code 插件（`.claude-plugin/`），支持 SessionStart 自动注入和 Subagent 隔离调度。

## 架构总览

```
┌─────────────┐     调度/汇报       ┌─────────────────┐
│  用户(Owner) │◀──────────────────│  Leader         │
│  需求发起者   │     沟通           │  唯一与用户沟通   │
│  最终决策者   │                    │  不写代码/文档/测试│
└─────────────┘                    └────────┬────────┘
                                            │ 调度
              ┌──────────────────────────────┼──────────────────────────────┐
              ▼                              ▼                              ▼
     ┌──────────────────┐          ┌──────────────────┐          ┌──────────────────┐
     │   Approver       │          │   DocKeeper      │          │   Worker         │
     │   批复角色        │          │   文档维护角色    │          │   实施角色        │
     │   走流程而非做评判 │          │   统一维护文档     │          │   只负责实施       │
     │   结论由流程产出   │          │   唯一负责写入文档  │          │   不具有调度能力    │
     └──────────────────┘          └──────────────────┘          └────────┬─────────┘
                                                                         │ 三重 Loop 验证
                                                                         ▼
                                                               ┌──────────────────────┐
                                             ┌────────────────│      Reviewer        │◀──┐
                                             │                │  内容审查角色          │   │
                                             │   Loop 1       │  7 维度质量审查        │   │
                                             │                └──────────┬───────────┘   │
                                             │                           ▼               │
                                             │                ┌──────────────────────┐   │
                                             │                │      Tester          │   │
                                             │   Loop 2       │   测试角色            │───┘
                                             │                │   5 类型测试覆盖       │
                                             │                └──────────┬───────────┘
                                             │                           ▼
                                             │                ┌──────────────────────┐
                                             └───────────────│      Approver        │
                                                                  批复角色审视验收结论
                                                 Loop 3         只有通过/不通过
                                                 └──────────────────────────────────────┘
```

## 7 角色速览

| 角色 | 职责 | 边界 |
|------|------|------|
| **Leader** | 需求理解、任务拆解、调度、管控、汇报 | 不写代码、不写文档、不写测试 |
| **Approver** | 走流程做评判：提取→对照→产出→通过/不通过 | 只有通过/不通过两种结果 |
| **Worker** | 实施具体工作（代码/配置/调试） | 只负责实施，不具有调度能力 |
| **Reviewer** | 审查代码/方案/文档质量 | 产出可操作的修改意见 |
| **Tester** | 编写并运行测试 | 产出测试报告 |
| **DocKeeper** | 统一维护项目文档 | 唯一负责写入文档 |
| **Owner** | 需求发起、决策、验收 | 最终决定权 |

## 四阶段工作流

```
用户需求 → [需求讨论] ───批复通过？───→ [任务规划] ───批复通过？───→ [实施调度] ───批复通过？───→ [总结报告]
                │ 否                        │ 否                        │ 否
                └── 修正需求 ←──┘           └── 修正规划 ←──┘           └── 修正实施 ←──┘
```

每阶段末由 Approver 做批复拦截。不通过则返回修正，通过则进入下一阶段。

## 插件结构

```
.claude-plugin/
├── plugin.json              ← 插件元数据
└── marketplace.json         ← 市场发布配置
skills/                      ← 角色技能定义（SKILL.md 格式）
├── using-skeptics/          ← 入口启动（SessionStart 自动注入）
├── workflow/                ← 四阶段工作流定义
├── approval/                ← 批复角色
├── reviewer/                ← 内容审查角色
├── tester/                  ← 测试角色
├── doc-keeper/              ← 文档维护角色
├── memory/                  ← 六论知识图谱管理
├── tdd/                     ← 测试驱动开发方法论
└── init/                    ← 项目初始化
prompts/                     ← Subagent 调度模板
├── implementer.md           ← Worker Subagent 指令
└── tester-agent.md          ← Tester Subagent 指令
hooks/                       ← 生命周期钩子
├── hooks.json               ← 钩子注册
└── session-start            ← SessionStart bash 脚本
package.json                 ← npm 包元数据
```

## 快速开始

### 方式一：作为插件使用（推荐）
```bash
# 在目标项目根目录下
npm install skeptics
# 或放置在 .claude/plugins/ 目录中
```

SessionStart 自动注入 `skeptics:using-skeptics`，Leader 模式自动激活。

### 方式二：独立 Skill 加载
在任意 Claude Code 会话中，通过 `/skill` 加载单个角色 skill：
```
/skill skeptics:workflow     # 启动四阶段工作流
/skill skeptics:approval     # 让批复角色审视产出
/skill skeptics:reviewer     # 审查代码质量
/skill skeptics:tester       # 执行测试
/skill skeptics:doc-keeper   # 更新文档
/skill skeptics:memory       # 管理知识图谱
/skill skeptics:tdd          # TDD 方法论
/skill skeptics:init         # 初始化项目
```

### 方式三：向后兼容（原 `.md` 文件）
原有的 `skills/*.md` 文件仍保留，可通过直接路径加载：
```
/skill skills/approval.md
/skill skills/reviewer.md
/skill skills/testing.md
/skill skills/doc-keeper.md
/skill skills/memory.md
```

## 目录结构

```
├── agent.md              ← 主 Agent 入口（加载此文件启动完整工作流）
├── CLAUDE.md             ← Claude Code 项目指南
├── README.md             ← 本文件
├── CHANGELOG.md          ← 版本变更记录
├── my.md                 ← 原始设计（只读）
├── make.md               ← 核心规范文档（持续维护）
├── SOUL.md               ← 主 Agent 人格定义
├── USER.md               ← 用户画像
├── .claude-plugin/       ← 插件元数据
├── skills/               ← 角色技能定义（SKILL.md 格式）
│   ├── using-skeptics/   ─┐
│   ├── workflow/          │
│   ├── approval/          │
│   ├── reviewer/          ├── 每个子目录一个 SKILL.md
│   ├── tester/            │
│   ├── doc-keeper/        │
│   ├── memory/            │
│   ├── tdd/              ─┘
│   └── init/             ← 项目初始化
├── prompts/              ← Subagent 调度模板
├── hooks/                ← 生命周期钩子
├── plugins/              ← MCP 工具与扩展配置
├── memory/               ← 持久知识与活动日志
└── project/.project/     ← 项目文档模板

## 与 Claude Code 配合

| 场景 | 操作 |
|------|------|
| 启动完整工作流 | 加载 `agent.md` 或说"启动 agent 工作流" |
| 加载批复角色 | `/skill skeptics:approval` |
| 审查代码质量 | `/skill skeptics:reviewer` |
| 执行测试 | `/skill skeptics:tester` |
| 更新文档 | `/skill skeptics:doc-keeper` |
| 记忆管理 | `/skill skeptics:memory` |
| 项目初始化 | `/skill skeptics:init` |
| 查阅完整规范 | 阅读 `make.md`（权威来源） |

## License

MIT
