---
name: after-report
trigger: 第四阶段—总结报告生成后
executor: 主 Agent
input: 总结报告内容
output: 归档记录
---

# Hook: 报告后归档

## 触发条件
第四阶段总结报告生成之后。

## 执行步骤

1. 将总结报告转化为 JSON 记录写入 `memory/JOURNAL.jsonl`
2. 识别报告中的"问题记录"和"验收问题"，同步到 `project/.project/learned.md`
3. 检查是否有高频问题（同一问题出现 ≥3 次）需要提升到 `memory/FACT.md`
4. 确认所有文档已更新完毕
5. 通知用户实施全部完成

## 输出格式

```json
{"ts":"ISO时间戳","tags":["phase-complete","report"],"text":"实施总结：N个任务，通过M个，失败K个，持续时间X"}
```
