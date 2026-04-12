# 第 12 天学习文档：工具执行主线与 Tool Orchestration

## 今日目标

今天要解决的问题是：

模型发出 `tool_use` 之后，Claude Code 是怎么真正执行工具、处理权限、产出进度和结果，再把控制权交回主循环的。

读完今天内容后，你应该能回答：

1. `Tool.ts` 定义了哪些公共契约
2. 单个工具调用是怎么从 `tool_use` 走到 `tool_result` 的
3. 为什么有的工具能并发，有的必须串行
4. streaming tool execution 和普通 tool orchestration 的差别是什么

---

## 学习成果要求

今天结束前，你至少要形成这 5 个判断：

1. `Tool.ts` 是工具系统的核心协议层
2. `toolExecution.ts` 负责“单次工具调用”的完整流水线
3. `toolOrchestration.ts` 负责“多个工具调用如何组合执行”
4. `StreamingToolExecutor.ts` 负责“流式 tool_use 到达时的执行与回收”
5. `BashTool` 和 `FileEditTool` 分别代表“复杂执行工具”和“典型编辑工具”

---

## 核心结论

今天最重要的主线可以压缩成一句话：

`query.ts 发现 tool_use -> toolExecution.ts 跑单个工具 -> toolOrchestration.ts / StreamingToolExecutor.ts 管理多工具执行 -> 结果回到 messages/query`

更细一点：

- `Tool.ts` 定义 tool 的输入、输出、权限、渲染、进度、上下文修改等接口
- `toolExecution.ts` 负责权限检查、hooks、进度事件、tool.call、结果包装
- `toolOrchestration.ts` 负责串行/并行地调度多个 tool_use block
- `StreamingToolExecutor.ts` 负责 streaming 场景下边到达边执行工具，并在 fallback/中断时补偿

今天要建立的主链是：

`Tool.ts -> toolExecution.ts -> toolOrchestration.ts -> StreamingToolExecutor.ts -> 代表性工具`

---

## 今日学习材料

按这个顺序读：

1. [restored-src/src/Tool.ts](../restored-src/src/Tool.ts)
2. [restored-src/src/services/tools/toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts)
3. [restored-src/src/services/tools/toolOrchestration.ts](../restored-src/src/services/tools/toolOrchestration.ts)
4. [restored-src/src/services/tools/StreamingToolExecutor.ts](../restored-src/src/services/tools/StreamingToolExecutor.ts)
5. [restored-src/src/tools/BashTool/BashTool.tsx](../restored-src/src/tools/BashTool/BashTool.tsx)
6. [restored-src/src/tools/FileEditTool/FileEditTool.ts](../restored-src/src/tools/FileEditTool/FileEditTool.ts)
7. [restored-src/src/query.ts](../restored-src/src/query.ts)

补充浏览：

- [restored-src/src/tools/BashTool/bashPermissions.ts](../restored-src/src/tools/BashTool/bashPermissions.ts)
- [restored-src/src/tools/FileEditTool/UI.tsx](../restored-src/src/tools/FileEditTool/UI.tsx)
- [restored-src/src/tools/BashTool/UI.tsx](../restored-src/src/tools/BashTool/UI.tsx)

---

## 学习步骤

### 第一步：先把 Tool.ts 当成“接口规范”来读

先读 [Tool.ts](../restored-src/src/Tool.ts)。

今天不要被文件长度吓住，你只抓几个核心概念：

- `ToolPermissionContext`
- `ToolUseContext`
- `ToolResult`
- `ToolProgress`
- `buildTool`

你要看明白：

1. tool 能访问哪些运行时上下文
2. tool 可以怎样返回结果
3. tool 可以怎样发进度
4. tool 怎么声明自己的输入输出 schema
5. tool 怎么声明权限、渲染和错误显示逻辑

这里最值得记住的是：

> 工具不是一个简单 `call(input)` 函数，而是一个带协议、带权限、带 UI 呈现、带上下文修改能力的运行时对象。

今天一定要写下：

> `Tool.ts` 定义的是工具系统的运行时契约，不只是 TypeScript 类型。

---

### 第二步：理解单个工具调用的完整流水线

然后读 [toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts)。

这是今天最重要的文件之一，因为它处理的是“单次工具调用怎么跑完”。

重点只看这些主题：

1. 如何找到 tool definition
2. `validateInput` 在哪里触发
3. `checkPermissions` / hooks 在哪里进入
4. `tool.call()` 在哪里真正执行
5. 进度消息和最终 `tool_result` 怎么被包装出来

你要特别注意这几个事实：

- 工具执行前不是直接 `call`，而是先走一整套验证/权限/钩子链
- 进度消息和最终结果在同一执行流水线里被统一处理
- 工具不仅能返回消息，还能返回 `contextModifier`

这说明：

> 工具执行系统不只是“跑函数”，而是在执行前后都允许插入策略、日志、遥测和上下文变更。

---

### 第三步：看 contextModifier 为什么关键

继续留在 [toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts)，重点看 `contextModifier`。

这是一个很重要的设计点。

为什么重要：

- 有些工具执行完不仅产生结果，还会改变后续工具上下文
- 这种变化不能靠全局副作用乱写
- 所以系统允许工具返回“如何修改 context”的函数

今天你要记住一句话：

> Claude Code 里的工具并不是纯函数，它们可以通过 `contextModifier` 安全地影响后续执行环境。

---

### 第四步：理解多工具 orchestration

接着读 [toolOrchestration.ts](../restored-src/src/services/tools/toolOrchestration.ts)。

这份文件的任务很清楚：

- 收到一组 `tool_use` block
- 按 concurrency-safe 与否分批
- 能并发的读类工具并发执行
- 不能并发的工具串行执行

重点看：

- `partitionToolCalls`
- `runToolsSerially`
- `runToolsConcurrently`

这里最重要的结论是：

> 系统不是“一刀切地并发所有工具”，而是根据工具声明的 `isConcurrencySafe` 做保守调度。

这很关键，因为：

- 读文件、查找类工具通常可并发
- 写文件、改状态、跑复杂 shell 往往必须串行

---

### 第五步：理解 streaming tool executor 的特殊性

然后读 [StreamingToolExecutor.ts](../restored-src/src/services/tools/StreamingToolExecutor.ts)。

这份文件解决的是另一类问题：

- 如果 assistant 的 tool_use 是流式到达的
- 而且 response 还可能 fallback / 中断 / 丢弃
- 系统怎么在保证顺序的同时，尽早开始执行工具

重点只抓这几个设计点：

1. 工具会经历 `queued / executing / completed / yielded` 状态
2. 进度消息会单独缓存并尽快吐给上层
3. fallback 或 abort 时，会生成 synthetic error message
4. sibling tool error 会触发其他工具的补偿性取消

今天你必须记住：

> `StreamingToolExecutor` 不是普通并发器，它是为“工具边流边来、结果还要顺序稳定”这个问题专门设计的。

---

### 第六步：看 query.ts 里是怎么接入两套工具执行模式的

回看 [query.ts](../restored-src/src/query.ts)。

今天只看工具相关调用点：

- 哪里创建 `StreamingToolExecutor`
- 哪里调用 `runTools`
- 什么情况下走 streaming executor
- 什么情况下走普通 orchestration

你要得出的结论是：

> `query.ts` 不自己执行工具，而是根据响应形态选择不同的工具执行后端。

这说明工具系统和 query 主循环之间是明显分层的。

---

### 第七步：看 BashTool 为什么是最复杂代表

然后读 [BashTool.tsx](../restored-src/src/tools/BashTool/BashTool.tsx)。

今天不要求啃完整个文件，但一定要看明白这些点：

- 它通过 `buildTool` 注册
- 它有很重的 input schema 和权限逻辑
- 它有进度上报
- 它会处理 background task、sandbox、timeout、输出持久化
- 它有丰富的 UI 渲染逻辑配套

这说明：

> BashTool 不只是“执行一条命令”，而是终端 agent 里最复杂的一类工具，兼具权限、安全、进度、后台化和结果压缩能力。

今天只要你能说出“为什么 BashTool 复杂”，就算读对了。

---

### 第八步：看 FileEditTool 为什么是典型编辑工具

再读 [FileEditTool.ts](../restored-src/src/tools/FileEditTool/FileEditTool.ts)。

和 BashTool 相比，它更适合帮助你理解典型工具结构：

- `inputSchema / outputSchema`
- `checkPermissions`
- `validateInput`
- `call`
- 配套 UI render 函数

重点看它验证了什么：

- 路径规范化
- deny rule
- 大文件保护
- 文件是否存在
- old_string / new_string 合法性

你会发现：

> 一个“看起来简单的编辑工具”，在真正进入 call 之前也要走很多安全和一致性检查。

这有助于你理解为什么工具系统必须有统一契约。

---

### 第九步：总结工具系统分层

今天最后请你自己把工具层拆成 5 层：

1. 协议层  
   `Tool.ts`

2. 单次执行层  
   `toolExecution.ts`

3. 多工具编排层  
   `toolOrchestration.ts`

4. 流式执行层  
   `StreamingToolExecutor.ts`

5. 工具实现层  
   `BashTool`、`FileEditTool` 等

如果你能把这 5 层讲明白，第 12 天就完成了。

---

## 今日建议时间安排

建议控制在 2 到 3 小时：

1. `Tool.ts`：35 分钟
2. `toolExecution.ts`：45 分钟
3. `toolOrchestration.ts + StreamingToolExecutor.ts`：40 分钟
4. `BashTool.tsx`：35 分钟
5. `FileEditTool.ts`：25 分钟

---

## 今日笔记模板

你可以直接按这个模板记：

```md
# Day 12 - 工具执行与编排

## 一句话总结

## 今日主链
- Tool.ts:
- toolExecution.ts:
- toolOrchestration.ts:
- StreamingToolExecutor.ts:
- BashTool.tsx:
- FileEditTool.ts:

## 我理解的工具契约
- ToolUseContext:
- ToolPermissionContext:
- ToolResult:
- ToolProgress:

## 我理解的执行分层
- 单个工具调用:
- 多工具编排:
- 流式工具执行:

## 两个代表性工具
- BashTool:
- FileEditTool:

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

### 作业 1：画工具执行总图

画一张最小流程图，至少包含这些节点：

- query.ts
- Tool.ts
- toolExecution.ts
- toolOrchestration.ts
- StreamingToolExecutor.ts
- BashTool / FileEditTool
- tool_result

要求你能讲清：

- 哪一层定义契约
- 哪一层执行单个工具
- 哪一层调度多个工具
- 哪一层处理 streaming 场景

### 作业 2：解释 Tool.ts 的职责

用 150 到 250 字回答：

为什么 `Tool.ts` 不能被理解成“仅仅定义工具类型”的文件？

要求至少提到：

- context
- permission
- progress
- render
- result mapping

### 作业 3：总结单次工具调用链

从 [toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts) 中总结单次工具调用的步骤。

建议你至少列出：

1. 找到工具
2. 校验输入
3. 权限检查
4. hooks / 预处理
5. tool.call
6. progress
7. tool_result
8. contextModifier

### 作业 4：比较两种工具执行模式

比较：

- `toolOrchestration.ts`
- `StreamingToolExecutor.ts`

回答这 3 个问题：

1. 它们各自解决什么问题
2. 为什么 streaming 场景不能只靠普通 orchestration
3. 它们如何处理顺序与并发的矛盾

### 作业 5：比较 BashTool 和 FileEditTool

从今天读的两个代表性工具中，各写 4 点对比：

- 输入复杂度
- 权限复杂度
- 进度机制
- 结果呈现方式

最后再写 100 到 150 字总结：

为什么 BashTool 更像“终端执行平台”，而 FileEditTool 更像“受约束的编辑操作”。

---

## 自检清单

今天结束前，确认你能回答：

- `Tool.ts` 为什么是工具系统的核心协议层
- `toolExecution.ts` 为什么不能直接省掉
- `toolOrchestration.ts` 为什么要区分 concurrency-safe 工具
- `StreamingToolExecutor.ts` 为什么是单独的一层
- `BashTool` 和 `FileEditTool` 各自代表了哪类工具复杂度

如果这 5 个问题里你有 4 个能顺畅讲出来，第 12 天就算完成。

---

## 明日预告

第 13 天建议进入 hooks、permissions 与 stop/recovery 交叉主线，重点看：

- `services/tools/toolHooks.ts`
- `utils/hooks.ts`
- `query/stopHooks.ts`
- 权限对工具执行流的切入点

到那一天，你会把今天的工具执行主线继续推进到：

“在工具真正执行前后，hooks、permission、stop 机制是怎样插进去并改变执行结果的。”
