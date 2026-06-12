---
name: plugins
description: 插件系统——MCP 工具配置和第三方集成扩展，当前为结构预留
type: flexible
---

# 插件系统

> 用于扩展工作流体系的能力边界。当前为结构预留，随需启用。

## 插件类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **MCP 工具** | 通过 MCP 协议接入外部能力 | 文件系统、数据库、Git、浏览器 |
| **技能注册** | 将 skill 文件映射到角色和场景 | approval/reviewer/testing 等 |
| **第三方集成** | 对接外部服务和 API | Slack、Jira、GitHub |

## 使用方式

插件定义在 `plugins/` 目录下的 Markdown 文件中。主 Agent 在需要对应能力时，读取插件定义并执行。

## 当前文件

| 文件 | 内容 |
|------|------|
| `mcp-tools.md` | MCP 工具列表与阶段映射 |
| `skills-registry.md` | 技能注册索引 |

## 扩展插件

新增插件只需在 `plugins/` 目录下创建 `.md` 文件，按照 Markdown + frontmatter 格式定义即可。建议的文件命名：`[能力名称].md`。
