# 第 4 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 4`

## 今日目标

今天开始拆开第 3 天看到的主流程装配层，重点研究两个核心注册表：

1. `restored-src/src/commands.ts`
2. `restored-src/src/tools.ts`

完成今天后，你应该能够明确回答：

1. Claude Code 的命令是如何被注册、筛选和组合的
2. Claude Code 的工具是如何被注册、筛选和组合的
3. feature flag、权限、模式切换、插件、skills、MCP 分别会影响哪一层
4. 为什么 `commands.ts` 和 `tools.ts` 都更像“注册与装配中心”而不是单个功能模块
5. 命令系统和工具系统在主流程中分别扮演什么角色

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `commands.ts`
2. 一句话定义 `tools.ts`
3. 一张“命令系统装配图”
4. 一张“工具系统装配图”
5. 一份“命令和工具的边界”说明

## 核心结论

先记住这几句话：

1. `commands.ts` 负责把内置命令、动态技能、插件命令、工作流命令等统一装配成“可见命令集”。
2. `tools.ts` 负责把内置工具、模式特例、权限过滤、MCP 工具组合成“可用工具池”。
3. 命令面向的是“用户或模型可触发的入口动作”，工具面向的是“模型实际可调用的执行能力”。
4. 两个文件都大量使用 feature flag、懒加载和过滤逻辑，目的是在同一套代码里支撑不同产品形态。
5. 第 4 天的重点不是背命令名和工具名，而是建立“注册 -> 过滤 -> 装配 -> 暴露”的心智模型。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/commands.ts](../restored-src/src/commands.ts)
2. [restored-src/src/tools.ts](../restored-src/src/tools.ts)
3. [restored-src/src/types/command.js](../restored-src/src/types/command.js)
4. [restored-src/src/Tool.ts](../restored-src/src/Tool.ts)

辅助观察目录：

1. [restored-src/src/commands](../restored-src/src/commands)
2. [restored-src/src/tools](../restored-src/src/tools)
3. [restored-src/src/skills](../restored-src/src/skills)
4. [restored-src/src/plugins](../restored-src/src/plugins)

## 学习步骤

### 步骤 1：先区分“命令系统”和“工具系统”

先不要急着看细节，先建立边界。

你要先回答：

1. 用户输入 `/review` 这种 slash command，属于命令还是工具
2. 模型调用 `BashTool`、`FileReadTool`，属于命令还是工具
3. 为什么主流程里既需要命令系统，也需要工具系统

你要提炼出的判断是：

命令是入口层，工具是执行层。

### 步骤 2：读 `commands.ts` 的注册表结构

先读 [restored-src/src/commands.ts](../restored-src/src/commands.ts) 的前半段。

重点看：

1. 大量命令 import
2. feature flag 控制的条件命令
3. `INTERNAL_ONLY_COMMANDS`
4. `COMMANDS = memoize(...)`

你要理解的是：

`commands.ts` 并不是在“实现所有命令”，而是在“定义默认命令集合”。

### 步骤 3：搞清 `getCommands(cwd)` 在做什么

继续读 `commands.ts` 的后半段，重点看：

1. `getSkills(...)`
2. `loadAllCommands(...)`
3. `getCommands(cwd)`
4. dynamic skills 的插入
5. cache 清理函数

你要回答：

1. 内置命令、插件命令、工作流命令、skills 是怎么汇总的
2. 为什么这里会用 memoize
3. 为什么 availability 和 isEnabled 要每次重新判断
4. 为什么 dynamic skills 不是一开始就固定写死

今天要建立的核心认知：

`getCommands(cwd)` 的职责不是“返回一份常量表”，而是“按当前环境实时计算命令可见集”。

### 步骤 4：理解技能相关命令视图

重点看：

1. `getMcpSkillCommands(...)`
2. `getSkillToolCommands(...)`
3. `getSlashCommandToolSkills(...)`

你要理解：

命令系统里并不是所有命令都同等对待，有一部分 prompt 型命令会被转化成模型侧可理解的技能集合。

你要回答：

1. 为什么 skill 相关函数要从完整命令集里再筛一遍
2. “可显示给用户的命令” 和 “可暴露给模型的 skill” 为什么不是完全同一批

### 步骤 5：看命令系统里的远程和桥接限制

重点看：

1. `REMOTE_SAFE_COMMANDS`
2. `BRIDGE_SAFE_COMMANDS`
3. `isBridgeSafeCommand(...)`
4. `filterCommandsForRemoteMode(...)`

你要理解的是：

命令系统不只是注册，还承担“在不同运行环境下裁剪能力”的职责。

### 步骤 6：转向 `tools.ts`，看工具总表

开始读 [restored-src/src/tools.ts](../restored-src/src/tools.ts)。

重点看：

1. 顶部的大量工具 import
2. 条件启用的工具
3. `TOOL_PRESETS`
4. `getAllBaseTools()`

你要先建立的认知是：

`getAllBaseTools()` 是工具系统的总清单来源，里面混合了基础工具、条件工具、模式工具和实验性工具。

### 步骤 7：理解 `getTools(permissionContext)` 的过滤逻辑

这是今天最关键的部分之一。

重点看：

1. simple mode
2. REPL mode
3. `filterToolsByDenyRules(...)`
4. `isEnabled()` 过滤
5. special tools 的排除

你要回答：

1. 为什么工具不是一股脑全部暴露给模型
2. 为什么 simple mode 下只保留少量基础工具
3. 为什么 REPL 开启后要隐藏某些 primitive tools
4. deny rules 为什么会在“工具展示前”就生效

### 步骤 8：理解 built-in tools 和 MCP tools 的组合

继续看：

1. `assembleToolPool(...)`
2. `getMergedTools(...)`

你要理解：

Claude Code 并不是只有内置工具。真正运行时，工具池通常是“内置工具 + MCP 工具”的组合。

你要回答：

1. 为什么 `assembleToolPool(...)` 要先处理 built-in tools 再处理 MCP tools
2. 为什么还要去重和排序
3. 为什么 `getMergedTools(...)` 和 `assembleToolPool(...)` 不完全等价

### 步骤 9：最后回到主流程视角

回想第 3 天的 `main.tsx`，把今天学到的内容接回去：

1. `getCommands(...)` 会在主流程哪个阶段被调用
2. `getTools(...)` 会在主流程哪个阶段被调用
3. 为什么 permissions 会影响 tools，而 auth/provider/feature flags 会影响 commands
4. 为什么这两个注册表必须在主流程里被单独抽出来

你要形成的整体图景是：

`main.tsx` 负责装配时序，`commands.ts` 和 `tools.ts` 负责可见能力的构建。

## 今日建议时间安排

1. `15 分钟`：先建立命令和工具的边界
2. `25 分钟`：阅读 `commands.ts` 的注册和筛选逻辑
3. `20 分钟`：整理 skills、remote、bridge 相关命令裁剪
4. `25 分钟`：阅读 `tools.ts` 的总表和过滤逻辑
5. `15 分钟`：理解 built-in tools 和 MCP tools 的组合
6. `20 分钟`：完成笔记与作业

总时长建议：`120 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 4 学习笔记

## 1. 我对 commands.ts 的一句话定义

## 2. 我对 tools.ts 的一句话定义

## 3. 命令系统的装配来源
- builtin:
- plugin:
- workflow:
- skills:
- dynamic skills:

## 4. 工具系统的装配来源
- builtin tools:
- conditional tools:
- mode-specific tools:
- MCP tools:

## 5. 命令系统如何过滤
1.
2.
3.

## 6. 工具系统如何过滤
1.
2.
3.

## 7. 命令和工具的边界

## 8. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：定义两个注册表

请分别用一句话定义：

1. `commands.ts`
2. `tools.ts`

要求：

1. 两句话不能写成同一个意思
2. 必须体现“注册 / 过滤 / 装配”中的至少一个关键词
3. 必须体现命令和工具的边界不同

### 作业 2：画出命令系统来源图

请按你的理解写出命令系统的主要来源。

要求：

1. 至少包含 builtin commands
2. 至少包含 plugin commands
3. 至少包含 bundled skills 或 skill dir commands
4. 至少包含 dynamic skills
5. 必须写出最终由哪个函数汇总

### 作业 3：解释 `getCommands(cwd)`

请回答：为什么 `getCommands(cwd)` 不是简单返回一个常量数组？

要求：

1. 至少提到一次 cwd
2. 至少提到一次 availability 或 isEnabled
3. 至少提到一次 plugin / skill / dynamic skill
4. 至少提到一次缓存但不是完全静态

### 作业 4：解释 `getTools(permissionContext)`

请回答：为什么 `getTools(permissionContext)` 一定要接收 permission context？

要求：

1. 至少提到 deny rules
2. 至少提到 simple mode 或 REPL mode
3. 至少提到“不是所有工具都应该暴露”

### 作业 5：比较命令和工具

请用表格或分点方式比较：

1. 命令的使用者是谁
2. 工具的使用者是谁
3. 命令的主要来源是什么
4. 工具的主要来源是什么
5. 两者在主流程里分别在哪里被装配

要求：

1. 不允许只写一句笼统的话
2. 必须体现“入口层 vs 执行层”

### 作业 6：解释 MCP 在两层里的位置

请分别回答：

1. MCP 为什么会影响命令系统
2. MCP 为什么会影响工具系统

要求：

1. 至少提到 `getMcpSkillCommands(...)` 或 MCP skill 相关过滤
2. 至少提到 `assembleToolPool(...)`
3. 必须说明命令和工具受 MCP 影响的方式不同

### 作业 7：写一段 Day 4 总结

写一个 `220-380` 字的小总结，要求包含：

1. 你今天如何理解命令系统
2. 你今天如何理解工具系统
3. 你认为哪一层更直接影响模型能力暴露，为什么

## 自检清单

今天结束前，逐项确认：

- 我知道 `commands.ts` 不是命令实现集合，而是命令注册与装配中心
- 我知道 `tools.ts` 不是工具实现集合，而是工具注册与装配中心
- 我知道 `getCommands(cwd)` 为什么需要动态计算
- 我知道 `getTools(permissionContext)` 为什么必须依赖权限上下文
- 我知道 remote 和 bridge 会对命令可见性造成限制
- 我知道 simple mode 和 REPL mode 会对工具暴露造成限制
- 我知道 MCP 会同时影响命令层和工具层，但方式不同
- 我能解释命令和工具在 Claude Code 里的边界

## 明日预告

第 5 天建议继续深入权限、模式和执行边界，重点进入工具权限与执行约束相关逻辑。

明天的重点文件预计是：

1. [restored-src/src/utils/permissions](../restored-src/src/utils/permissions)
2. [restored-src/src/constants/tools.ts](../restored-src/src/constants/tools.ts)
3. [restored-src/src/Tool.ts](../restored-src/src/Tool.ts)

明天的目标是把“工具如何装配”进一步推进到“工具为什么能调用、为什么会被拒绝、为什么会被限制”。
