---
name: skills-registry
description: 技能注册索引——将 skill 文件映射到角色、触发场景和使用方式
type: reference
---

# 技能注册索引

> 将技能文件映射到角色和触发场景。主 Agent 据此决定何时加载哪个 skill。

## 插件名空间方式（推荐）

skeptics 插件安装后，通过 `/skill skeptics:*` 调用：

| 插件技能 | 对应角色 | 触发场景 |
|---------|---------|---------|
| `skeptics:workflow` | Leader | 启动四阶段工作流 |
| `skeptics:approval` | 批复角色 | 需求讨论、任务规划、实施验收 |
| `skeptics:reviewer` | 内容审查角色 | 实施完成后质量审查 |
| `skeptics:tester` | 测试角色 | 实施完成后测试验证 |
| `skeptics:doc-keeper` | 文档维护角色 | 需求确认、任务确认、实施完成、总结 |
| `skeptics:memory` | Leader | 会话初始化、知识图谱管理 |
| `skeptics:tdd` | Worker | 实施阶段 TDD 方法论指导 |
| `skeptics:init` | Leader | 首次项目初始化 |

## 直接路径方式（向后兼容）

| 技能文件 | 对应角色 | 触发场景 |
|---------|---------|---------|
| `skills/approval/SKILL.md` | 批复角色 | 需求讨论、任务规划、实施验收 |
| `skills/reviewer/SKILL.md` | 内容审查角色 | 实施完成后审查 |
| `skills/tester/SKILL.md` | 测试角色 | 实施完成后测试 |
| `skills/doc-keeper/SKILL.md` | 文档维护角色 | 各阶段文档更新 |
| `skills/memory/SKILL.md` | Leader | 记忆管理 |
| `skills/tdd/SKILL.md` | Worker | TDD 实施方法论 |
| `skills/workflow/SKILL.md` | Leader | 工作流定义 |
| `skills/init/SKILL.md` | Leader | 项目初始化 |
| `skills/using-skeptics/SKILL.md` | —（自动注入） | SessionStart |

## 注册规范

新增技能时，在对应 `skills/<name>/` 目录下创建 `SKILL.md`，frontmatter 必须包含：

```yaml
---
name: 技能名称
description: 一句话描述
---
```

同时在此索引中添加对应条目。
