# 第 3 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 3`

## 今日目标

今天正式进入完整 CLI 主流程，核心学习对象是 `restored-src/src/main.tsx`。

完成今天后，你应该能够明确回答：

1. 为什么 `main.tsx` 是完整 CLI 的主装配层
2. 这个文件为什么会在文件头就集中导入大量模块
3. 启动过程中哪些事情必须先做，哪些事情可以延后或并行
4. 命令、工具、权限、插件、skills、MCP 是如何在主流程里被组装起来的
5. 程序最终是如何从初始化走到 REPL 或其他运行路径的

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `main.tsx`
2. 一张“完整 CLI 初始化顺序”清单
3. 一段“从 `cli.tsx` 进入 `main.tsx` 后发生了什么”的说明
4. 一份“必须先做 / 可以并行 / 可以延后”的分类表
5. 一组能验证你是否真正理解主流程装配的自测题

## 核心结论

先记住这几句话：

1. `main.tsx` 不是单一业务模块，而是完整 CLI 的主装配层。
2. 这个文件的重点不是某一个算法，而是“启动顺序”和“系统拼装”。
3. 文件头的大量 import 反映的不是混乱，而是它负责把多个子系统接到一起。
4. 启动性能依赖两个关键策略：尽早做必要校准，以及把能并行的预取任务提前发出去。
5. 第 3 天的重点不是逐行读完 `main.tsx`，而是先建立对主流程结构的整体认知。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/main.tsx](../restored-src/src/main.tsx)
2. [restored-src/src/entrypoints/init.ts](../restored-src/src/entrypoints/init.ts)
3. [restored-src/src/commands.ts](../restored-src/src/commands.ts)
4. [restored-src/src/tools.ts](../restored-src/src/tools.ts)
5. [restored-src/src/replLauncher.tsx](../restored-src/src/replLauncher.tsx)

辅助观察目录：

1. [restored-src/src/state](../restored-src/src/state)
2. [restored-src/src/services](../restored-src/src/services)
3. [restored-src/src/utils](../restored-src/src/utils)
4. [restored-src/src/screens](../restored-src/src/screens)

## 学习步骤

### 步骤 1：先把 `main.tsx` 的角色定位清楚

先读 [restored-src/src/main.tsx](../restored-src/src/main.tsx) 的文件头、注释和 import 区，不急着追业务细节。

你要先回答：

1. 为什么这个文件会 import 这么多模块
2. 为什么它更像“系统装配层”而不是“单点功能实现”
3. 它和第 2 天读过的 `cli.tsx` 是什么关系

你要提炼出的判断是：

`cli.tsx` 负责启动分流，`main.tsx` 负责完整 CLI 的初始化、装配和进入运行态。

### 步骤 2：观察文件头的启动前校准

重点看文件最前面的 top-level side effects 和注释。

特别注意：

1. `profileCheckpoint('main_tsx_entry')`
2. `startMdmRawRead()`
3. `startKeychainPrefetch()`
4. `profileCheckpoint('main_tsx_imports_loaded')`
5. debug/inspection 检查

你要理解的是：

这些动作说明 `main.tsx` 不是“导入后马上干活”，而是在导入阶段就已经非常在意启动性能和运行环境校准。

### 步骤 3：找到真正的 `main()` 主函数

在 [restored-src/src/main.tsx](../restored-src/src/main.tsx) 中定位 `export async function main()`。

阅读时重点看下面几个问题：

1. 它在一开始先解析了哪些 CLI 状态
2. 哪些 flag 必须在 `init()` 之前处理
3. `init()` 为什么处在一个很关键的分界点
4. `init()` 之后为什么还会继续做大量初始化

今天要建立的核心认知：

`main()` 不是“一次性把所有事情做完”，而是把主流程拆成多个阶段，每个阶段解决不同级别的问题。

### 步骤 4：梳理初始化顺序

你不需要今天把所有局部变量都记住，但一定要把顺序梳理出来。

建议按下面 4 段理解：

1. 进入主函数前后的环境和状态预备
2. `init()` 前必须完成的 CLI 级判断
3. `init()` 后的配置、策略、权限、模型、插件、skills、commands、tools 装配
4. 最终进入渲染、setup screens、REPL 或其他运行路径

你要形成的心智模型是：

`main.tsx` 的难点不是单点逻辑，而是“顺序正确”。

### 步骤 5：理解关键装配点

今天不要试图逐行掌握所有功能，只抓主装配点：

1. `init()`
2. `loadPolicyLimits()`
3. `initializeToolPermissionContext(...)`
4. `getTools(...)`
5. `initBuiltinPlugins()`
6. `initBundledSkills()`
7. `getCommands(...)`
8. `showSetupScreens(...)`
9. `launchRepl(...)`

你要回答：

1. 为什么权限上下文要先于工具装配
2. 为什么 bundled plugins 和 bundled skills 要在 `getCommands()` 前注册
3. 为什么 `commands` 和 `agent definitions` 会被并行加载
4. 为什么 `showSetupScreens()` 会成为一个重要节点
5. 为什么最后不是直接渲染，而是由 `launchRepl(...)` 接管

### 步骤 6：把“并行”和“延后”看出来

这是今天最值钱的部分。

请主动在代码里找出：

1. 哪些事情被故意提前发起，用来隐藏等待成本
2. 哪些事情被延后到 `init()` 之后
3. 哪些事情用 `Promise.all(...)` 并行执行
4. 哪些事情是为了避免阻塞 first render

你不需要把所有案例都找全，但至少要抓到这类设计思想：

主流程代码不只是“能跑”，还在努力“尽快跑起来”。

### 步骤 7：确认最终落点是交互运行态

最后回头看 `launchRepl(...)`、`renderAndRun(...)`、`showSetupScreens(...)` 等调用点。

今天你要明确：

1. `main.tsx` 的目标不是停在初始化，而是进入可交互状态
2. REPL 不是一个孤立功能，而是前面整套初始化完成后的运行载体
3. 命令、工具、权限、配置在真正开始交互前已经基本就位

## 今日建议时间安排

1. `15 分钟`：快速浏览 `main.tsx` 文件头和 import 区
2. `20 分钟`：定位 `main()` 并划分阶段
3. `20 分钟`：梳理 `init()` 前后的职责边界
4. `20 分钟`：整理权限、工具、命令、插件、skills 的装配顺序
5. `15 分钟`：观察 `showSetupScreens()` 和 `launchRepl()` 的位置
6. `20 分钟`：完成笔记与作业

总时长建议：`110 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 3 学习笔记

## 1. 我对 main.tsx 的一句话定义

## 2. main.tsx 的主流程分段
1.
2.
3.
4.

## 3. init() 前要解决什么
1.
2.
3.

## 4. init() 后开始装配什么
- policy:
- permission:
- tools:
- plugins:
- skills:
- commands:
- repl:

## 5. 哪些事情被并行或延后了
1.
2.
3.

## 6. 今天我最重要的理解

## 7. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：一句话定义 `main.tsx`

请用你自己的话写一句不超过 60 字的定义。

要求：

1. 必须包含“完整 CLI”或“主流程”
2. 必须体现“装配”而不是“单点功能”
3. 必须说明它承接了第 2 天的入口分流之后的工作

### 作业 2：画出初始化顺序

请按你的理解写出 `main.tsx` 中 6 到 10 步的初始化链路。

要求：

1. 必须包含 `init()`
2. 必须包含权限或 permission context
3. 必须包含 tools 或 commands
4. 必须包含 `showSetupScreens()` 或 `launchRepl()`

### 作业 3：区分先后关系

请回答下面两个问题：

1. 为什么 `initializeToolPermissionContext(...)` 要早于 `getTools(...)`
2. 为什么 `initBuiltinPlugins()` 和 `initBundledSkills()` 要早于 `getCommands(...)`

要求：

1. 不要抄代码
2. 必须用“先后依赖”解释
3. 必须说明如果顺序错了会造成什么问题

### 作业 4：找出并行设计

请从今天阅读的内容里列出至少 3 个“为了启动性能而做的优化点”。

提示方向：

1. 顶层预取
2. 延后初始化
3. `Promise.all(...)`
4. first render 之前和之后的划分

要求：

1. 每个点都要写“优化了什么”
2. 每个点都要写“如果不这样做会怎样”

### 作业 5：画出职责边界

请分别用 3 句话说明下面 3 个文件各自负责什么：

1. `entrypoints/cli.tsx`
2. `main.tsx`
3. `replLauncher.tsx`

要求：

1. 不能混着写
2. 必须指出它们之间的接力关系
3. 必须体现“入口分流 -> 主装配 -> 进入 REPL”这条主线

### 作业 6：写一段 Day 3 总结

写一个 `200-350` 字的小总结，要求包含：

1. 你今天如何重新理解 `main.tsx`
2. 你认为主流程里最关键的两个顺序点是什么
3. 你准备在第 4 天继续追哪个方向

## 自检清单

今天结束前，逐项确认：

- 我知道 `main.tsx` 为什么是完整 CLI 的主装配层
- 我知道文件头那些 top-level side effects 在解决什么问题
- 我知道 `init()` 在主流程里为什么是关键分界点
- 我知道权限、工具、命令、插件、skills 不是乱序拼上的
- 我知道为什么有些初始化要并行，有些要延后
- 我知道 `showSetupScreens()` 和 `launchRepl()` 在主流程中的位置
- 我能复述“从 `cli.tsx` 进入 `main.tsx` 后发生了什么”

## 明日预告

第 4 天建议开始拆分主流程中的两个核心注册表：命令系统和工具系统。

明天的重点文件预计是：

1. [restored-src/src/commands.ts](../restored-src/src/commands.ts)
2. [restored-src/src/tools.ts](../restored-src/src/tools.ts)

明天的目标是把“主流程装配”进一步细化为“命令如何注册”和“工具如何装配并受权限控制”。
