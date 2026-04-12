# 第 10 天学习文档：消息队列、输入处理与会话恢复

## 今日目标

今天要回答的问题是：

用户在 REPL 里输入一句话之后，它是怎么被排队、分流、持久化，并在下次继续会话时恢复出来的。

读完今天内容后，你应该能清楚回答：

1. 为什么项目里要有独立的命令队列
2. prompt、slash command、bash 三类输入是怎么分流的
3. 会话为什么能写成可恢复的 JSONL 记录
4. `/resume` 或 `--continue` 时，旧会话状态是怎么重新接回来的

---

## 学习成果要求

今天结束前，你至少要形成这 4 个判断：

1. `messageQueueManager.ts` 解决的是“输入调度”，不是消息渲染
2. `processUserInput/` 解决的是“输入分发”，不是最终 query 执行
3. `sessionStorage.ts` 是会话持久化中枢
4. `sessionRestore.ts` 是恢复逻辑装配层，把磁盘记录重新映射回运行时状态

---

## 核心结论

先给今天最重要的主线：

- REPL 不会直接把所有输入立即执行，而是通过统一队列调度
- 输入进入系统后，会按模式分流为文本 prompt、slash command 或 bash 命令
- 会话运行过程会持续写入 transcript / metadata / queue operation 等日志
- 恢复时不是简单“读一份聊天记录”，而是连 file history、attribution、todo、worktree、session metadata 一起恢复

今天要建立的主链是：

`messageQueueManager.ts -> processUserInput.ts -> sessionStorage.ts -> sessionRestore.ts -> REPL resume path`

---

## 今日学习材料

按这个顺序读：

1. [restored-src/src/utils/messageQueueManager.ts](../restored-src/src/utils/messageQueueManager.ts)
2. [restored-src/src/utils/processUserInput/processUserInput.ts](../restored-src/src/utils/processUserInput/processUserInput.ts)
3. [restored-src/src/utils/processUserInput/processTextPrompt.ts](../restored-src/src/utils/processUserInput/processTextPrompt.ts)
4. [restored-src/src/utils/processUserInput/processSlashCommand.tsx](../restored-src/src/utils/processUserInput/processSlashCommand.tsx)
5. [restored-src/src/utils/processUserInput/processBashCommand.tsx](../restored-src/src/utils/processUserInput/processBashCommand.tsx)
6. [restored-src/src/utils/sessionStorage.ts](../restored-src/src/utils/sessionStorage.ts)
7. [restored-src/src/utils/sessionRestore.ts](../restored-src/src/utils/sessionRestore.ts)
8. [restored-src/src/screens/REPL.tsx](../restored-src/src/screens/REPL.tsx)

补充定位：

- [restored-src/src/utils/handlePromptSubmit.ts](../restored-src/src/utils/handlePromptSubmit.ts)
- [restored-src/src/types/messageQueueTypes.ts](../restored-src/src/types/messageQueueTypes.ts)
- [restored-src/src/types/logs.ts](../restored-src/src/types/logs.ts)

---

## 学习步骤

### 第一步：理解为什么需要统一命令队列

先读 [messageQueueManager.ts](../restored-src/src/utils/messageQueueManager.ts)。

你要先看明白这份文件在做什么：

- 维护模块级 `commandQueue`
- 提供 `enqueue`、`dequeue`、`peek`、`dequeueAllMatching`
- 提供 `useSyncExternalStore` 兼容接口
- 记录 queue operation 到 session storage

这里最重要的设计点是：

- 队列不属于某个单独组件
- 队列有优先级，至少区分 `now`、`next`、`later`
- 用户输入和系统通知共用一个调度管道

你应该得出结论：

> 这个队列是 REPL 的前台调度器，用来避免用户输入、后台通知、任务结果彼此抢执行时机。

---

### 第二步：把输入处理器看成“路由层”

接着读 [processUserInput.ts](../restored-src/src/utils/processUserInput/processUserInput.ts)。

这份文件很关键，但今天不要把它当 query 内核来读。你只需要抓住它的职责：

- 规范化输入
- 处理 pasted content / 图片块
- 判断是否是 slash command
- 根据 mode 走不同处理器
- 在真正 query 之前执行 hooks

这里有两个特别值得注意的点：

1. 它返回的是 `ProcessUserInputBaseResult`
   说明这个阶段还只是“整理输入结果”，不一定立即发起 query

2. 它区分 `shouldQuery`
   说明有些输入只会产生日志消息或工具结果，不会继续进入模型对话

今天你要记住一句话：

> `processUserInput.ts` 是输入预处理和分流总线，不是最终执行器。

---

### 第三步：看最普通文本 prompt 是怎么转成消息的

然后读 [processTextPrompt.ts](../restored-src/src/utils/processUserInput/processTextPrompt.ts)。

它体现了最纯的一条路径：

- 生成 `promptId`
- 记录 telemetry
- 把文本和图片块整理成 `UserMessage`
- 返回 `shouldQuery: true`

这里的重点不是复杂逻辑，而是理解它在整个链路中的角色：

- 它不碰队列
- 不直接 query
- 不负责 UI
- 只负责把“用户输入”变成“标准消息对象”

你要得出结论：

> prompt 路径的第一步不是让模型回答，而是先规范化成统一消息结构。

---

### 第四步：看 slash command 为什么最复杂

再读 [processSlashCommand.tsx](../restored-src/src/utils/processUserInput/processSlashCommand.tsx)。

这份文件复杂是正常的，因为 slash command 本身就有多种形态：

- 直接本地执行
- 生成结果后重新入队
- fork 到子 agent
- 通过插件或 skill 扩展

今天重点只抓这几件事：

1. command lookup 是怎么做的
2. 哪些 command 会 fork 子 agent
3. 为什么有些结果会重新 `enqueuePendingNotification`
4. 为什么 slash command 有时 `shouldQuery: false`

这里最能体现架构思路的一点是：

> slash command 不一定直接形成一轮普通对话，它可能只是触发本地编排动作，再把结果转成下一轮隐藏 prompt。

这也是为什么命令系统不能简单看成“给模型一个特殊 prompt”。

---

### 第五步：看 bash 输入为什么也走统一输入系统

继续读 [processBashCommand.tsx](../restored-src/src/utils/processUserInput/processBashCommand.tsx)。

要重点看清：

- `!` 风格输入会路由到 shell 工具
- shell 结果会被包装成结构化消息
- 这条路径通常 `shouldQuery: false`
- UI 上仍然有专门的进度显示

今天你要理解的不是 shell 细节，而是这个设计：

> 用户在输入框里做的 bash 输入，也被收敛进统一的消息与会话体系，而不是单独旁路执行。

这保证了：

- 可以被记录
- 可以进入 transcript
- 可以被恢复
- 可以和其他输入统一排队

---

### 第六步：进入会话持久化核心

接着读 [sessionStorage.ts](../restored-src/src/utils/sessionStorage.ts)。

这份文件很大，今天不要试图全啃完。你只抓 5 个主题：

1. transcript 路径和 session 文件路径怎么计算
2. 什么叫 transcript message，什么不该进入主链
3. 写入是怎么排队和落盘的
4. 元数据除了消息本身还保存了什么
5. 为什么要有 `adoptResumedSessionFile()`、`restoreSessionMetadata()` 这种函数

你应该特别注意这些设计点：

- transcript 不是简单文本，而是结构化 JSONL
- progress 这类临时 UI 消息并不总属于 transcript 主链
- 会话文件可能很大，因此代码里有大量“避免 OOM”的读取策略
- 除了消息外，还会保存 queue operation、title、tag、summary、worktree state 等附加记录

今天最重要的结论是：

> `sessionStorage.ts` 实际上是 Claude Code 的会话事件日志层，而不只是“存聊天记录”。

---

### 第七步：理解恢复不是“读聊天记录”，而是重建运行时

然后读 [sessionRestore.ts](../restored-src/src/utils/sessionRestore.ts)。

重点只看这些函数和主题：

- `restoreSessionStateFromLog`
- `computeRestoredAttributionState`
- `computeStandaloneAgentContext`
- `restoreAgentFromSession`
- worktree 恢复相关逻辑

你要理解恢复时发生了什么：

- file history 被恢复
- attribution 被恢复
- todo 状态可能从 transcript 中提取
- session metadata 会重新接回
- agent 设定和 model override 可能被恢复
- worktree 也可能被恢复到正确上下文

所以今天必须写下这句话：

> session restore 的目标不是还原“文本历史”，而是重建“可继续执行的运行时上下文”。

---

### 第八步：回到 REPL 看真实调用点

最后回到 [REPL.tsx](../restored-src/src/screens/REPL.tsx)。

今天不要通读，只盯住这些关键词：

- `handlePromptSubmit`
- `getCommandQueueLength`
- `enqueue`
- `restoreSessionStateFromLog`
- `restoreSessionMetadata`
- `adoptResumedSessionFile`
- `removeTranscriptMessage`

你要看明白 3 条主链：

1. 用户提交输入后，REPL 怎么决定直接处理还是先入队
2. query 正在运行时，新输入为什么会排队
3. resume 时，REPL 怎么把磁盘状态接回当前会话

看到这里，你应该能把今天的全链路讲出来：

`PromptInput -> handlePromptSubmit -> processUserInput -> queue / messages / shouldQuery -> sessionStorage -> resume path`

---

## 今日建议时间安排

建议控制在 2 到 3 小时：

1. `messageQueueManager.ts`：25 分钟
2. `processUserInput/` 四个文件：50 分钟
3. `sessionStorage.ts`：45 分钟
4. `sessionRestore.ts`：30 分钟
5. `REPL.tsx` 调用点回看与笔记：30 分钟

---

## 今日笔记模板

你可以直接按这个模板记：

```md
# Day 10 - 队列、输入处理、会话恢复

## 一句话总结

## 今日主链
- messageQueueManager.ts:
- processUserInput.ts:
- processTextPrompt.ts:
- processSlashCommand.tsx:
- processBashCommand.tsx:
- sessionStorage.ts:
- sessionRestore.ts:
- REPL.tsx:

## 我理解的输入分流
- prompt:
- slash command:
- bash:

## 我理解的会话持久化
- transcript:
- metadata:
- queue operation:
- resume:

## 今天确认的 3 个设计点
1.
2.
3.

## 还没搞懂的 3 个问题
1.
2.
3.
```

---

## 今日作业

### 作业 1：画输入流转图

画一张最小流程图，至少包含这些节点：

- PromptInput
- handlePromptSubmit
- processUserInput
- processTextPrompt / processSlashCommand / processBashCommand
- queue
- sessionStorage
- query

要求你能讲清：

- 哪一步是“分流”
- 哪一步是“排队”
- 哪一步是“落盘”

### 作业 2：总结队列设计

用 150 到 250 字回答：

为什么项目要把用户输入、任务通知、隐藏 prompt 都统一放进 `messageQueueManager.ts`？

要求至少提到：

- 优先级
- React 外部可访问
- 和会话记录的关系

### 作业 3：分类 3 条输入路径

从 `processUserInput/` 中分别总结：

- 文本 prompt 路径
- slash command 路径
- bash 路径

每条路径都要回答：

- 输入长什么样
- 产出什么消息
- `shouldQuery` 通常是什么
- 是否可能重新入队

### 作业 4：列 6 个 sessionStorage 的职责

从 [sessionStorage.ts](../restored-src/src/utils/sessionStorage.ts) 中列出 6 个职责，不能只写“保存消息”。

例如你应该覆盖其中一部分：

- transcript 路径管理
- 读取策略
- 写入排队
- metadata 恢复
- 子 agent transcript
- worktree/session 附加信息

### 作业 5：解释 resume 的真正含义

用 200 到 300 字回答：

为什么 `sessionRestore.ts` 的恢复不是简单地把历史消息读回来显示，而是运行时重建？

要求必须提到至少 4 项：

- file history
- attribution 或 todo
- agent/model
- worktree 或 session metadata

---

## 自检清单

今天结束前，确认你能回答：

- `messageQueueManager.ts` 为什么是模块级队列，而不是组件局部 state
- `processUserInput.ts` 和真正的 `query()` 有什么边界
- 为什么 `slash command` 不能简单等同于“特殊 prompt”
- `sessionStorage.ts` 为什么更像事件日志层
- `sessionRestore.ts` 为什么必须恢复运行时而不只是恢复 transcript

如果这 5 个问题里你有 4 个能顺畅讲出来，第 10 天就算完成。

---

## 明日预告

第 11 天建议进入 query 与消息执行主线，重点看：

- `query.ts`
- `utils/messages.ts`
- `utils/handlePromptSubmit.ts`
- `hooks/useQueueProcessor.ts`

到那一天，你会把今天的“输入进入系统”继续推进到：

“消息如何真正驱动一轮模型调用、工具流式输出和 turn 生命周期管理。”
