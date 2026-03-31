# 第 7 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 7`

## 今日目标

今天进入 MCP，也就是 Claude Code 把本地内置能力扩展到外部服务器能力的那一层。

核心学习对象是：

1. `restored-src/src/services/mcp/`
2. `restored-src/src/tools/ListMcpResourcesTool/`
3. `restored-src/src/tools/ReadMcpResourceTool/`
4. `restored-src/src/commands/mcp/`

完成今天后，你应该能够明确回答：

1. MCP 配置是从哪里来的
2. MCP server 是如何被连接、认证和管理的
3. MCP 提供的 tools / commands / resources 是如何进入 Claude Code 的
4. 为什么 MCP 既影响命令系统，也影响工具系统
5. 本地工具池和外部 MCP 能力是如何被统一装配的

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `services/mcp`
2. 一张“配置 -> 连接 -> 资源/工具暴露”主线图
3. 一份“本地工具 vs MCP 工具”的比较
4. 一份“MCP 命令层和工具层分别负责什么”的说明
5. 一组能验证你是否真正理解 MCP 接入流程的自测题

## 核心结论

先记住这几句话：

1. `services/mcp/` 是 Claude Code 的外部能力接入层，不只是一个网络客户端目录。
2. MCP 不只是“外部工具调用”，它还包括配置解析、server 连接、认证、资源读取、skills 暴露和命令管理。
3. MCP 的能力进入 Claude Code 后，不会单独悬空存在，而是被接进现有的 commands / tools / resources 体系。
4. `ListMcpResourcesTool` 和 `ReadMcpResourceTool` 说明资源型能力和工具型能力并不是一回事。
5. 第 7 天的重点不是协议细节，而是弄清“外部 server 能力如何被接入主流程”。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/services/mcp/client.ts](../restored-src/src/services/mcp/client.ts)
2. [restored-src/src/services/mcp/config.ts](../restored-src/src/services/mcp/config.ts)
3. [restored-src/src/services/mcp/types.ts](../restored-src/src/services/mcp/types.ts)
4. [restored-src/src/services/mcp/claudeai.ts](../restored-src/src/services/mcp/claudeai.ts)
5. [restored-src/src/services/mcp/utils.ts](../restored-src/src/services/mcp/utils.ts)
6. [restored-src/src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts](../restored-src/src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts)
7. [restored-src/src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts](../restored-src/src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts)
8. [restored-src/src/commands/mcp/index.ts](../restored-src/src/commands/mcp/index.ts)
9. [restored-src/src/commands/mcp/addCommand.ts](../restored-src/src/commands/mcp/addCommand.ts)

辅助观察目录：

1. [restored-src/src/services/mcp](../restored-src/src/services/mcp)
2. [restored-src/src/commands/mcp](../restored-src/src/commands/mcp)
3. [restored-src/src/tools/ListMcpResourcesTool](../restored-src/src/tools/ListMcpResourcesTool)
4. [restored-src/src/tools/ReadMcpResourceTool](../restored-src/src/tools/ReadMcpResourceTool)

## 学习步骤

### 步骤 1：先把 `services/mcp` 的角色定位清楚

先整体浏览 [restored-src/src/services/mcp](../restored-src/src/services/mcp) 目录。

你要先回答：

1. 为什么这里不只是一个单纯的 client 文件
2. 为什么这里会同时出现 config、auth、headers、types、utils、connection manager
3. 为什么 MCP 在 Claude Code 里是一个完整子系统

你要提炼出的判断是：

`services/mcp` 负责把外部 server 能力转化成 Claude Code 可消费的工具、命令和资源。

### 步骤 2：先看配置入口

阅读 [restored-src/src/services/mcp/config.ts](../restored-src/src/services/mcp/config.ts) 和 [restored-src/src/services/mcp/types.ts](../restored-src/src/services/mcp/types.ts)。

重点关注：

1. MCP config 长什么样
2. `parseMcpConfig(...)`
3. `parseMcpConfigFromFilePath(...)`
4. `getClaudeCodeMcpConfigs(...)`
5. `filterMcpServersByPolicy(...)`
6. `dedupClaudeAiMcpServers(...)`

你要理解：

MCP 不可能直接连“任意 server”，它必须先经过配置解析、去重、策略过滤和作用域整理。

### 步骤 3：理解配置来源和作用域

继续围绕 config 层思考：

1. 本地配置从哪里读
2. dynamic MCP config 从哪里来
3. Claude AI 侧下发的 MCP config 是如何并入的
4. enterprise / policy 为什么会影响 server 可用性

你要形成的认知是：

MCP 配置不是单一来源，而是“本地 + 动态 + 平台 + 策略”共同决定的结果。

### 步骤 4：读 `client.ts`，看连接和暴露主线

开始阅读 [restored-src/src/services/mcp/client.ts](../restored-src/src/services/mcp/client.ts)。

今天不要试图一次吃掉全部细节，重点抓 4 件事：

1. server 如何被连接
2. tools / commands / resources 如何从 server 拿到
3. auth 失败时会发生什么
4. resource tools 为什么要被单独补进去

重点定位：

1. `getMcpToolsCommandsAndResources(...)`
2. `prefetchAllMcpResources(...)`
3. auth cache 相关逻辑
4. resource tools 注入逻辑

### 步骤 5：理解 MCP tools 和 MCP resources 的区别

重点读：

1. [restored-src/src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts](../restored-src/src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts)
2. [restored-src/src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts](../restored-src/src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts)

你要回答：

1. 为什么列资源和读资源会被做成独立工具
2. 为什么 resources 不等同于普通 MCP tool
3. 为什么这些工具是 MCP 外围“辅助入口”而不是业务工具本身

你要形成的心智模型是：

MCP server 不只是暴露“可执行工具”，还可能暴露“可读取资源”，Claude Code 需要分别处理。

### 步骤 6：看 MCP 如何影响命令层

阅读 [restored-src/src/commands/mcp/index.ts](../restored-src/src/commands/mcp/index.ts) 和 [restored-src/src/commands/mcp/addCommand.ts](../restored-src/src/commands/mcp/addCommand.ts)。

重点关注：

1. `/mcp` 命令在用户层解决什么问题
2. 添加 MCP config 为什么属于命令层而不是工具层
3. 为什么命令层会负责修改配置文件、引导认证或管理 server

你要理解：

命令层负责“管理和配置外部能力”，工具层负责“调用和消费外部能力”。

### 步骤 7：把 MCP 接回第 4 天的命令和工具注册表

今天一定要回想第 4 天内容，把 MCP 接回命令和工具系统：

1. MCP skill commands 如何从命令集里被筛出
2. MCP tools 如何并入工具池
3. resource tools 为什么会额外注入
4. 为什么 MCP server 级 deny 规则会影响工具可见性

你要形成的整体图景是：

MCP 不是旁路系统，而是被整合进 Claude Code 已有的能力装配框架。

### 步骤 8：理解主流程里的 MCP 连接时机

回看 `main.tsx` 中关于 MCP 的逻辑。

重点回答：

1. 为什么会有 MCP config promise
2. 为什么资源预取会被延后到 trust dialog 之后
3. 为什么 regular MCP 和 Claude AI MCP 要分批连接
4. 为什么连接后还要对 commands / resources 再做排除过滤

你要理解：

MCP 接入不只是“拿到配置就连”，而要兼顾信任、策略、性能和冲突消解。

### 步骤 9：总结外部能力接入的完整主线

今天最后请把主线整理成一句长链：

配置来源 -> 解析与过滤 -> server 连接 -> 认证处理 -> tools / commands / resources 生成 -> 注入主流程 -> 受权限和模式继续约束。

## 今日建议时间安排

1. `15 分钟`：先浏览 `services/mcp` 目录结构
2. `25 分钟`：阅读 `config.ts` 和 `types.ts`
3. `30 分钟`：阅读 `client.ts`，抓连接与暴露主线
4. `15 分钟`：阅读 `ListMcpResourcesTool` 和 `ReadMcpResourceTool`
5. `20 分钟`：阅读 `commands/mcp`，理解管理入口
6. `20 分钟`：回接第 4 天和第 3 天主流程
7. `20 分钟`：完成笔记与作业

总时长建议：`145 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 7 学习笔记

## 1. 我对 services/mcp 的一句话定义

## 2. MCP 配置来源
- local:
- dynamic:
- claude ai:
- policy / enterprise:

## 3. MCP 接入主线
1.
2.
3.
4.
5.

## 4. MCP tools 和 MCP resources 的区别

## 5. MCP 对命令层的影响

## 6. MCP 对工具层的影响

## 7. 我今天最重要的理解

## 8. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：定义 MCP 子系统

请用你自己的话解释 `services/mcp` 在 Claude Code 中的角色。

要求：

1. 不超过 80 字
2. 必须体现“外部能力接入”
3. 必须体现 tools / commands / resources 中至少两个

### 作业 2：列出 MCP 配置来源

请按你的理解列出 MCP 配置可能来自哪些来源。

要求：

1. 至少包含本地配置
2. 至少包含动态配置或 CLI 注入
3. 至少包含 Claude AI 相关来源
4. 必须提到 policy / enterprise 过滤

### 作业 3：解释 `getMcpToolsCommandsAndResources(...)`

请回答：这个函数为什么是 MCP 接入的关键节点？

要求：

1. 至少提到 server 连接
2. 至少提到 tools / commands / resources 生成
3. 至少提到 auth 或失败处理
4. 至少提到 resource tools 的补充注入

### 作业 4：比较 MCP tools 和 MCP resources

请比较：

1. MCP tool 是什么
2. MCP resource 是什么
3. 为什么 Claude Code 要为 resources 单独提供 `ListMcpResourcesTool` 和 `ReadMcpResourceTool`

要求：

1. 不能只写“一个能执行，一个能读取”
2. 必须体现两者在接入方式和使用方式上的差异

### 作业 5：解释 MCP 为什么同时影响命令层和工具层

请分别回答：

1. MCP 为什么会影响命令系统
2. MCP 为什么会影响工具系统

要求：

1. 至少提到 MCP skills 或命令过滤
2. 至少提到工具池组装
3. 必须说明两层影响方式不同

### 作业 6：还原一条 MCP 接入链

请写出一条从 MCP 配置到最终能力暴露的 6 到 9 步链路。

要求：

1. 必须包含配置解析
2. 必须包含策略过滤
3. 必须包含连接或认证
4. 必须包含 tools / commands / resources 中至少两个
5. 必须包含主流程注入

### 作业 7：写一段 Day 7 总结

写一个 `220-400` 字的小总结，要求包含：

1. 你今天如何理解 MCP 子系统
2. 你认为 MCP 接入里最关键的一个设计点是什么
3. 你准备在第 8 天继续追哪条主线

## 自检清单

今天结束前，逐项确认：

- 我知道 `services/mcp` 不只是一个客户端文件夹
- 我知道 MCP 配置不是单一来源
- 我知道 MCP server 连接前需要经过配置解析和策略过滤
- 我知道 tools / commands / resources 是三类不同暴露面
- 我知道为什么 resources 需要独立工具入口
- 我知道 `/mcp` 这类命令属于管理层而不是执行层
- 我知道 MCP 会同时影响命令系统和工具系统
- 我能复述一条完整的 MCP 接入主线

## 明日预告

第 8 天建议进入插件和 skills 体系，开始理解 Claude Code 如何让本地可扩展能力进入命令和模型能力视图。

明天的重点文件预计是：

1. [restored-src/src/skills](../restored-src/src/skills)
2. [restored-src/src/plugins](../restored-src/src/plugins)
3. [restored-src/src/skills/loadSkillsDir.js](../restored-src/src/skills/loadSkillsDir.js)
4. [restored-src/src/utils/plugins/loadPluginCommands.js](../restored-src/src/utils/plugins/loadPluginCommands.js)

明天的目标是把“外部 server 能力”继续扩展到“本地插件和 skills 如何接入 Claude Code 能力体系”。
