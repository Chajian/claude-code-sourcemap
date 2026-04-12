# 第 2 天学习文档
项目：`claude-code-sourcemap`  
日期：`Day 2`

## 今日目标

今天不直接啃 `main.tsx` 的大而全实现，而是先把 CLI 启动链路的分流逻辑看清楚。
完成今天后，你应该能清楚回答：

1. 为什么第 2 天先看 `restored-src/src/entrypoints/cli.tsx`
2. 启动前的环境变量和 top-level side effects 在做什么
3. `main()` 的职责边界是什么
4. 什么是 fast-path，为什么它对启动性能很重要
5. 代码里到底分流了哪些启动路径

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `cli.tsx`
2. 一张启动分流清单
3. 一段“从命令行到 `main.js`”的路径说明
4. 一份“为什么先读 `cli.tsx` 再读 `main.tsx`”的理由
5. 一组能验证你是否真正看懂分流的自测题

## 核心结论

先记住这几句话：

1. `restored-src/src/entrypoints/cli.tsx` 是 CLI 的门卫，它先判断命令是不是特殊入口，再决定是否加载完整 CLI。
2. 这个文件在顶层就做了少量环境准备，这些准备要早于后续模块加载，否则某些开关会失效。
3. `main()` 的主要职责不是“处理所有业务”，而是“先分流，再在必要时加载完整 CLI”。
4. fast-path 的本质是“尽量少加载、尽量早返回、尽量别碰重模块”。
5. 第 2 天先读 `cli.tsx`，是因为它决定了后面到底会不会进入 `main.tsx`，以及会以哪条路进入。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/entrypoints/cli.tsx](../restored-src/src/entrypoints/cli.tsx)
2. [restored-src/src/main.tsx](../restored-src/src/main.tsx)

辅助观察目录：

1. [restored-src/src/entrypoints](../restored-src/src/entrypoints)
2. [restored-src/src/utils](../restored-src/src/utils)
3. [restored-src/src/cli](../restored-src/src/cli)

## 学习步骤

### 步骤 1: 先把入口文件定位清楚

先读 [restored-src/src/entrypoints/cli.tsx](../restored-src/src/entrypoints/cli.tsx)，重点看它的角色，而不是细节实现。

你要先回答：

1. 为什么这个文件不是普通业务模块，而是启动分流入口
2. 为什么它会比 `main.tsx` 更适合作为第 2 天起点
3. 为什么这里可以看到“命令级路由”，而不是“交互主循环”

你要提炼出的判断是：

`cli.tsx` 是启动时的第一道路由层，先决定要不要进入完整 CLI。

### 步骤 2: 观察启动前的顶层副作用

只看文件开头的 top-level side effects，不要急着看后面的命令分支。

重点观察：

1. `COREPACK_ENABLE_AUTO_PIN` 为什么要在最前面改
2. `CLAUDE_CODE_REMOTE === 'true'` 时为什么要追加 `NODE_OPTIONS`
3. `ABLATION_BASELINE` 为什么会在顶层批量补环境变量
4. 为什么这些动作不能等到 `main()` 里再做

你要理解的是：

这些顶层动作不是业务逻辑，而是启动前的环境校准，目的是在模块继续加载前把运行条件摆正。

### 步骤 3: 理解 `main()` 的职责

继续看 `cli.tsx` 里的 `main()`，把它理解成“启动调度器”，不要理解成“主业务函数”。

重点回答：

1. 它为什么先读取 `process.argv`
2. 它为什么先处理极少数 fast-path
3. 它为什么在最后才动态 import `../main.js`
4. 它为什么要在导入 `main.js` 之前先准备早期输入捕获

你要得出的结论是：

`main()` 的职责是按参数分流，并且只在必要时才把重型 CLI 拉起来。

### 步骤 4: 把 fast-path 画成一棵分流树

把下面这些路径整理成一棵从上到下的分流树，不要只记名字：

1. `--version` / `-v` / `-V`
2. `--dump-system-prompt`
3. `--claude-in-chrome-mcp`
4. `--chrome-native-host`
5. `--computer-use-mcp`
6. `--daemon-worker=<kind>`
7. `remote-control` / `rc` / `remote` / `sync` / `bridge`
8. `daemon`
9. `ps` / `logs` / `attach` / `kill` / `--bg` / `--background`
10. `new` / `list` / `reply`
11. `environment-runner`
12. `self-hosted-runner`
13. `--worktree` + `--tmux`
14. 最终 fallback 到 `main.js`

你要注意的不是“有哪些名字”，而是“为什么这些入口会被放在前面”。

### 步骤 5: 解释分流顺序背后的原则

整理每一类路径为什么要提前处理：

1. 纯信息输出类，如 `--version`
2. 工具导出类，如 `--dump-system-prompt`
3. 专用宿主类，如 chrome/native host
4. 后台工作类，如 daemon worker 和 daemon
5. 会话管理类，如 bg sessions
6. 模板/任务类，如 templates
7. 受环境约束的 runner 类
8. 需要在完整 CLI 之前抢先处理的 worktree + tmux
9. 其余命令统一交给 `main.js`

你要形成的心智模型是：

分流顺序体现的是启动成本、执行环境和依赖重量的排序。

### 步骤 6: 再回头看 `main.tsx`

只做少量对照，不要今天深入展开。

你只需要确认：

1. `main.tsx` 是完整 CLI 逻辑的载体
2. 它适合在入口分流之后再加载
3. `cli.tsx` 的价值之一，就是尽量避免无关路径触发 `main.tsx` 的大规模模块初始化

今天不需要继续追 `main.tsx` 的全部内部流程，先把入口和路由关系弄懂就够了。

## 时间安排

1. `10 分钟`: 先读 `cli.tsx` 的文件头和注释
2. `20 分钟`: 梳理顶层副作用和环境变量调整
3. `20 分钟`: 拆解 `main()` 的职责边界
4. `25 分钟`: 画出启动分流树
5. `15 分钟`: 对照少量 `main.tsx`
6. `20 分钟`: 完成笔记和作业

总时长建议：`90 分钟`

## 笔记模板

你可以直接照下面这个模板填写：

```md
# Day 2 学习笔记

## 1. 我对 cli.tsx 的一句话定义

## 2. 启动前的 top-level side effects
1.
2.
3.

## 3. main() 的职责边界
1.
2.
3.

## 4. 启动分流树
- --version:
- --dump-system-prompt:
- chrome/native host:
- daemon worker:
- remote-control/bridge:
- daemon:
- bg sessions:
- templates:
- environment-runner:
- self-hosted-runner:
- worktree + tmux:
- fallback to main.js:

## 5. 为什么要先读 cli.tsx 再读 main.tsx

## 6. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1: 用一句话定义入口文件

请用你自己的话写一句不超过 60 字的定义。

要求：

1. 必须包含 `CLI` 或 `启动`
2. 必须体现“先分流再进入主逻辑”
3. 必须说明它不是最终业务实现

### 作业 2: 画出 fast-path 清单

不要抄代码，按理解列出 fast-path。

要求：

1. 至少列出 8 条路径
2. 必须包含 `--version`
3. 必须包含 `--dump-system-prompt`
4. 必须包含 `daemon worker`
5. 必须包含 `main.js` fallback

### 作业 3: 解释顺序

请回答：为什么 `--version` 要放在最前面，而不是和 `daemon` 放在一起？

要求：

1. 用自己的语言解释
2. 至少提到一次“零或极少模块加载”
3. 至少提到一次“避免进入完整 CLI”

### 作业 4: 区分职责

把下面两个问题分开回答：

1. `cli.tsx` 负责什么
2. `main.tsx` 负责什么

要求：

1. 不允许用同一段话糊过去
2. 必须指出两者的边界
3. 必须说明为什么今天不深入 `main.tsx`

### 作业 5: 还原一条启动链路

任选一条路径，写出从命令行到结果的 4 到 6 步链路。

可选路径：

1. `claude --version`
2. `claude --dump-system-prompt`
3. `claude daemon`
4. `claude remote-control`
5. `claude ps`
6. `claude --worktree --tmux`

要求：

1. 每一步都要说清楚“为什么会走到这里”
2. 必须提到至少一个动态 import
3. 必须提到最后是“提前返回”还是“进入 `main.js`”

### 作业 6: 排序题

把下面这些路径按你理解的优先级重新排序，并解释原因：

1. `--version`
2. `daemon`
3. `--dump-system-prompt`
4. `main.js`
5. `remote-control`
6. `bg sessions`

要求：

1. 排序结果必须合理
2. 每个位置至少写一句理由

## 自检清单

今天结束前，逐项确认：

- 我能说清 `cli.tsx` 为什么是第 2 天入口
- 我能说清哪些逻辑属于启动前的环境校准
- 我能说清 `main()` 和 `main.js` 的关系
- 我能说清 fast-path 的含义
- 我能完整列出主要分流路径
- 我能解释为什么这些分流在进入完整 CLI 前就要处理
- 我能不用看代码，复述一条完整的启动链路

## 明日预告

第 3 天将继续沿着入口之后的主线往下看，重点会进入完整 CLI 的内部组织方式。

明天的重点文件预计是：

1. [restored-src/src/main.tsx](../restored-src/src/main.tsx)

明天的目标是把“入口分流”进一步推进到“主流程结构”和“核心初始化顺序”。
