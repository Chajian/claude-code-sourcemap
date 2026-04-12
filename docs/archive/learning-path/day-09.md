# 第 9 天学习文档：状态管理与 REPL 交互主线

## 今日目标

今天只解决一个问题：Claude Code 的交互状态到底放在哪里，状态变化又是怎么驱动 REPL 行为的。

读完今天的内容后，你应该能清楚回答：

1. 全局状态由哪个文件定义
2. React 组件如何读取和更新状态
3. 哪些状态变化会触发额外副作用
4. REPL 为什么能在一个地方同时协调命令、工具、MCP、插件、任务和 UI

---

## 学习成果要求

今天结束前，你至少要形成这 3 个判断：

1. `AppStateStore.ts` 是“状态模型定义”，不是 UI 文件
2. `AppState.tsx` 是“状态接入层”，负责把 store 接进 React
3. `REPL.tsx` 是“状态消费中心”，大量行为都是围绕 `useAppState()` 和 `setAppState()` 展开的

---

## 核心结论

先给今天最重要的结论：

- `state/store.ts` 提供最底层的通用 store 抽象
- `state/AppStateStore.ts` 定义 Claude Code 的全量 `AppState`
- `state/AppState.tsx` 把这个 store 包成 React Provider 和 selector hooks
- `state/onChangeAppState.ts` 负责把状态变化同步到配置、会话元数据、权限模式等外部系统
- `replLauncher.tsx` 只是装配入口，真正的交互核心在 `screens/REPL.tsx`
- `REPL.tsx` 不是单纯“界面组件”，而是 CLI 运行时的前台控制台

今天要建立的不是“每个字段都记住”，而是这条主链：

`AppStateStore.ts -> AppState.tsx -> onChangeAppState.ts -> REPL.tsx`

---

## 今日学习材料

按这个顺序读：

1. [restored-src/src/state/store.ts](../restored-src/src/state/store.ts)
2. [restored-src/src/state/AppStateStore.ts](../restored-src/src/state/AppStateStore.ts)
3. [restored-src/src/state/AppState.tsx](../restored-src/src/state/AppState.tsx)
4. [restored-src/src/state/onChangeAppState.ts](../restored-src/src/state/onChangeAppState.ts)
5. [restored-src/src/state/selectors.ts](../restored-src/src/state/selectors.ts)
6. [restored-src/src/replLauncher.tsx](../restored-src/src/replLauncher.tsx)
7. [restored-src/src/screens/REPL.tsx](../restored-src/src/screens/REPL.tsx)
8. [restored-src/src/context/notifications.tsx](../restored-src/src/context/notifications.tsx)
9. [restored-src/src/context/overlayContext.tsx](../restored-src/src/context/overlayContext.tsx)

补充浏览：

- [restored-src/src/context/mailbox.tsx](../restored-src/src/context/mailbox.tsx)
- [restored-src/src/context/voice.tsx](../restored-src/src/context/voice.tsx)

---

## 学习步骤

### 第一步：先看最底层 store 抽象

先读 [store.ts](../restored-src/src/state/store.ts)。

今天你不用把它当业务代码读，只需要确认它提供了 3 个最基本能力：

- `getState`
- `setState`
- `subscribe`

你要记住一句话：

> 这个项目的全局状态不是 Redux/Zustand，而是自定义轻量 store，再通过 React 的 `useSyncExternalStore` 接进去。

---

### 第二步：精读 AppState 的结构

接着读 [AppStateStore.ts](../restored-src/src/state/AppStateStore.ts)。

这份文件是今天最重要的内容。你不用一次记住全部字段，但一定要学会按“状态域”拆分。

建议你把 `AppState` 粗分成这些区域：

1. 基础运行配置  
   例如 `settings`、`verbose`、`mainLoopModel`

2. 权限与模式  
   例如 `toolPermissionContext`、`isBriefOnly`

3. 命令与工具生态  
   例如 `mcp`、`plugins`、`agentDefinitions`

4. 会话与任务  
   例如 `tasks`、`viewingAgentTaskId`、`foregroundedTaskId`

5. UI 展示状态  
   例如 `expandedView`、`footerSelection`、`spinnerTip`、`notifications`

6. REPL 运行时附加状态  
   例如 `fileHistory`、`attribution`、`sessionHooks`、`speculation`

今天的关键不是背字段，而是理解：

- 为什么这个类型这么大  
  因为它是 REPL 前台统一协调层

- 为什么它既有业务状态又有 UI 状态  
  因为这个项目不是典型前后端分层应用，而是终端 agent 运行时

- 为什么会有很多可选字段  
  因为不同模式下会启用不同能力，比如 remote、bridge、tmux、swarm、MCP

读完这一节后，你应该能说出：

> `AppState` 不是一个单纯的“页面状态对象”，而是整个 CLI 会话运行时的内存模型。

---

### 第三步：看 React 如何接入这个状态系统

然后读 [AppState.tsx](../restored-src/src/state/AppState.tsx)。

重点只看 4 件事：

1. `AppStateProvider`
2. `AppStoreContext`
3. `useAppState(selector)`
4. `useSetAppState()`

这里最值得记的点有：

- Provider 只创建一次 store
- 组件通过 selector 订阅状态切片，而不是整棵状态树
- `useSetAppState()` 给的是稳定引用，适合传给非 UI 逻辑
- `useAppStateStore()` 允许非 React 风格代码直接拿 `getState/setState`

你还要特别注意两点：

1. 文件里明确强调 selector 不要返回新对象  
   这说明作者非常在意终端 UI 的重渲染成本

2. `AppStateProvider` 不只是放 context  
   它还顺手挂了 `MailboxProvider`、`VoiceProvider`，并监听 settings change

结论要写成一句话：

> `AppState.tsx` 的职责不是定义状态，而是把 store 变成 React 可消费的基础设施。

---

### 第四步：理解状态变化后的副作用出口

再读 [onChangeAppState.ts](../restored-src/src/state/onChangeAppState.ts)。

这份文件是今天最容易被忽略，但非常关键的一层。

你要重点理解：

- 权限模式变化后，会同步到 session metadata / SDK / CCR
- `mainLoopModel` 变化会写回 settings 和 bootstrap override
- `expandedView`、`verbose` 这类状态会持久化到全局配置
- `settings` 变化会清理鉴权缓存，并重新应用环境变量

这里反映出一个设计原则：

> 状态本身保持纯粹，副作用集中到统一的 onChange 钩子里做。

这比把副作用散落在几十个 `setAppState()` 调用点里更容易维护。

---

### 第五步：看选择器和派生状态

读 [selectors.ts](../restored-src/src/state/selectors.ts)。

这一步的目标不是看代码量，而是理解“为什么还需要 selector 文件”。

你要注意：

- 有些状态不是原始字段，而是从 `tasks + viewingAgentTaskId` 这类组合里推导出来
- 这种派生逻辑如果直接散落在 REPL 里，会让大文件更难维护

所以今天要记住：

> `AppStateStore.ts` 放原始状态结构，`selectors.ts` 放派生状态规则。

---

### 第六步：看 REPL 是如何被启动的

读 [replLauncher.tsx](../restored-src/src/replLauncher.tsx)。

这个文件很短，但意义很明确：

- 动态导入 `App`
- 动态导入 `REPL`
- 把 `initialState` 和 REPL props 一起挂进去

你需要得出的结论是：

> `replLauncher.tsx` 不是交互逻辑中心，只是把状态容器和 REPL 屏幕挂起来的薄装配层。

---

### 第七步：精读 REPL 的状态消费方式

最后读 [REPL.tsx](../restored-src/src/screens/REPL.tsx)。

这一天不要试图通读全部细节，而是只抓“状态如何被读取和驱动行为”。

建议你重点盯住这些模式：

1. 大量 `useAppState(s => ...)`  
   这说明 REPL 是状态切片消费者

2. `setAppState(prev => ...)`  
   这说明 REPL 直接承担很多运行时状态推进

3. 本地 `useState(...)` 和全局 `AppState` 并存  
   说明作者区分了“全局会话状态”和“局部展示状态”

4. `useMergedTools`、`useMergedCommands`、`useMergedClients`  
   说明 REPL 在运行时动态组合能力池

5. `QueryGuard`、`abortController`、`streamingToolUses`、`streamingThinking`  
   说明 REPL 同时也是一次 query 生命周期的前台控制器

你读 REPL 时可以按 4 个问题拆：

1. 它从 `AppState` 里取了哪些全局字段
2. 它有哪些只属于当前屏幕的局部状态
3. 它怎么组合命令、工具、MCP、插件
4. 它怎么管理 query 执行、打断、恢复、转场

今天不要求把 `REPL.tsx` 全吃透，但至少要看明白：

> 这个文件是“状态消费 + 运行时协调 + 终端 UI 渲染”的三合一中心。

---

### 第八步：补看两个上下文例子

最后浏览：

- [notifications.tsx](../restored-src/src/context/notifications.tsx)
- [overlayContext.tsx](../restored-src/src/context/overlayContext.tsx)

这一步是为了区分两类状态：

- 放进 `AppState` 的全局会话状态
- 用独立 context 承载的局部能力状态

你要想清楚：

- 为什么不是所有状态都塞进 `AppState`
- 为什么通知、overlay、mailbox、voice 这类能力适合做独立 context

结论一般会是：

> `AppState` 负责主会话运行时，局部横切能力则通过单独 context 扩展。

---

## 今日建议时间安排

建议控制在 2 到 3 小时：

1. `store.ts + AppStateStore.ts`：40 分钟
2. `AppState.tsx + onChangeAppState.ts`：35 分钟
3. `selectors.ts + replLauncher.tsx`：15 分钟
4. `REPL.tsx` 主线阅读：50 分钟
5. 笔记整理与作业：30 分钟

---

## 今日笔记模板

你可以直接按这个模板记：

```md
# Day 09 - 状态管理与 REPL

## 一句话总结

## 今天的状态主链
- store.ts:
- AppStateStore.ts:
- AppState.tsx:
- onChangeAppState.ts:
- REPL.tsx:

## AppState 的 6 个状态域
- 基础运行配置：
- 权限与模式：
- 命令与工具生态：
- 会话与任务：
- UI 展示状态：
- REPL 运行时附加状态：

## 关键机制
- useAppState(selector)：
- useSetAppState()：
- onChangeAppState()：
- REPL 中的全局状态与局部状态边界：

## 我今天确认的 3 个设计点
1.
2.
3.

## 我还没弄懂的 3 个问题
1.
2.
3.
```

---

## 今日作业

### 作业 1：画状态流转图

自己画一张最小图，只保留这几个节点：

- `createStore`
- `AppStateProvider`
- `useAppState`
- `setAppState`
- `onChangeAppState`
- `REPL`

要求你能讲清“状态从创建到被消费”的路径。

### 作业 2：给 AppState 分类

从 [AppStateStore.ts](../restored-src/src/state/AppStateStore.ts) 中挑 20 个字段，按今天的 6 个状态域分类。

要求：

- 每类至少列 2 个字段
- 每个字段后面写一句“它为什么属于这一类”

### 作业 3：解释 selector 设计

用 150 到 250 字回答：

为什么 `useAppState(selector)` 要求尽量返回稳定引用，而不鼓励直接返回新对象？

要求你必须结合终端 UI 渲染成本来解释。

### 作业 4：找 5 个状态副作用

从 [onChangeAppState.ts](../restored-src/src/state/onChangeAppState.ts) 里找出 5 个“状态变化 -> 外部动作”的例子。

格式建议：

- 哪个状态变了
- 触发了什么副作用
- 这个副作用为什么不应该直接写在组件里

### 作业 5：拆 REPL 状态

从 [REPL.tsx](../restored-src/src/screens/REPL.tsx) 里各找：

- 8 个通过 `useAppState` 读取的全局状态
- 8 个通过 `useState` 管理的局部状态

最后写 100 字总结：

为什么这两类状态不能混在一起。

---

## 自检清单

今天结束前，确认你能回答：

- `AppStateStore.ts` 和 `AppState.tsx` 的职责差异是什么
- 为什么这个项目要自己实现 store，而不是直接依赖 Redux 风格方案
- `onChangeAppState.ts` 解决了什么架构问题
- `REPL.tsx` 为什么不只是一个“界面文件”
- 哪些状态适合进 `AppState`，哪些更适合独立 context

如果这 5 个问题里你有 4 个能顺畅讲出来，第 9 天就算完成。

---

## 明日预告

第 10 天建议进入“会话恢复、消息队列与输入处理主线”，重点看：

- `utils/messageQueueManager.ts`
- `utils/processUserInput/`
- `utils/sessionRestore.ts`
- `utils/sessionStorage.ts`

到那一天，你会把今天的状态系统继续往前推进到：

“用户输入是怎么真正变成一次 query、一次工具调用和一次可恢复会话的。”
