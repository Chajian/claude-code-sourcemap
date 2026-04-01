# 学习总目录

这套文档面向开源读者，目标不是“逐文件抄读”，而是帮助你用较低成本建立 Claude Code 的整体认知。

推荐方式：

1. 按天顺序阅读
2. 每天配合对应源码目录一起看
3. 每天至少完成笔记模板和作业中的核心部分

## 14 天学习计划

1. [第 1 天：仓库定位与 source map 还原机制](day-01-learning-plan.md)
2. [第 2 天：CLI 启动分流](day-02-learning-plan.md)
3. [第 3 天：main.tsx 主装配层](day-03-learning-plan.md)
4. [第 4 天：commands.ts 与 tools.ts 注册表](day-04-learning-plan.md)
5. [第 5 天：权限模型与工具边界](day-05-learning-plan.md)
6. [第 6 天：权限系统在具体工具中的落地](day-06-learning-plan.md)
7. [第 7 天：MCP 集成](day-07-learning-plan.md)
8. [第 8 天：skills 与 plugins 体系](day-08-learning-plan.md)
9. [第 9 天：状态管理与 REPL 交互](day-09-learning-plan.md)
10. [第 10 天：消息队列、输入处理与会话恢复](day-10-learning-plan.md)
11. [第 11 天：Query 主线、消息协议与队列消费](day-11-learning-plan.md)
12. [第 12 天：工具执行主线与 Tool Orchestration](day-12-learning-plan.md)
13. [第 13 天：Hooks、Permission 与 Stop/Recovery](day-13-learning-plan.md)
14. [第 14 天：总复盘、术语表与二开入口索引](day-14-learning-plan.md)

## 配套材料

- [项目架构图](claude-code-architecture.drawio)
- Buddy 专栏与图鉴
- [Buddy 指令专栏](buddy-command-column.html)
- [Buddy 外观图鉴](buddy-appearance-gallery.html)

## 建议先看的源码目录

- `../restored-src/src/`
- `../restored-src/src/entrypoints/`
- `../restored-src/src/state/`
- `../restored-src/src/services/`
- `../restored-src/src/tools/`
- `../restored-src/src/commands/`

## 阅读原则

- 先入口，后实现
- 先主链，后分支
- 先注册表，后单点能力
- 先写自己的理解，再回看源码细节
