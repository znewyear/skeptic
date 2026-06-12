---
name: memory
description: 记忆管理——管理 FACT.md/JOURNAL.jsonl/六论知识图谱。本文件为旧版，新版本位于 memory/SKILL.md
---

# 记忆管理（旧版）

> 本文件仅为向后兼容保留。新版内容已迁移至 `skills/memory/SKILL.md`。

## 迁移说明

| 项目 | 旧版路径 | 新版路径 |
|------|---------|---------|
| 记忆管理 | `skills/memory.md` | `skills/memory/SKILL.md` |
| 调用方式 | `/skill skills/memory.md` | `/skill skeptics:memory` |

新版优化：
- SessionStart 5 步初始化加载顺序
- 六论知识图谱（需求论/经验论/错误论/技术栈论/架构论/工具论）
- 完整图存储格式（graph.json + schema.json + index.json + archive.json）
- 节点生命周期管理（创建→活跃→冻结→归档→删除）
- 跨论关联自动发现
