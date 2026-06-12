---
name: mcp-tools
description: MCP 工具列表——工作流各阶段可用的 MCP 工具映射
type: reference
---

# MCP 工具

> 工作流各阶段可用的 MCP 工具。MCP 配置在 Claude Code 的 `settings.json` 中，此处只做角色-阶段映射。

## 工具映射

| MCP 工具 | 用途 | 适用阶段 | 调用角色 |
|----------|------|----------|----------|
| 文件系统 MCP | 读写项目文件 | 实施阶段 | 实施角色、文档维护角色 |
| 数据库 MCP | 数据库操作、查询验证 | 实施阶段 | 实施角色、测试角色 |
| Git MCP | 版本控制、分支管理 | 实施阶段 | 实施角色 |
| 浏览器 MCP | 页面预览、UI 验证 | 验收阶段 | 测试角色、内容审查角色 |
| API 调用 MCP | 接口调试、第三方集成 | 实施/测试阶段 | 实施角色、测试角色 |
| 搜索 MCP | 技术文档检索、方案调研 | 所有阶段 | 所有角色 |
| 终端 MCP | 执行构建/部署/测试命令 | 实施/测试阶段 | 实施角色、测试角色 |

## 配置方式

在 Claude Code 的 `settings.json` 中配置：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    }
  }
}
```

具体配置参见各 MCP 服务器的官方文档。
