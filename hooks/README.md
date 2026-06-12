---
name: hooks
description: 工作流生命周期钩子系统——在工作流特定节点自动触发，当前为结构预留
type: flexible
---

# Hook 系统

> 在工作流生命周期的特定节点触发预定义操作。当前为结构预留，随需启用。

## 执行方式

钩子由主 Agent 在工作流推进到对应节点时读取并执行。不是自动脚本，而是 Markdown 规范定义——主 Agent 根据定义执行对应操作。

## 当前预留钩子

| 钩子文件 | 触发节点 | 用途 |
|---------|---------|------|
| `before-requirement-review.hook.md` | 第一阶段需求批复前 | 预检查需求理解完整性 |
| `before-implementation.hook.md` | 第三阶段实施前 | 检查环境就绪状态 |
| `after-approval.hook.md` | 任意阶段批复通过后 | 记录批复结论 |
| `after-report.hook.md` | 第四阶段报告生成后 | 归档总结报告 |

## 钩子接口规范

每个钩子文件必须定义以下元数据：

```yaml
---
name: 钩子名称
trigger: 触发节点（何阶段何事件）
executor: 执行角色
input: 输入内容
output: 产出内容
---
```

然后以 Markdown 描述钩子的具体执行步骤和验收标准。

## 扩展钩子

新增钩子只需在 `hooks/` 目录下创建 `.hook.md` 文件，并按照上述 frontmatter + Markdown 的格式定义即可。主 Agent 在工作流到达对应节点时自动识别并加载。
