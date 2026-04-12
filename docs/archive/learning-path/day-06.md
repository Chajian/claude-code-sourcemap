# 第 6 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 6`

## 今日目标

今天把第 5 天的抽象权限模型继续落到具体工具场景，重点研究文件系统权限检查、shell 规则匹配，以及 Bash / FileRead / FileEdit 这几类高频工具如何接入安全策略。

核心学习对象是：

1. `restored-src/src/utils/permissions/filesystem.ts`
2. `restored-src/src/utils/permissions/shellRuleMatching.ts`
3. `restored-src/src/tools/BashTool/`
4. `restored-src/src/tools/FileReadTool/`
5. `restored-src/src/tools/FileEditTool/`

完成今天后，你应该能够明确回答：

1. 文件读写为什么不只是“看路径是否在项目里”
2. 哪些路径会被视为敏感、危险或需要额外确认
3. shell 权限规则中的 exact / prefix / wildcard 分别是什么意思
4. BashTool 为什么比普通文件工具更复杂
5. FileReadTool 和 FileEditTool 为什么会共享一部分权限基础设施但判断顺序不同

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `filesystem.ts`
2. 一句话定义 `shellRuleMatching.ts`
3. 一张“文件权限检查主线”图
4. 一张“shell 规则匹配主线”图
5. 一份“Bash / FileRead / FileEdit 三者差异”说明

## 核心结论

先记住这几句话：

1. `filesystem.ts` 不是简单路径工具库，而是文件权限安全策略的重要落地层。
2. 文件路径安全不只涉及工作目录，还涉及敏感文件、危险目录、UNC 路径、可疑 Windows 路径模式、symlink 解析等问题。
3. `shellRuleMatching.ts` 把 shell 权限规则从字符串语法变成可计算的 exact / prefix / wildcard 匹配逻辑。
4. BashTool 的权限判断比文件工具更复杂，因为它既涉及命令语义，也涉及路径、重定向、子命令和 read-only 识别。
5. FileReadTool 和 FileEditTool 看似简单，但它们背后也有一整套规则优先级和安全兜底逻辑。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/utils/permissions/filesystem.ts](../restored-src/src/utils/permissions/filesystem.ts)
2. [restored-src/src/utils/permissions/shellRuleMatching.ts](../restored-src/src/utils/permissions/shellRuleMatching.ts)
3. [restored-src/src/tools/BashTool/BashTool.tsx](../restored-src/src/tools/BashTool/BashTool.tsx)
4. [restored-src/src/tools/BashTool/bashPermissions.ts](../restored-src/src/tools/BashTool/bashPermissions.ts)
5. [restored-src/src/tools/BashTool/readOnlyValidation.ts](../restored-src/src/tools/BashTool/readOnlyValidation.ts)
6. [restored-src/src/tools/FileReadTool/FileReadTool.ts](../restored-src/src/tools/FileReadTool/FileReadTool.ts)
7. [restored-src/src/tools/FileEditTool/FileEditTool.ts](../restored-src/src/tools/FileEditTool/FileEditTool.ts)

辅助观察目录：

1. [restored-src/src/tools/BashTool](../restored-src/src/tools/BashTool)
2. [restored-src/src/tools/FileReadTool](../restored-src/src/tools/FileReadTool)
3. [restored-src/src/tools/FileEditTool](../restored-src/src/tools/FileEditTool)

## 学习步骤

### 步骤 1：先把 `filesystem.ts` 看成“文件权限安全层”

先读 [restored-src/src/utils/permissions/filesystem.ts](../restored-src/src/utils/permissions/filesystem.ts)，不要把它仅仅理解为 path helper。

你要先回答：

1. 为什么文件权限检查要单独抽成一个模块
2. 为什么这里会同时处理路径规范化、路径比较、安全文件名单、规则匹配
3. 为什么路径安全本身就是权限系统的一部分

你要提炼出的判断是：

`filesystem.ts` 是“文件相关权限与路径安全策略”的集中落地层。

### 步骤 2：重点观察危险路径与敏感文件

继续读 `filesystem.ts` 时，重点关注：

1. `DANGEROUS_FILES`
2. `DANGEROUS_DIRECTORIES`
3. `normalizeCaseForComparison(...)`
4. `isClaudeSettingsPath(...)`
5. `isClaudeConfigFilePath(...)`
6. `checkPathSafetyForAutoEdit(...)`

你要理解：

Claude Code 不是单纯防止“越出工作目录”，还要防止模型无意中改到能影响执行环境、配置加载或数据泄露的关键文件。

今天要特别记住：

1. `.git`、`.claude`、shell 配置文件等都可能被提升为敏感路径
2. 大小写混淆、UNC 路径、可疑 Windows 路径格式都可能是绕过安全检查的入口

### 步骤 3：梳理文件权限检查顺序

在 `filesystem.ts` 里不要试图记住所有函数名，重点抓顺序。

你要整理：

1. 哪些检查是 defense-in-depth
2. 哪些检查属于 read-specific deny / ask rules
3. 哪些检查属于 edit access implies read access 之类的继承逻辑
4. 哪些检查属于 working directory 限制
5. 什么时候默认 ask，什么时候直接 deny

你要建立的核心认知：

文件权限检查不是“路径在不在 cwd”这么简单，而是“按风险优先级逐层排查”。

### 步骤 4：理解 shell 权限规则语法

阅读 [restored-src/src/utils/permissions/shellRuleMatching.ts](../restored-src/src/utils/permissions/shellRuleMatching.ts)。

重点看：

1. `permissionRuleExtractPrefix(...)`
2. `hasWildcards(...)`
3. `matchWildcardPattern(...)`
4. `parsePermissionRule(...)`
5. `suggestionForExactCommand(...)`
6. `suggestionForPrefix(...)`

你要理解：

shell 权限规则不是单一字符串比较，而是至少有三类匹配语义：

1. exact
2. prefix
3. wildcard

这一步的重点不是背实现，而是掌握规则语法和匹配语义。

### 步骤 5：看 BashTool 为什么复杂

开始读 [restored-src/src/tools/BashTool/bashPermissions.ts](../restored-src/src/tools/BashTool/bashPermissions.ts) 和 [restored-src/src/tools/BashTool/readOnlyValidation.ts](../restored-src/src/tools/BashTool/readOnlyValidation.ts)。

重点思考：

1. 为什么 BashTool 需要单独的 `bashPermissions.ts`
2. 为什么要判断 read-only command
3. 为什么路径约束、deny rules、ask rules、子命令结果要混合考虑
4. 为什么 compound commands、`cd`、`git`、重定向、UNC 路径会让问题变复杂

你要提炼出的判断是：

BashTool 不是一个“能运行 shell 就行”的工具，它本质上是权限系统里风险最高、判断最细的一类工具。

### 步骤 6：看 read-only 识别的价值

重点读 `readOnlyValidation.ts` 的思路，不必一次吃掉整个 allowlist。

你要回答：

1. 为什么系统要努力识别 read-only shell 命令
2. 为什么只依赖正则或黑名单不够
3. 为什么这里会有大段 allowlist 设计
4. 为什么一些看起来 harmless 的 flag 也会被严格限制

你要理解：

read-only 识别的本质是尽可能自动放行低风险命令，同时避免把可执行、可写入、可联网的命令误判成安全操作。

### 步骤 7：对比 FileReadTool 和 FileEditTool

开始读：

1. [restored-src/src/tools/FileReadTool/FileReadTool.ts](../restored-src/src/tools/FileReadTool/FileReadTool.ts)
2. [restored-src/src/tools/FileEditTool/FileEditTool.ts](../restored-src/src/tools/FileEditTool/FileEditTool.ts)

重点看：

1. 它们如何接入文件权限检查
2. 为什么读和写的风险等级不一样
3. 为什么 edit 需要额外的 auto-edit safety check
4. 为什么 read 也不是总能直接 allow

你要形成的比较是：

FileReadTool 更偏“读取范围与敏感路径控制”，FileEditTool 更偏“写入风险与修改对象保护”。

### 步骤 8：把三类工具接回权限主线

回顾第 5 天的抽象权限系统，把今天的三类场景接回去：

1. FileReadTool 主要如何使用 `filesystem.ts`
2. FileEditTool 主要如何使用 `filesystem.ts`
3. BashTool 如何同时使用 shell 规则、路径约束、工具特定判断

你要形成的整体图景是：

抽象权限模型并不是悬空存在的，它会被不同工具以不同方式实例化。

### 步骤 9：总结“高风险工具为什么难”

今天最后请主动总结：

1. 文件编辑难在哪里
2. shell 执行难在哪里
3. 为什么 BashTool 的权限逻辑会比 FileReadTool 长得多
4. 为什么这类逻辑一定要高度防御式设计

## 今日建议时间安排

1. `25 分钟`：阅读 `filesystem.ts`，抓危险路径和文件权限主线
2. `20 分钟`：阅读 `shellRuleMatching.ts`，掌握规则语义
3. `30 分钟`：阅读 `bashPermissions.ts` 和 `readOnlyValidation.ts`
4. `20 分钟`：对比 `FileReadTool.ts` 和 `FileEditTool.ts`
5. `15 分钟`：把三类工具接回第 5 天权限模型
6. `20 分钟`：完成笔记与作业

总时长建议：`130 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 6 学习笔记

## 1. 我对 filesystem.ts 的一句话定义

## 2. 我对 shellRuleMatching.ts 的一句话定义

## 3. 文件权限高风险点
- dangerous files:
- dangerous directories:
- UNC:
- suspicious Windows patterns:
- config files:

## 4. shell 规则语义
- exact:
- prefix:
- wildcard:

## 5. BashTool 为什么复杂
1.
2.
3.

## 6. FileReadTool 和 FileEditTool 的差异
1.
2.
3.

## 7. 我今天最重要的理解

## 8. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：定义两个基础模块

请分别用一句话定义：

1. `filesystem.ts`
2. `shellRuleMatching.ts`

要求：

1. 两句话不能表达同一层含义
2. 第一条必须体现“文件权限”或“路径安全”
3. 第二条必须体现“shell 规则匹配”

### 作业 2：列出文件高风险场景

请按你的理解列出至少 5 个会让文件操作进入 ask 或 deny 的高风险场景。

要求：

1. 至少包含敏感文件或危险目录
2. 至少包含 UNC 路径
3. 至少包含可疑 Windows 路径模式
4. 至少包含 `.claude` 或配置文件相关场景

### 作业 3：解释 shell 三类匹配

请分别解释 shell 规则里的：

1. exact
2. prefix
3. wildcard

要求：

1. 每一类都要给一个例子
2. 必须说明它们的匹配粒度差异

### 作业 4：解释 BashTool 为什么难

请回答：为什么 BashTool 的权限逻辑会比 FileReadTool 和 FileEditTool 更复杂？

要求：

1. 至少提到子命令或 compound command
2. 至少提到路径问题
3. 至少提到 read-only 识别
4. 至少提到规则匹配或建议生成

### 作业 5：比较 FileReadTool 和 FileEditTool

请从下面 4 个维度比较：

1. 风险类型
2. 默认倾向
3. 关键安全检查
4. 为什么不能共用完全相同的权限顺序

要求：

1. 不能只写“一个读一个写”
2. 必须体现两者共享基础设施但风险模型不同

### 作业 6：还原一条权限判断链

任选下面一类场景，写出 5 到 8 步的权限判断链：

1. 读取项目外文件
2. 编辑 `.claude/settings.json`
3. 执行带重定向的 shell 命令
4. 执行包含 UNC 路径的读取命令

要求：

1. 必须说明每一步在判断什么
2. 必须说明最终为什么是 allow / ask / deny

### 作业 7：写一段 Day 6 总结

写一个 `220-400` 字的小总结，要求包含：

1. 你今天如何理解文件系统安全检查
2. 你今天如何理解 shell 命令权限匹配
3. 你认为哪一类工具最难做安全设计，为什么

## 自检清单

今天结束前，逐项确认：

- 我知道 `filesystem.ts` 为什么属于权限系统而不只是工具函数
- 我知道敏感文件、危险目录、UNC 路径、Windows 特殊路径为什么要特别处理
- 我知道 shell 规则里的 exact / prefix / wildcard 分别是什么
- 我知道 BashTool 为什么比文件工具复杂很多
- 我知道 read-only shell 命令识别的价值是什么
- 我知道 FileReadTool 和 FileEditTool 共享基础设施但决策重点不同
- 我知道抽象权限模型如何落到真实工具行为上
- 我能复述至少一条具体工具的权限判断链

## 明日预告

第 7 天建议进入 MCP 和外部能力接入，开始理解 Claude Code 如何把本地内置能力扩展成外部服务器提供的能力。

明天的重点文件预计是：

1. [restored-src/src/services/mcp](../restored-src/src/services/mcp)
2. [restored-src/src/tools/ListMcpResourcesTool](../restored-src/src/tools/ListMcpResourcesTool)
3. [restored-src/src/tools/ReadMcpResourceTool](../restored-src/src/tools/ReadMcpResourceTool)
4. [restored-src/src/commands/mcp](../restored-src/src/commands/mcp)

明天的目标是把本地命令和工具系统继续推进到“外部能力如何被注册、连接、过滤和暴露”。
