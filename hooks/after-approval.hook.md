---
name: after-approval
trigger: 任意阶段—批复通过后
executor: 主 Agent
input: 批复结论
output: 批复记录
---

# Hook: 批复后记录

## 触发条件
批复角色产出"通过"结论之后。

## 执行步骤

1. 记录批复结论到 `memory/JOURNAL.jsonl`
2. 若批复涉及需求变更或任务状态变更，联动更新对应文档
3. 通知用户当前阶段已通过批复，可进入下一阶段

## 输出格式

记录到 JOURNAL.jsonl：
```json
{"ts":"ISO时间戳","tags":["approval-pass"],"text":"[阶段名]批复通过，检查项N项，全部通过"}
```
