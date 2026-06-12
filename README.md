# skeptics — 多 Agent 协作工作流体系

> 一套结构化多 AI Agent 协作方案——**7 角色 + 4 阶段工作流 + 三重 Loop 验证 + 六论知识图谱**，专为 vibecoding 场景下的 AI 辅助编程设计。
>
> 已封装为 Claude Code 插件，支持 SessionStart 自动注入和 Subagent 隔离调度。

---

## 项目介绍

skeptics 是一套**以批复对抗为驱动**的多 Agent 协作体系。核心理念是：AI 辅助编程中，**不相信任何单一角色的产出，所有结论必须经过至少三层验证才能放行**。

- **7 个专业化角色**各司其职，每个角色有严格的职责边界
- **4 个阶段串行推进**，每阶段末由批复角色做门控拦截
- **三重 Loop 验证**确保实施质量：质量审查、测试验证、批复门控
- **六论知识图谱**持久化项目知识，跨会话积累经验

---

## 核心流程

```
用户需求 → [需求讨论] ──批复通过？──→ [任务规划] ──批复通过？──→ [实施调度] ──批复通过？──→ [总结报告]
                │ 否                         │ 否                         │ 否
                └── 修正需求 ←──┘            └── 修正规划 ←──┘            └── 修正实施 ←──┘
```

### 四阶段详解

| 阶段 | 目标 | 输入 | 产出 | 批复依据 |
|------|------|------|------|---------|
| **需求讨论** | 理解用户真实需求，对齐预期 | 用户原始需求 | 需求文档（requirement.md） | 需求完整性/正确性/可行性 |
| **任务规划** | 将需求拆解为可执行的任务 | 需求文档 | 任务文档（task.md）+ 进度文档（progress.md） | 任务粒度/优先级/依赖合理性 |
| **实施调度** | 分配任务给 Worker，逐项实施并验证 | 任务文档 + 进度文档 | 实施产出 + 测试报告 + 审查报告 | 实施结果/测试结论/审查结论 |
| **总结报告** | 汇总实施结果，归档经验 | 所有阶段产出 | 总结报告 + 经验文档（learned.md） | 报告完整性/经验提炼质量 |

---

## 7 大角色

| 角色 | 身份 | 核心职责 | 严格边界 |
|------|------|---------|---------|
| **Leader** | 主 Agent — 调度者 | 需求理解、任务拆解、角色调度、流程管控、用户汇报 | 不写代码、不写文档、不写测试 |
| **Approver** | 批复角色 — 门控者 | 走流程做评判：提取→对照→产出→通过/不通过。**对抗性审视**，对每个结论进行 PUA 式追问 | 只有通过/不通过，不做主观判断 |
| **Worker** | 实施角色 — 执行者 | 根据任务描述实施具体工作（代码/配置/调试），内部执行 TDD（RED→GREEN→REFACTOR） | 只负责实施，不具备调度能力 |
| **Reviewer** | 内容审查角色 — 质检者 | 7 维度审查实施产出：代码质量、设计合理性、完整性、安全性、性能、可维护性、一致性 | 产出可操作的修改意见，不负责测试 |
| **Tester** | 测试角色 — 验证者 | 5 类型测试覆盖：单元测试、集成测试、E2E 测试、边界测试、回归测试 | 产出测试报告，不修复代码 |
| **DocKeeper** | 文档维护角色 — 记录者 | 按模板统一维护需求/进度/任务/实施经验四类文档 | **唯一**负责写入文档，其他角色不直接操作文档 |
| **Owner** | 用户 — 决策者 | 需求发起、关键决策、最终验收 | 最终决定权，不参与具体实施 |

---

## 三重 Loop 验证

Worker 实施完成后，触发三层递进式验证：

```
                    ┌──────────────────────────────────┐
                    │         Worker 实施完成           │
                    └────────────────┬─────────────────┘
                                     ▼
    ┌────────────────────────────────────────────────────────────────┐
    │  Loop 1: 质量审查                                                │
    │  Reviewer → Worker（审查不通过则返回修改）                          │
    │  7 维度检查：空代码/死代码/过度设计/安全漏洞/N+1查询/命名可读性等      │
    └────────────────────────────────┬────────────────────────────────┘
                                     ▼
    ┌────────────────────────────────────────────────────────────────┐
    │  Loop 2: 测试验证                                                │
    │  Tester → Worker（测试不通过则返回修复）                            │
    │  5 类型覆盖：正常路径/异常路径/边界值/集成/回归                      │
    │  6 原则：新鲜运行/git HEAD 验证/时间戳/≥2 断言/无依赖/合理 mock      │
    └────────────────────────────────┬────────────────────────────────┘
                                     ▼
    ┌────────────────────────────────────────────────────────────────┐
    │  Loop 3: 批复门控                                                │
    │  Approver 审视审查结论和测试结论                                   │
    │  只有通过/不通过——不通过则整批返回重修                              │
    └────────────────────────────────────────────────────────────────┘
```

---

## 六论知识图谱

项目知识以六论知识图谱持久化，跨会话积累，越用越聪明：

| 论域 | 节点类型 | 解决的问题 | 生命周期 |
|------|---------|-----------|---------|
| **需求论** | 需求 / 功能点 / 约束 | "项目要做什么？" | proposed → confirmed → implemented → deprecated |
| **经验论** | 模式 / 教训 / 最佳实践 | "哪些做法有效？" | observed → validated → established → archived |
| **错误论** | 错误 / 根因 / 修复方案 | "哪里出过问题？" | detected → diagnosed → fixed → monitored |
| **技术栈论** | 技术 / 依赖 / 配置 | "用了什么技术？" | introduced → active → deprecated → removed |
| **架构论** | 模块 / 接口 / 数据流 | "系统如何组织？" | designed → implemented → refactored → removed |
| **工具论** | MCP 工具 / Skill / CLI 工具 | "有什么工具可用？" | discovered → configured → adopted → archived |

---

## 插件结构

```
skeptics/
├── .claude-plugin/           ← 插件元数据
│   ├── plugin.json           ← 插件注册信息
│   └── marketplace.json      ← 市场发布配置
├── skills/                   ← 角色技能定义（SKILL.md）
│   ├── using-skeptics/       ← 入口启动（SessionStart 自动注入 Leader 模式）
│   ├── workflow/             ← 四阶段工作流完整定义
│   ├── approval/             ← 批复角色（PUA 对抗追问 + L0-L4 压力升级）
│   ├── reviewer/             ← 内容审查角色（7 维度审查清单）
│   ├── tester/               ← 测试角色（5 类型测试 + 6 条执行原则）
│   ├── doc-keeper/           ← 文档维护角色（4 套文档模板 + 5 条写作规范）
│   ├── memory/               ← 六论知识图谱管理（18 节点类型 + 15 关系类型）
│   ├── tdd/                  ← 测试驱动开发方法论（RED→GREEN→REFACTOR）
│   └── init/                 ← 项目初始化（CLAUDE.md 追加 + 目录脚手架）
├── prompts/                  ← Subagent 隔离调度模板
│   ├── implementer.md        ← Worker Subagent 指令（含 TDD + 文件锁协议）
│   └── tester-agent.md       ← Tester Subagent 指令（含新鲜验证标准）
├── hooks/                    ← 生命周期钩子
│   ├── hooks.json            ← SessionStart 钩子注册
│   └── session-start         ← 自动注入脚本（bash）
├── plugins/                  ← MCP 工具映射与技能注册索引
├── package.json              ← npm 包元数据
├── agent.md                  ← 工作流主入口（加载即启动）
├── make.md                   ← 核心规范文档（权威来源）
├── SOUL.md                   ← Leader 人格定义
├── USER.md                   ← 用户画像
└── CLAUDE.md                 ← 项目指南
```

---

## 快速开始

### 方式一：插件安装（推荐）

```bash
# 将 skeptics 放置到 Claude Code 插件目录
git clone https://github.com/znewyear/skeptic.git /path/to/.claude/plugins/skeptics
```

或通过 npm：

```bash
npm install skeptics
```

安装后启动 Claude Code，SessionStart 钩子自动注入 `skeptics:using-skeptics`，Leader 模式自动激活。

### 方式二：独立加载角色

在 Claude Code 会话中手动加载单个角色：

```
/skill skeptics:workflow     启动四阶段工作流
/skill skeptics:approval     让批复角色审视产出
/skill skeptics:reviewer     审查代码质量
/skill skeptics:tester       执行测试
/skill skeptics:doc-keeper   更新文档
/skill skeptics:memory       管理知识图谱
/skill skeptics:tdd          TDD 方法论指导
/skill skeptics:init         初始化项目环境
```

### 方式三：通过 agent.md 启动

```
加载 agent.md，或直接说"启动 agent 工作流"
```

### 方式四：向后兼容

原有的 `skills/*.md` 文件仍保留，可通过直接路径加载：

```
/skill skills/approval.md
/skill skills/reviewer.md
/skill skills/testing.md
/skill skills/doc-keeper.md
/skill skills/memory.md
```

---

## 设计原则

| 原则 | 说明 |
|------|------|
| **批复对抗** | 每个结论必须经过对抗性质疑才能放行，杜绝"看起来没问题" |
| **流程驱动** | 所有检查项都是固定流程——先做 A、再做 B、产出 C，结论由流程产出决定 |
| **困难升级** | 同一内容连续批复不通过时压力自动升级（L0→L4），第 5 次上报 Leader |
| **文件锁** | 并发 Subagent 修改文件前必须注册文件锁，防止冲突 |
| **新鲜验证** | 所有测试和审查必须在当前代码状态下新鲜运行，禁止引用缓存结果 |
| **根因优先** | 遇到问题必须追查根因，不能只修复表面症状 |
| **ASCII 可视化** | 所有流程、架构、协作关系使用 ASCII 图表表达 |
| **TDD 内建** | Worker 实施必须执行 RED→GREEN→REFACTOR 循环，先测试后代码 |

---

## 与 Claude Code 配合速查

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

---

## License

MIT
