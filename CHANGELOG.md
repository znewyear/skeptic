# CHANGELOG

## v0.2.0 (2026-06-12)

### skeptics 插件化
- 封装为 Claude Code 插件（`.claude-plugin/plugin.json`）
- 9 个 SKILL.md 技能文件，遵循 superpowers 规范
- SessionStart 钩子自动注入 `skeptics:using-skeptics`
- 三重 Loop 验证机制：Reviewer↔Worker（质量）、Tester↔Worker（验证）、Approver（门控）
- 六论知识图谱：需求论/经验论/错误论/技术栈论/架构论/工具论，含完整生命周期管理
- TDD 方法论集成：RED→GREEN→REFACTOR，作为 Worker 内部实施流程
- PUA 对抗追问：批复角色新增冰山追问、证据质证、懒惰验证、假设反转
- 困难升级机制：L0-L4 五级压力等级，连续不通过自动升级
- Worker/Tester Subagent 模板（`prompts/implementer.md` + `prompts/tester-agent.md`）
- 项目初始化 Skill（`skeptics:init`）：自动追加 CLAUDE.md + 创建 .project/ 骨架
- 全局规则：文件锁、新鲜验证（git HEAD + 时间戳）、根因优先、ASCII 可视化
- 旧版 `.md` 向后兼容，保留为重定向文件

## v0.1.0 (2026-06-12)

### 初始版本
- 完整 7 角色体系：主 Agent、批复角色、实施角色、内容审查角色、测试角色、文档维护角色、用户
- 四阶段工作流：需求讨论 → 任务规划 → 实施调度 → 总结报告
- 批复角色流程驱动模式：每个批复项统一为"提取→对照→产出"三步流程
- 5 个技能文件：approval.md / reviewer.md / testing.md / doc-keeper.md / memory.md
- 文档模板集：需求/进度/任务/实施经验 4 类文档
- 记忆体系：FACT.md + JOURNAL.jsonl，含提升/降级流转规则
- 难题体系：5 级难度，突破计划（调研→POC→实施），自动升级机制
- Hooks/Plugins 扩展占位
