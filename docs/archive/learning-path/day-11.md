# 第 11 天学习文档：Query 主线、消息协议与队列消费

## 今日目标

今天要解决的问题是：

一轮 turn 真正开始之后，Claude Code 是怎么把输入推进成流式响应、工具调用、消息追加和 turn 收尾的。

读完今天内容后，你应该能回答：

1. `handlePromptSubmit` 和 `query()` 的职责边界是什么
2. 队列中的命令在什么时候真正被执行
3. `messages.ts` 为什么是协议层而不是普通工具函数文件
4. 一轮流式响应中，assistant message、tool_use、tool_result、stream_event 是怎么协同的

---

## 学习成果要求

今天结束前，你至少要形成这 4 个判断：

1. `handlePromptSubmit.ts` 是 turn 入口协调层
2. `useQueueProcessor.ts` 是“空闲时自动排水”的触发器
3. `query.ts` 是真正的 agentic query 循环
4. `messages.ts` 定义了几乎所有消息对象和消息转换规则

---

## 核心结论

今天的核心链路可以压缩成一句话：

`输入提交 -> handlePromptSubmit -> processUserInput -> onQuery -> query() -> stream/tool -> messages append`

更细一点是：

- `handlePromptSubmit.ts` 决定现在执行、立即命令、还是入队
- `useQueueProcessor.ts` 在 query 空闲时触发队列消费
- `query.ts` 负责模型调用、流式事件、工具编排、恢复分支、自动 compact
- `messages.ts` 负责把这些不同阶段的内容整理成统一消息协议

今天要建立的主链是：

`handlePromptSubmit.ts -> useQueueProcessor.ts -> query.ts -> messages.ts -> REPL onQuery`

---

## 今日学习材料

按这个顺序读：

1. [restored-src/src/utils/handlePromptSubmit.ts](../restored-src/src/utils/handlePromptSubmit.ts)
2. [restored-src/src/hooks/useQueueProcessor.ts](../restored-src/src/hooks/useQueueProcessor.ts)
3. [restored-src/src/query.ts](../restored-src/src/query.ts)
4. [restored-src/src/utils/messages.ts](../restored-src/src/utils/messages.ts)
5. [restored-src/src/screens/REPL.tsx](../restored-src/src/screens/REPL.tsx)

补充定位：

- [restored-src/src/utils/QueryGuard.ts](../restored-src/src/utils/QueryGuard.ts)
- [restored-src/src/utils/queueProcessor.ts](../restored-src/src/utils/queueProcessor.ts)
- [restored-src/src/query/config.ts](../restored-src/src/query/config.ts)
- [restored-src/src/query/deps.ts](../restored-src/src/query/deps.ts)

---

## 学习步骤

### 第一步：先看 turn 提交入口

先读 [handlePromptSubmit.ts](../restored-src/src/utils/handlePromptSubmit.ts)。

这份文件今天最重要，因为它决定了“用户刚提交输入时会发生什么”。

重点只看这些问题：

1. 直接输入路径和 `queuedCommands` 路径怎么区分
2. 为什么会先处理 pasted refs、图片引用、immediate command
3. query 正忙时，为什么有些输入要入队
4. `executeUserInput` 是如何把预处理结果交给 `onQuery`

这份文件反映的是一个非常清晰的职责边界：

- 它不自己实现模型循环
- 不自己处理消息流式协议
- 它负责在“提交瞬间”做调度、清理、排队、保护和转发

你今天要记住一句话：

> `handlePromptSubmit.ts` 是 turn 的入口编排器，不是 query 引擎。

---

### 第二步：理解为什么队列消费要做成 hook

接着读 [useQueueProcessor.ts](../restored-src/src/hooks/useQueueProcessor.ts)。

这份文件很短，但很关键。

你要看清它依赖了什么：

- `queryGuard`
- `subscribeToCommandQueue`
- `getCommandQueueSnapshot`
- `processQueueIfReady`

它的触发条件本质上只有 3 个：

- 当前没有 query 在跑
- 当前没有本地 JSX UI 阻塞
- 队列里确实有命令

结论要写成一句话：

> 队列不是靠 while-loop 死轮询消费，而是通过 `useSyncExternalStore + useEffect` 在“系统空闲”时自然触发。

这和前一天的 `messageQueueManager.ts` 正好闭环。

---

### 第三步：把 query.ts 当成“主循环”来读

然后读 [query.ts](../restored-src/src/query.ts)。

今天不要试图把所有 feature flag 和 recovery path 一次看懂。你只抓主线：

1. `query()` 是一个 async generator
2. 它会 yield `StreamEvent`、`Message`、`ToolUseSummaryMessage` 等事件
3. 真正循环体在 `queryLoop()`
4. 里面维护的是一份跨 iteration 的 `State`

这里最重要的理解是：

> `query.ts` 不是“一次 API 调用的包装”，而是一个可继续、可分支、可恢复的 agentic 循环。

你应该特别注意这些点：

- `messages` 会在循环中不断增长
- `toolUseContext` 也会在循环中持续演化
- 可能发生 auto compact / reactive compact / token budget recovery
- 工具使用和模型继续生成属于同一条主循环的一部分

所以今天一定要写下：

> 在 Claude Code 里，一轮 query 本身就可能包含多次 assistant -> tool_use -> tool_result -> assistant 的连续轨迹。

---

### 第四步：看 query 为什么必须流式产出

继续留在 [query.ts](../restored-src/src/query.ts)，重点看这些概念：

- `AsyncGenerator`
- `StreamEvent`
- `RequestStartEvent`
- `ToolUseSummaryMessage`
- `Terminal` / `Continue`

你要思考：

- 为什么不是一次性返回完整结果
- 为什么 REPL 需要边流边渲染
- 为什么工具执行也会插入这条流

今天的答案应该是：

> 终端 agent 的体验依赖“增量可见性”，所以 query 层必须天然支持流式事件，而不是等所有结果结束再统一返回。

---

### 第五步：messages.ts 不只是“造消息”

再读 [messages.ts](../restored-src/src/utils/messages.ts)。

这份文件极大，不要试图全看。今天只抓 4 类函数：

1. `createUserMessage`
2. `createAssistantMessage`
3. 各种 system / synthetic message builder
4. 流式消息处理与 tool_use / tool_result 配对逻辑

你应该很快会发现，这不是简单的“message factory”。

它实际上负责：

- 消息创建
- 消息规范化
- streaming delta 合并
- tool result pairing
- synthetic message 规则
- queued command 到 message 的映射
- compact boundary 相关处理

所以今天必须明确：

> `messages.ts` 是 Claude Code 的消息协议层，定义了系统内部“什么算一条消息、消息如何合法流转”。

---

### 第六步：重点理解 tool_use / tool_result pairing

继续看 [messages.ts](../restored-src/src/utils/messages.ts)，今天最值得抓住的就是工具消息配对问题。

原因很简单：

- 模型会生成 `tool_use`
- 工具执行后系统要补 `tool_result`
- 流式过程中消息还可能不完整
- compact 或恢复时这些配对关系不能乱

你会看到很多围绕这些事情的逻辑：

- 收集 `tool_use_id`
- 找对应 `tool_result`
- 标记 unresolved / errored
- 处理 orphaned server_tool_use / mcp_tool_use

今天要记住：

> 如果没有一个严谨的消息协议层，Claude Code 的工具轨迹在流式、恢复、compact、多 agent 场景下会非常容易断裂。

---

### 第七步：回到 REPL 看 onQuery 调用点

最后回到 [REPL.tsx](../restored-src/src/screens/REPL.tsx)。

今天只盯这些点：

- `onQueryEvent`
- `onQueryImpl`
- `onQuery`
- `executeQueuedInput`
- `useQueueProcessor`

你要看明白：

1. REPL 是怎样消费 `query()` 产生的流式事件
2. 流式文本、streamingToolUses、spinner、messages 是怎么同步更新的
3. query 忙时为什么会把后续输入重新 `enqueue`
4. 队列清空后怎么自然进入下一轮

这一步的目标不是读懂 REPL 全部 UI，而是把前几天的内容在这里真正闭环：

- 输入是怎么进入系统的
- 队列是怎么控制节奏的
- query 是怎么流出来的
- messages 是怎么落到 REPL 里的

---

## 今日建议时间安排

建议控制在 2 到 3 小时：

1. `handlePromptSubmit.ts`：40 分钟
2. `useQueueProcessor.ts + queueProcessor.ts`：15 分钟
3. `query.ts` 主线阅读：55 分钟
4. `messages.ts` 协议层主线：45 分钟
5. `REPL.tsx` 对照调用点：25 分钟

---

## 今日笔记模板

你可以直接按这个模板记：

```md
# Day 11 - Query 主线与消息协议

## 一句话总结

## 今日主链
- handlePromptSubmit.ts:
- useQueueProcessor.ts:
- query.ts:
- messages.ts:
- REPL.tsx:

## 我理解的 turn 入口
- 直接提交:
- 立即命令:
- 入队:

## 我理解的 query 主循环
- 为什么是 async generator:
- 为什么会多轮 assistant/tool/use/result:
- 为什么会有 compact / recovery:

## 我理解的消息协议层
- createUserMessage:
- createAssistantMessage:
- synthetic message:
- tool_use / tool_result pairing:

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

### 作业 1：画 turn 执行图

画一张最小流程图，至少包含这些节点：

- PromptInput
- handlePromptSubmit
- processUserInput
- onQuery
- query()
- messages.ts
- REPL render

要求你能明确指出：

- 哪一步是入口调度
- 哪一步是主循环
- 哪一步是协议整理
- 哪一步是前台渲染

### 作业 2：解释 handlePromptSubmit 的边界

用 150 到 250 字回答：

为什么 `handlePromptSubmit.ts` 不能被理解为“真正执行 query 的地方”？

要求至少提到：

- 排队
- immediate command
- `processUserInput`
- `onQuery`

### 作业 3：总结 query.ts 的本质

用 200 到 300 字回答：

为什么 `query.ts` 是一个 agentic 主循环，而不是一个普通的 API 请求包装器？

要求至少提到：

- async generator
- 流式事件
- 工具调用
- 多 iteration 状态
- compact 或 recovery

### 作业 4：整理消息类型

从 [messages.ts](../restored-src/src/utils/messages.ts) 中整理 10 种你今天见到的重要消息或消息相关概念。

建议至少覆盖：

- user message
- assistant message
- system message
- stream event
- tool_use
- tool_result
- synthetic message
- queued command
- compact boundary
- tombstone

每个概念后写一句它在系统里的用途。

### 作业 5：解释为什么需要 pairing

用 150 到 250 字回答：

为什么 `tool_use` 和 `tool_result` 的配对在这个项目里必须被严肃处理？

要求至少提到：

- 流式生成
- 恢复
- compact
- 多工具或多 agent 情况下的一致性

---

## 自检清单

今天结束前，确认你能回答：

- `handlePromptSubmit.ts` 和 `query.ts` 的边界是什么
- `useQueueProcessor.ts` 为什么能无轮询地消费队列
- `query.ts` 为什么必须设计成流式生成器
- `messages.ts` 为什么是协议层而不是普通工具库
- 为什么工具消息配对是这个系统稳定性的关键

如果这 5 个问题里你有 4 个能顺畅讲出来，第 11 天就算完成。

---

## 明日预告

第 12 天建议进入工具执行与 Tool orchestration 主线，重点看：

- `services/tools/toolOrchestration.ts`
- `services/tools/StreamingToolExecutor.ts`
- `Tool.ts`
- 代表性工具实现

到那一天，你会把今天的 query 主循环继续推进到：

“模型发出 tool_use 之后，系统到底怎么真正执行工具、处理结果并把控制权交回模型。”
