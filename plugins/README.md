---
name: plugins
description: 插件系统——MCP 工具配置、第三方集成扩展和技能注册索引
type: flexible
---

# 插件系统

> 扩展工作流体系的能力边界。本目录管理 MCP 工具映射、技能注册索引和第三方集成。

## 目录内容

| 文件 | 内容 |
|------|------|
| `mcp-tools.md` | MCP 工具列表与角色-阶段映射 |
| `skills-registry.md` | 技能注册索引（新旧路径对照） |

## 与 skeptics 插件的关系

skeptics 插件（`.claude-plugin/`）提供工作流核心能力。本目录（`plugins/`）管理工作流外围集成：

- **skeptics 插件**：7 角色技能、SessionStart 钩子、Subagent 调度模板
- **本目录**：MCP 工具配置指引、技能注册索引、第三方集成参考

## 扩展方式

新增插件配置只需在本目录下创建 `.md` 文件，按 Markdown + frontmatter 格式定义即可。
