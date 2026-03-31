# 第 14 天学习文档：总复盘、架构图与二开入口索引

## 今日目标

今天不再继续扩展阅读范围，只做收官复盘。

目标很明确：

把前 13 天读过的内容压缩成你自己的系统认知，最终回答这 4 个问题：

1. Claude Code 的主链路到底是什么
2. 这个项目最核心的 5 个抽象分别是什么
3. 如果我要二开，我应该从哪里下手
4. 哪些模块我已经看懂，哪些还只是“见过”

---

## 学习成果要求

今天结束前，你至少要产出这 4 份东西：

1. 一张总架构图
2. 一份术语表
3. 一份模块索引
4. 一份自评总结

今天不是“继续读更多文件”，而是把前面读过的内容沉淀成自己的理解结构。

---

## 核心结论

如果你前 13 天都学对了，今天应该能把整个项目压缩成这条主链：

`entrypoints/cli.tsx -> main.tsx -> commands.ts / tools.ts -> AppState + REPL -> handlePromptSubmit -> processUserInput -> query.ts -> toolExecution / orchestration -> sessionStorage / sessionRestore`

你也应该能把整个项目压缩成这 6 层：

1. 启动装配层  
   `entrypoints/cli.tsx`、`main.tsx`

2. 能力注册层  
   `commands.ts`、`tools.ts`

3. 运行时状态层  
   `state/*`、`context/*`

4. 交互与 turn 入口层  
   `REPL.tsx`、`handlePromptSubmit.ts`、`processUserInput/*`

5. query 与工具执行层  
   `query.ts`、`messages.ts`、`Tool.ts`、`services/tools/*`

6. 会话与扩展层  
   `sessionStorage.ts`、`sessionRestore.ts`、`skills/`、`plugins/`、`services/mcp/`、`utils/hooks.ts`

今天要做的就是把这 6 层真正固定下来。

---

## 今日复盘材料

今天不新增学习材料，只回看前面最关键的总骨架文件：

1. [restored-src/src/entrypoints/cli.tsx](../restored-src/src/entrypoints/cli.tsx)
2. [restored-src/src/main.tsx](../restored-src/src/main.tsx)
3. [restored-src/src/commands.ts](../restored-src/src/commands.ts)
4. [restored-src/src/tools.ts](../restored-src/src/tools.ts)
5. [restored-src/src/state/AppStateStore.ts](../restored-src/src/state/AppStateStore.ts)
6. [restored-src/src/screens/REPL.tsx](../restored-src/src/screens/REPL.tsx)
7. [restored-src/src/query.ts](../restored-src/src/query.ts)
8. [restored-src/src/Tool.ts](../restored-src/src/Tool.ts)
9. [restored-src/src/utils/sessionStorage.ts](../restored-src/src/utils/sessionStorage.ts)
10. [restored-src/src/utils/sessionRestore.ts](../restored-src/src/utils/sessionRestore.ts)

补充回看：

- [restored-src/src/services/mcp/](../restored-src/src/services/mcp)
- [restored-src/src/skills/](../restored-src/src/skills)
- [restored-src/src/plugins/](../restored-src/src/plugins)
- [restored-src/src/utils/hooks.ts](../restored-src/src/utils/hooks.ts)

---

## 学习步骤

### 第一步：画总架构图

今天最先做的事不是写文字，而是画图。

你至少要把下面这些节点画出来：

- CLI entry
- main
- commands registry
- tools registry
- AppState
- REPL
- handlePromptSubmit
- processUserInput
- query
- tool execution
- session storage
- session restore
- MCP / plugins / skills / hooks

你不用画细节，只画“谁调用谁、谁依赖谁、谁提供扩展点”。

今天的原则是：

> 架构图只画主链，不画所有边角能力。

---

### 第二步：整理术语表

然后做一份术语表。

今天你至少要整理这些术语：

- command
- tool
- skill
- plugin
- MCP
- AppState
- REPL
- query
- tool_use
- tool_result
- queued command
- hook
- stop hook
- session transcript
- contextModifier

要求不是抄定义，而是用你自己的话写出：

- 这个词在本项目里具体指什么
- 它和相邻术语有什么边界

例如今天你应该已经能说清：

- command 和 tool 的边界
- skill 和 plugin command 的边界
- transcript message 和 progress message 的边界

---

### 第三步：做二开入口索引

这是今天最有价值的产物。

请你按“我要改什么能力”来建索引，而不是按文件夹建索引。

建议按下面这种方式整理：

1. 我要加一个新 slash command  
   去看 `commands.ts` 和对应 command 目录

2. 我要加一个新工具  
   去看 `Tool.ts`、`tools.ts`、现有 `tools/*`

3. 我要改权限  
   去看 `utils/permissions/*`、`Tool.ts`、具体工具的 `checkPermissions`

4. 我要改 REPL 交互  
   去看 `REPL.tsx`、`handlePromptSubmit.ts`、`processUserInput/*`

5. 我要改会话恢复  
   去看 `sessionStorage.ts`、`sessionRestore.ts`

6. 我要改插件 / skill / MCP 接入  
   去看 `plugins/`、`skills/`、`services/mcp/`

今天要做的不是“记住所有文件”，而是建立：

> 需求 -> 模块入口 的映射。

---

### 第四步：给自己做理解分级

今天请把前 13 天内容按 3 个级别标出来：

1. 真正看懂  
   能讲清主线、职责和边界

2. 基本看懂  
   知道作用和位置，但细节还不稳

3. 只见过  
   读过文件，但还没形成稳定理解

建议你直接按模块打标：

- CLI 启动
- main 装配
- commands / tools registry
- AppState / REPL
- 输入与队列
- query 主循环
- 工具执行
- MCP
- skills / plugins
- hooks
- session restore

这一步很重要，因为它能防止你误以为“我都看过了，所以都懂了”。

---

### 第五步：写一页“我现在怎么理解 Claude Code”

今天最后写一页总结，不超过 500 字。

建议结构就是 4 段：

1. 这个项目本质上是什么
2. 主链路是什么
3. 最核心的 5 个抽象是什么
4. 如果我要二开，我会先改哪里

如果你能把这 4 段写顺，你就真的把前面 14 天串起来了。

---

## 今日建议时间安排

建议控制在 2 到 2.5 小时：

1. 总架构图：30 分钟
2. 术语表：30 分钟
3. 二开入口索引：35 分钟
4. 理解分级：15 分钟
5. 最终总结：20 分钟

---

## 今日笔记模板

你可以直接按这个模板记：

```md
# Day 14 - 总复盘

## 一句话总结

## 总架构 6 层
1.
2.
3.
4.
5.
6.

## 主链路

## 我定义的核心术语
- command:
- tool:
- skill:
- plugin:
- MCP:
- AppState:
- REPL:
- query:
- hook:
- transcript:

## 二开入口索引
- 加命令:
- 加工具:
- 改权限:
- 改 REPL:
- 改会话恢复:
- 改 MCP/插件/技能:

## 我的理解分级
- 真正看懂:
- 基本看懂:
- 只见过:

## 我现在怎么理解 Claude Code
```

---

## 今日作业

### 作业 1：画最终架构图

画一张最终架构图，必须至少包含这 6 层：

1. 启动装配层
2. 能力注册层
3. 状态层
4. 交互入口层
5. query / 工具执行层
6. 会话与扩展层

要求每层至少写 2 个核心文件。

### 作业 2：写术语表

写 12 到 15 个术语，每个术语用 1 到 3 句话解释。

要求至少覆盖：

- command
- tool
- skill
- plugin
- MCP
- AppState
- query
- hook
- transcript

### 作业 3：做二开索引

按“我要做什么改动”列一个索引表。

至少覆盖这 6 类改动：

- 新命令
- 新工具
- 权限策略
- REPL 交互
- 会话恢复
- 外部集成

每类改动后面都写上“第一入口文件”和“第二入口文件”。

### 作业 4：写 300 到 500 字总结

题目是：

“我现在如何理解 Claude Code 的架构和运行主线”

要求至少提到：

- 启动链
- 状态链
- 输入链
- query 链
- 工具链
- 会话链

### 作业 5：做个人理解分级

把前 13 天内容分成：

- 真正看懂
- 基本看懂
- 只见过

每类至少列 3 个模块，并写一句原因。

---

## 最终验收清单

这 14 天学完后，确认你能回答：

- Claude Code 从启动到进入 REPL 的主链路是什么
- command、tool、skill、plugin、MCP 的边界分别是什么
- 用户输入是怎么进入 query 主循环的
- 工具执行为什么需要统一契约、单次执行层和编排层
- 会话为什么能恢复，而且恢复的不只是聊天记录
- 如果你现在要二开，你会先去哪些文件

如果这 6 个问题你都能稳定回答，这 14 天学习计划就真正完成了。

---

## 结束建议

14 天结束后，你有 3 个合理下一步：

1. 开始做一项小型二开  
   例如加一个简单 command 或只读工具

2. 按专题二刷  
   例如只刷 `query + tools` 或只刷 `MCP + plugins`

3. 反向做讲解输出  
   写一篇你自己的项目导读，或者做一份内部分享 PPT

如果你能把“输出”做出来，这套学习才算真正沉淀。
