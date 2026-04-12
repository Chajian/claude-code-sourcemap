# 第 8 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 8`

## 今日目标

今天进入 plugins 和 skills 体系，重点理解 Claude Code 如何把“本地可扩展能力”接入现有的命令系统和模型能力视图。

核心学习对象是：

1. `restored-src/src/skills/`
2. `restored-src/src/plugins/`
3. `restored-src/src/skills/loadSkillsDir.ts`
4. `restored-src/src/skills/bundledSkills.ts`
5. `restored-src/src/plugins/builtinPlugins.ts`
6. `restored-src/src/utils/plugins/loadPluginCommands.ts`

完成今天后，你应该能够明确回答：

1. skill 是如何从磁盘或内置注册表加载进来的
2. plugin 是如何把命令和 skills 暴露给系统的
3. bundled skills、skill dir skills、dynamic skills、plugin skills 的差异是什么
4. built-in plugin 和 bundled skill 为什么不是同一个概念
5. 本地扩展能力最终是如何汇入统一命令集和模型 skill 视图的

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `loadSkillsDir.ts`
2. 一句话定义 `loadPluginCommands.ts`
3. 一张“skills 来源分类图”
4. 一张“plugins 接入命令层主线图”
5. 一份“bundled skill vs built-in plugin vs plugin skill”对比说明

## 核心结论

先记住这几句话：

1. `skills/` 负责把 prompt 型能力整理成系统可理解的 skill 命令。
2. `plugins/` 负责把用户可安装或内置可切换的扩展能力接入命令、skills、hooks、MCP 等体系。
3. bundled skill 是编进 CLI 的技能能力，built-in plugin 是用户可开关的内置插件，这两者边界不同。
4. dynamic skills 说明 skill 并不一定在启动时就全部固定好，有一部分会在运行过程中被发现并激活。
5. 第 8 天的重点不是“会写一个 skill”，而是搞懂本地扩展能力如何被系统发现、加载、过滤并暴露。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/skills/loadSkillsDir.ts](../restored-src/src/skills/loadSkillsDir.ts)
2. [restored-src/src/skills/bundledSkills.ts](../restored-src/src/skills/bundledSkills.ts)
3. [restored-src/src/skills/bundled/index.ts](../restored-src/src/skills/bundled/index.ts)
4. [restored-src/src/plugins/builtinPlugins.ts](../restored-src/src/plugins/builtinPlugins.ts)
5. [restored-src/src/utils/plugins/loadPluginCommands.ts](../restored-src/src/utils/plugins/loadPluginCommands.ts)
6. [restored-src/src/commands.ts](../restored-src/src/commands.ts)

辅助观察目录：

1. [restored-src/src/skills](../restored-src/src/skills)
2. [restored-src/src/plugins](../restored-src/src/plugins)
3. [restored-src/src/utils/plugins](../restored-src/src/utils/plugins)

## 学习步骤

### 步骤 1：先区分 skill 和 plugin

先不要急着追实现，先把概念边界立住。

你要先回答：

1. skill 更接近“能力提示 / prompt 入口”还是“完整扩展包”
2. plugin 更接近“单个能力”还是“一组可安装组件”
3. 为什么两者最后都会影响命令系统，但本体又不是同一回事

你要提炼出的判断是：

skill 更偏模型可调用能力单元，plugin 更偏可安装、可管理、可组合的扩展载体。

### 步骤 2：读 `loadSkillsDir.ts`，看磁盘技能加载

阅读 [restored-src/src/skills/loadSkillsDir.ts](../restored-src/src/skills/loadSkillsDir.ts)。

重点关注：

1. frontmatter 解析
2. markdown skill 到 Command 的转换
3. `getSkillDirCommands(...)`
4. `getDynamicSkills()`
5. `clearSkillCaches()`

你要理解：

skill 并不是固定硬编码在代码里的，很多 skill 是通过目录扫描和 markdown/frontmatter 解析构造出来的。

### 步骤 3：理解一个 skill 命令是如何被构造出来的

继续围绕 `loadSkillsDir.ts` 观察：

1. `parseSkillFrontmatterFields(...)`
2. `createSkillCommand(...)`
3. `allowedTools`
4. `whenToUse`
5. `disableModelInvocation`
6. `userInvocable`
7. `hooks`
8. `context`

你要回答：

1. 一个 markdown skill 为什么最终会长成一个 `type: 'prompt'` 的 Command
2. 为什么 skill 既要给用户可见，又要给模型可见
3. 为什么 frontmatter 会同时影响展示、模型调用和执行上下文

### 步骤 4：理解 dynamic skills

重点看 `getDynamicSkills()` 及其相关逻辑。

你要理解：

有一部分 skill 并不是启动时全量加载，而是在文件操作或特定触发器下被发现后加入系统。

你要回答：

1. 为什么 Claude Code 要支持 dynamic skills
2. 为什么命令系统里要单独处理 dynamic skills 的插入和去重

### 步骤 5：读 `bundledSkills.ts`，看内置 skill 注册表

阅读 [restored-src/src/skills/bundledSkills.ts](../restored-src/src/skills/bundledSkills.ts)。

重点看：

1. `registerBundledSkill(...)`
2. `getBundledSkills()`
3. skill files 的懒提取
4. `skillRoot`
5. `prependBaseDir(...)`

你要理解：

bundled skills 是“编进 CLI 的能力”，但它们仍然会被包装成 Command，并遵守 skill 体系的公共约定。

### 步骤 6：区分 bundled skills 和 built-in plugins

阅读 [restored-src/src/plugins/builtinPlugins.ts](../restored-src/src/plugins/builtinPlugins.ts)。

重点关注：

1. `registerBuiltinPlugin(...)`
2. `getBuiltinPlugins()`
3. `getBuiltinPluginSkillCommands()`
4. enabled / disabled 状态
5. `defaultEnabled`

你要回答：

1. 为什么 built-in plugin 不是 bundled skill
2. 为什么 built-in plugin 会出现在 `/plugin` UI 中
3. 为什么 built-in plugin 可以提供 skills、hooks、MCP servers 等多种组件

今天要建立的认知是：

bundled skill 是“直接内置的 skill 能力”，built-in plugin 是“可开关的内置扩展包”。

### 步骤 7：读 `loadPluginCommands.ts`，看插件命令和插件 skill 如何进入系统

阅读 [restored-src/src/utils/plugins/loadPluginCommands.ts](../restored-src/src/utils/plugins/loadPluginCommands.ts)。

重点关注：

1. `getPluginCommands()`
2. `getPluginSkills()`
3. `clearPluginCommandCache()`
4. `clearPluginSkillsCache()`

你要理解：

plugin 不只是“装完就完”，系统还要把 plugin 提供的命令和 skill 分开处理，并纳入命令装配缓存。

### 步骤 8：回到 `commands.ts`，看统一汇总

回看 [restored-src/src/commands.ts](../restored-src/src/commands.ts)。

重点看：

1. `getSkills(...)`
2. `loadAllCommands(...)`
3. `getCommands(cwd)`
4. `getSkillToolCommands(...)`
5. `getSlashCommandToolSkills(...)`

你要回答：

1. skill dir commands、plugin skills、bundled skills、builtin plugin skills 为什么会在这里汇总
2. 为什么最终所有这些能力都会变成 Command 视图的一部分
3. 为什么“给用户看”和“给模型看”的过滤条件不完全一致

### 步骤 9：整理四类 skill 的差异

今天最后请把下面几类能力并列比较：

1. skill dir skills
2. bundled skills
3. builtin plugin skills
4. plugin skills
5. dynamic skills

你要重点比较：

1. 来源
2. 是否用户可开关
3. 是否启动时固定
4. 是否需要磁盘扫描
5. 是否主要面向模型

你要形成的整体图景是：

Claude Code 的本地扩展能力并不是一条线，而是多条来源、同一落点。

## 今日建议时间安排

1. `15 分钟`：先区分 skill 和 plugin 的概念边界
2. `30 分钟`：阅读 `loadSkillsDir.ts`
3. `20 分钟`：阅读 `bundledSkills.ts` 和 `skills/bundled/index.ts`
4. `20 分钟`：阅读 `builtinPlugins.ts`
5. `25 分钟`：阅读 `loadPluginCommands.ts`
6. `20 分钟`：回到 `commands.ts` 串起统一汇总逻辑
7. `20 分钟`：完成笔记与作业

总时长建议：`150 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 8 学习笔记

## 1. 我对 skill 的一句话理解

## 2. 我对 plugin 的一句话理解

## 3. skills 的来源
- skill dir:
- bundled:
- builtin plugin:
- plugin:
- dynamic:

## 4. plugins 的来源和能力
- builtin plugin:
- external plugin:
- skills:
- hooks:
- MCP:

## 5. 为什么最终都汇入 commands.ts

## 6. bundled skill 和 built-in plugin 的区别

## 7. 我今天最重要的理解

## 8. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：定义 skill 和 plugin

请分别用一句话定义：

1. skill
2. plugin

要求：

1. 两句话不能写成同一个意思
2. 必须体现 skill 更偏能力单元，plugin 更偏扩展载体

### 作业 2：列出 skill 的 5 种来源

请按你的理解列出：

1. skill dir skills
2. bundled skills
3. builtin plugin skills
4. plugin skills
5. dynamic skills

要求：

1. 每一类都要写一句来源说明
2. 至少说明一类是否可用户开关

### 作业 3：解释 `loadSkillsDir.ts`

请回答：为什么 `loadSkillsDir.ts` 是 skill 体系的关键入口？

要求：

1. 至少提到 markdown/frontmatter
2. 至少提到 skill command 构造
3. 至少提到动态发现或缓存

### 作业 4：解释 bundled skill 和 built-in plugin 的区别

请分别说明它们在下面几个维度的不同：

1. 是否可开关
2. 是否属于插件 UI 管理对象
3. 是否只提供单一 skill 能力
4. 是否会携带 hooks 或 MCP server

要求：

1. 不能只写“一个是 skill 一个是 plugin”
2. 必须体现管理方式不同

### 作业 5：解释 `loadPluginCommands.ts`

请回答：

1. 为什么插件命令和插件 skill 要分开加载
2. 为什么它们都要有缓存和清缓存逻辑

要求：

1. 至少提到 `getPluginCommands()`
2. 至少提到 `getPluginSkills()`
3. 至少提到刷新或重新装配场景

### 作业 6：解释为什么最终都要回到 `commands.ts`

请回答：为什么 skills、plugin skills、bundled skills 最终都要汇总到统一命令视图里？

要求：

1. 至少提到 `getCommands(cwd)`
2. 至少提到 `getSkillToolCommands(...)` 或 `getSlashCommandToolSkills(...)`
3. 必须说明“用户可见视图”和“模型可见视图”之间的关系

### 作业 7：写一段 Day 8 总结

写一个 `220-400` 字的小总结，要求包含：

1. 你今天如何理解 skills 体系
2. 你今天如何理解 plugins 体系
3. 你认为 Claude Code 为什么要同时保留这两套扩展机制

## 自检清单

今天结束前，逐项确认：

- 我知道 skill 和 plugin 不是同一个概念
- 我知道磁盘 skill、bundled skill、plugin skill 的来源不同
- 我知道 dynamic skills 为什么存在
- 我知道 built-in plugin 为什么不是 bundled skill
- 我知道插件可以提供的不只是 skills
- 我知道 `loadPluginCommands.ts` 为什么要区分命令和 skills
- 我知道这些扩展能力最终为什么都要回到 `commands.ts`
- 我能解释本地扩展能力如何进入模型 skill 视图

## 明日预告

第 9 天建议进入状态管理和 REPL 交互主线，开始理解这些命令、工具、技能最终如何进入用户可交互的运行界面。

明天的重点文件预计是：

1. [restored-src/src/state](../restored-src/src/state)
2. [restored-src/src/replLauncher.tsx](../restored-src/src/replLauncher.tsx)
3. [restored-src/src/screens/REPL.tsx](../restored-src/src/screens/REPL.tsx)
4. [restored-src/src/context](../restored-src/src/context)

明天的目标是把“能力注册与装配”继续推进到“这些能力如何进入交互界面和会话状态机”。  
