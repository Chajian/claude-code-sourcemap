# 第 13 天学习文档：Hooks、Permission 与 Stop/Recovery 交叉主线

## 今日目标

今天要解决的问题是：

在工具真正执行前后，hooks、permission、stop 机制是怎样插进执行链路，并改变工具和 query 最终走向的。

读完今天内容后，你应该能回答：

1. hooks 在这个项目里到底是“扩展点”还是“执行控制器”
2. PreToolUse / PostToolUse / Stop hooks 分别卡在什么位置
3. permission 决策为什么会和 hooks 深度交织
4. stop hook 为什么能触发继续、阻断、补充上下文甚至重试

---

## 学习成果要求

今天结束前，你至少要形成这 5 个判断：

1. `utils/hooks.ts` 是全局 hook 基础设施，不只是几个帮助函数
2. `services/tools/toolHooks.ts` 是工具执行链上的 hook 适配层
3. `toolExecution.ts` 里真正把 hooks、permission、tool.call 串在了一起
4. `query/stopHooks.ts` 负责 turn 结束时的 stop hook 编排
5. hooks 在这个项目里不仅能记录信息，还能阻断、改写、恢复执行路径

---

## 核心结论

今天最重要的主线是：

`toolExecution.ts -> PreToolUse hooks -> permission decision -> tool.call -> PostToolUse hooks -> query.ts -> Stop hooks`

你要明确：

- hooks 不是外围小插件，而是执行链的一部分
- permission 不是单独的一步，而是会被 hooks 提前影响、事后补偿
- stop hooks 不只是“收尾日志”，而是会决定这一轮是否继续

今天要建立的主链是：

`utils/hooks.ts -> services/tools/toolHooks.ts -> toolExecution.ts -> query/stopHooks.ts -> query.ts`

---

## 今日学习材料

按这个顺序读：

1. [restored-src/src/utils/hooks.ts](../restored-src/src/utils/hooks.ts)
2. [restored-src/src/services/tools/toolHooks.ts](../restored-src/src/services/tools/toolHooks.ts)
3. [restored-src/src/services/tools/toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts)
4. [restored-src/src/query/stopHooks.ts](../restored-src/src/query/stopHooks.ts)
5. [restored-src/src/query.ts](../restored-src/src/query.ts)

补充浏览：

- [restored-src/src/utils/permissions/permissions.ts](../restored-src/src/utils/permissions/permissions.ts)
- [restored-src/src/utils/permissions/PermissionResult.ts](../restored-src/src/utils/permissions/PermissionResult.ts)
- [restored-src/src/utils/hooks/postSamplingHooks.ts](../restored-src/src/utils/hooks/postSamplingHooks.ts)

---

## 学习步骤

### 第一步：把 hooks.ts 当成全局 hook 平台来读

先读 [utils/hooks.ts](../restored-src/src/utils/hooks.ts)。

这份文件很大，但今天不要细抠所有 hook event。你只要先建立一个判断：

> 这不是一个“辅助脚本调用文件”，而是整个 hook 平台的核心实现。

你要重点看明白：

- hook input 是怎么统一构造的
- hook 可以来自哪些来源
- command / prompt / http / function / callback 这些 hook 类型怎么接入
- 交互模式下为什么所有 hook 都要求 trust
- 哪些 hook 在 REPL 内执行，哪些会走 outside-REPL 路径

今天的关键不是记所有 hook 名称，而是理解：

> 项目把 hook 当成了一级扩展机制，而不是零散的回调。

---

### 第二步：只梳理 hook 的事件分类

继续留在 [utils/hooks.ts](../restored-src/src/utils/hooks.ts)，这次只做一件事：

把 hook 事件分成 4 类。

建议你这样分：

1. 工具前后事件  
   例如 `PreToolUse`、`PostToolUse`、`PostToolUseFailure`

2. turn / session 生命周期事件  
   例如 `UserPromptSubmit`、`Stop`、`SessionStart`、`SessionEnd`

3. 环境与配置事件  
   例如 `ConfigChange`、`CwdChanged`、`FileChanged`、`InstructionsLoaded`

4. 特殊系统事件  
   例如 `PermissionRequest`、`Elicitation`、`WorktreeCreate`

这一步不是为了分类本身，而是为了让你建立一个直觉：

> hooks 覆盖的不只是工具调用，而是整个 Claude Code 生命周期。

---

### 第三步：看 toolHooks.ts 为什么单独存在

然后读 [services/tools/toolHooks.ts](../restored-src/src/services/tools/toolHooks.ts)。

这份文件的作用是把通用 hooks 基础设施接到“工具执行链”上。

重点看：

- `runPostToolUseHooks`
- `runPostToolUseFailureHooks`
- 与 `executePreToolHooks` / `executePostToolHooks` 的衔接

这里最重要的认识是：

> hooks 平台是通用的，但工具执行需要一层专门适配器把 hook 结果转成消息、上下文和控制流。

你会看到这层适配器做的事情包括：

- 生成 attachment message
- 转换 blockingError
- 传递 additionalContexts
- 对 MCP 工具结果做更新

所以 `toolHooks.ts` 的定位应该记成：

> 工具链上的 hook 编排适配层。

---

### 第四步：回到 toolExecution.ts 看 hooks 和 permission 如何交织

再读 [toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts)。

今天你只追这一条链：

1. `runPreToolUseHooks`
2. permission resolve
3. `tool.call`
4. `runPostToolUseHooks`
5. 失败时 `runPostToolUseFailureHooks`

你要看明白几个关键点：

- PreToolUse hooks 可以返回 updatedInput
- hooks 甚至可以给出 permission result
- permission decision 之后，不同分支会走 allow / ask / deny
- auto mode classifier denial 还会触发 `PermissionDenied` hooks
- 成功和失败分别走不同 post-hook 路径

今天最值得写进笔记的一句话是：

> 在 Claude Code 里，hooks 和 permission 不是前后串联的两步，而是相互影响、共同决定工具命运的一套执行机制。

---

### 第五步：看 permission 在工具执行中的真实位置

继续留在 [toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts)，这次专门看 permission。

你要注意：

- tool-specific permission 和 general permission 都会参与
- permission 决策会带 `decisionReason`
- `decisionReason` 又会影响 telemetry、hook message 和后续分支
- `ask`、`allow`、`deny` 不只是 UI 结果，它们会直接改变是否产出 `tool_result`、是否继续 turn

今天应该形成的结论是：

> permission 在这个系统里不是“弹不弹框”的问题，而是一次完整的执行决策对象。

---

### 第六步：理解 stopHooks.ts 为什么不是普通收尾逻辑

然后读 [query/stopHooks.ts](../restored-src/src/query/stopHooks.ts)。

你要重点看：

- `handleStopHooks`
- `executeStopHooks`
- blockingErrors
- preventContinuation
- stop hook summary message

这份文件最容易被低估，因为很多人会以为 stop hook 只是“收尾通知”。

实际上它做了很多事：

- 构造 turn-end 的 hook context
- 执行 stop hooks 并收集 progress / error / output
- 产出 blocking error message
- 根据结果决定本轮是否继续
- 触发 prompt suggestion、memory extraction、auto-dream 等 turn-end side work

所以今天一定要写下：

> stop hook 是 query 末端的重要控制点，而不是旁路日志。

---

### 第七步：回到 query.ts 看 stop hook 如何改变主循环

最后回到 [query.ts](../restored-src/src/query.ts)。

今天只看 stop hook 相关路径：

- 哪里调用 `handleStopHooks`
- 什么时候 skip stop hooks
- `preventContinuation` 后会怎样
- `blockingErrors` 后为什么可能进入新的 continue 分支

这里最重要的认识是：

> stop hook 的结果会直接反馈回 query 主循环，甚至改变 transition reason 和后续恢复路径。

也就是说，hook 不是“执行后附加消息”，而是主循环状态机的一部分。

---

### 第八步：总结 hooks 的 3 种能力

今天最后把 hooks 的能力归纳成 3 类：

1. 观测  
   记录执行、发 telemetry、补充附件信息

2. 变更  
   修改输入、附加上下文、更新 MCP 输出

3. 控制  
   阻断、拒绝、要求停止继续、触发重试或恢复

如果你能把这 3 类能力讲清楚，第 13 天就完成了。

---

## 今日建议时间安排

建议控制在 2 到 3 小时：

1. `utils/hooks.ts` 主体浏览：50 分钟
2. `toolHooks.ts`：25 分钟
3. `toolExecution.ts` 中 hooks/permission 交叉逻辑：45 分钟
4. `stopHooks.ts`：30 分钟
5. `query.ts` 对照 stop hook 分支：20 分钟

---

## 今日笔记模板

你可以直接按这个模板记：

```md
# Day 13 - Hooks、Permission、Stop

## 一句话总结

## 今日主链
- utils/hooks.ts:
- toolHooks.ts:
- toolExecution.ts:
- stopHooks.ts:
- query.ts:

## 我理解的 hooks 平台
- hook 来源:
- hook 类型:
- hook 输入:
- trust 约束:

## 我理解的执行切入点
- PreToolUse:
- Permission:
- PostToolUse:
- PostToolUseFailure:
- Stop:

## hooks 的 3 种能力
- 观测:
- 变更:
- 控制:

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

### 作业 1：画 hooks 插入点总图

画一张最小流程图，至少包含这些节点：

- UserPromptSubmit
- PreToolUse
- PermissionRequest
- tool.call
- PostToolUse / PostToolUseFailure
- Stop
- query continue / terminate

要求你能明确指出：

- 哪些 hook 在工具前
- 哪些 hook 在工具后
- 哪些 hook 在整轮 turn 结束时执行

### 作业 2：解释为什么 hooks 不是旁路机制

用 180 到 260 字回答：

为什么这个项目里的 hooks 不能被理解成“执行完成后顺手通知一下”的旁路机制？

要求至少提到：

- updatedInput
- blockingError
- preventContinuation
- permission decision

### 作业 3：总结 hooks 和 permission 的交叉点

从 [toolExecution.ts](../restored-src/src/services/tools/toolExecution.ts) 中列出 5 个 hooks 与 permission 相互影响的点。

例如你应该覆盖其中一部分：

- hook 提供 permission result
- hook 修改输入后进入权限判断
- permission deny 后触发 PermissionDenied hooks
- hook 决策如何影响 message 输出
- decisionReason 如何进入 telemetry

### 作业 4：解释 stop hooks 的真实作用

用 180 到 260 字回答：

为什么 `stopHooks.ts` 不是普通收尾逻辑？

要求至少提到：

- blockingErrors
- preventContinuation
- stop summary
- 对 query 主循环的反馈

### 作业 5：列 10 个 hook 事件

从 [utils/hooks.ts](../restored-src/src/utils/hooks.ts) 中列 10 个你今天见到的 hook event，并按下面 3 类归类：

- 工具链事件
- 生命周期事件
- 环境/系统事件

每个事件后写一句“它介入了哪个阶段”。

---

## 自检清单

今天结束前，确认你能回答：

- `utils/hooks.ts` 为什么是 hook 平台而不是工具库
- `toolHooks.ts` 为什么要单独存在
- hooks 和 permission 为什么是交叉而不是串联关系
- `stopHooks.ts` 为什么能改变 query 后续分支
- hooks 在这个项目里有哪些“控制能力”

如果这 5 个问题里你有 4 个能顺畅讲出来，第 13 天就算完成。

---

## 明日预告

第 14 天做总复盘，不再继续钻细节，而是把前 13 天内容压缩成：

- 一张总架构图
- 一份术语表
- 一份二开入口索引
- 一份“我已经真正搞懂 Claude Code 哪些层”的总结
