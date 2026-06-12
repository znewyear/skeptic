---
name: skills-registry
description: 技能注册索引——将 skill 文件映射到角色、触发场景和使用方式
type: reference
---

# 技能注册索引

> 将技能文件映射到角色和触发场景。主 Agent 据此决定何时加载哪个 skill。

## 技能清单

| 技能文件 | 对应角色 | 触发场景 | 加载方式 |
|---------|---------|---------|---------|
| `skills/approval.md` | 批复角色 | 需求讨论、任务规划、实施验收 | 主 Agent 调度时加载 |
| `skills/reviewer.md` | 内容审查角色 | 实施完成后验收 | 主 Agent 调度时加载 |
| `skills/testing.md` | 测试角色 | 实施完成后验收 | 主 Agent 调度时加载 |
| `skills/doc-keeper.md` | 文档维护角色 | 需求确认、任务确认、实施完成、总结阶段 | 主 Agent 调度时加载 |
| `skills/memory.md` | 主 Agent | 会话初始化、流转判定 | 主 Agent 自加载 |

## 注册规范

新增技能文件时，在此索引中添加对应条目。技能文件的 frontmatter 必须包含：

```yaml
---
name: 技能名称
description: 一句话描述
type: rigid | flexible
---
```

- `rigid`：强制执行，不自由裁量（如批复/审查/测试）
- `flexible`：可调节，适配上下文（如记忆管理）
