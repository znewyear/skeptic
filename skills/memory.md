---
name: memory
description: 记忆管理——管理 FACT.md 持久知识、JOURNAL.jsonl 活动日志、记忆流转规则
type: flexible
---

# 记忆管理

> 管理项目的记忆体系：持久知识读写、活动日志记录、记忆自动流转。

## 记忆体系

| 类型 | 优先级 | 存储位置 | 更新时机 |
|------|--------|----------|----------|
| 需求记忆 | 🔴 高度 | requirement.md + FACT.md | 需求确认后 |
| 进度记忆 | 🔴 高度 | progress.md | 每阶段推进时 |
| 任务记忆 | 🔴 高度 | task.md | 规划确认 + 实施更新 |
| 纠错记忆 | 🟠 中度 | learned.md + FACT.md | 验收后 |
| 报错记忆 | 🟠 中度 | learned.md | 实施+验收过程 |

## 会话初始化

每次会话开始时，必须执行：
1. 读取 `project/.project/requirement.md` → 需求列表和状态
2. 读取 `project/.project/progress.md` → 进度和状态
3. 读取 `memory/FACT.md` → 持久知识
4. 读取 `learned.md`"反复出现的问题" → 避免重复犯错

## 流转规则

### 提升（learned.md → FACT.md）
- 同一问题出现 ≥3 次，或不同编号但本质相同累计 ≥3 次
- 复制到 FACT.md，标注来源编号和首次出现时间

### 降级（FACT.md → learned.md 归档区）
- FACT.md 条目 ≥6 个月未被任何文档引用
- 搜索所有项目文档无匹配后移回

## 日志记录

JOURNAL.jsonl 每行一个 JSON：
```json
{"ts":"ISO时间戳","tags":["标签1","标签2"],"text":"事件描述"}
```

必须记录：需求变更、任务状态变更、验收结果、重要决策、实施路线变更
