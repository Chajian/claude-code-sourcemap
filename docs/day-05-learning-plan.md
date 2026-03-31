# 第 5 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 5`

## 今日目标

今天继续沿着第 4 天的工具系统往下走，重点研究权限模型、工具约束和执行边界。

核心学习对象是：

1. `restored-src/src/Tool.ts`
2. `restored-src/src/constants/tools.ts`
3. `restored-src/src/utils/permissions/`

完成今天后，你应该能够明确回答：

1. 工具为什么不是“存在就能调用”
2. `ToolPermissionContext` 到底承载了哪些权限信息
3. agent、async agent、coordinator mode、simple mode、REPL mode 为什么会有不同工具边界
4. deny / ask / allow 规则大致是怎么进入工具决策链路的
5. 权限系统是如何在“工具展示前”和“真正调用时”同时生效的

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义 `Tool.ts`
2. 一句话定义 `ToolPermissionContext`
3. 一张“工具权限决策链”示意图
4. 一份“模式和 agent 类型如何影响工具边界”的清单
5. 一组能验证你是否真正理解权限模型的自测题

## 核心结论

先记住这几句话：

1. `Tool.ts` 定义的是工具系统的公共契约，不是单个工具实现。
2. `ToolPermissionContext` 是工具权限判断的核心上下文，里面不仅有 mode，还有 allow / deny / ask 规则和额外工作目录等信息。
3. `constants/tools.ts` 负责表达“哪些工具在哪类 agent 或模式下根本不该开放”。
4. `utils/permissions/` 负责表达“某个工具在当前上下文下究竟是 allow、ask 还是 deny”。
5. Claude Code 的权限系统不是单点判断，而是“静态限制 + 运行时规则 + 模式约束 + 工具特定判断”的组合。

## 今日学习材料

按顺序阅读以下文件：

1. [restored-src/src/Tool.ts](../restored-src/src/Tool.ts)
2. [restored-src/src/constants/tools.ts](../restored-src/src/constants/tools.ts)
3. [restored-src/src/utils/permissions/PermissionMode.ts](../restored-src/src/utils/permissions/PermissionMode.ts)
4. [restored-src/src/utils/permissions/permissionSetup.ts](../restored-src/src/utils/permissions/permissionSetup.ts)
5. [restored-src/src/utils/permissions/permissions.ts](../restored-src/src/utils/permissions/permissions.ts)
6. [restored-src/src/utils/permissions/permissionsLoader.ts](../restored-src/src/utils/permissions/permissionsLoader.ts)

辅助观察目录：

1. [restored-src/src/utils/permissions](../restored-src/src/utils/permissions)
2. [restored-src/src/tools](../restored-src/src/tools)
3. [restored-src/src/constants](../restored-src/src/constants)

## 学习步骤

### 步骤 1：先把 `Tool.ts` 看成“工具协议层”

先读 [restored-src/src/Tool.ts](../restored-src/src/Tool.ts)，不要试图一次吸收所有类型定义。

你要先回答：

1. 为什么 `Tool.ts` 会定义那么多类型而不是具体工具逻辑
2. 为什么 `ToolUseContext` 里会塞入 commands、tools、mcpClients、messages、permission context 等信息
3. 为什么工具系统需要一套统一契约

你要提炼出的判断是：

`Tool.ts` 负责定义“工具在系统里该如何被描述、如何接收上下文、如何参与权限与执行流程”。

### 步骤 2：重点理解 `ToolPermissionContext`

在 [restored-src/src/Tool.ts](../restored-src/src/Tool.ts) 中重点看 `ToolPermissionContext`。

你要特别注意：

1. `mode`
2. `additionalWorkingDirectories`
3. `alwaysAllowRules`
4. `alwaysDenyRules`
5. `alwaysAskRules`
6. `isBypassPermissionsModeAvailable`
7. `isAutoModeAvailable`
8. `shouldAvoidPermissionPrompts`
9. `prePlanMode`

你要理解的是：

权限上下文不是单个开关，而是一组会影响工具可见性和运行行为的状态集合。

### 步骤 3：从 `constants/tools.ts` 看静态边界

阅读 [restored-src/src/constants/tools.ts](../restored-src/src/constants/tools.ts)。

重点看：

1. `ALL_AGENT_DISALLOWED_TOOLS`
2. `CUSTOM_AGENT_DISALLOWED_TOOLS`
3. `ASYNC_AGENT_ALLOWED_TOOLS`
4. `IN_PROCESS_TEAMMATE_ALLOWED_TOOLS`
5. `COORDINATOR_MODE_ALLOWED_TOOLS`

你要回答：

1. 为什么要在常量层面先限制一批工具
2. 为什么 async agents、in-process teammates、coordinator 的工具边界不一样
3. 为什么这里强调“递归”“主线程抽象”“单例虚拟终端冲突”等问题

今天要建立的认知是：

不是所有权限限制都来自用户规则，系统自身也会因为架构安全和执行模型而预先砍掉一批工具。

### 步骤 4：理解 permission mode

阅读 [restored-src/src/utils/permissions/PermissionMode.ts](../restored-src/src/utils/permissions/PermissionMode.ts)。

你要关注：

1. 权限模式有哪些
2. 内部 mode 和 external mode 的关系
3. mode 的 title、symbol、color 等 UI 表达
4. 为什么 mode 本身会影响工具行为

你要理解：

permission mode 不只是 UI 文案，它会真实改变后续权限决策路径。

### 步骤 5：看 `permissionSetup.ts` 如何构造权限上下文

阅读 [restored-src/src/utils/permissions/permissionSetup.ts](../restored-src/src/utils/permissions/permissionSetup.ts)。

重点关注：

1. 初始化 tool permission context
2. CLI 参数如何影响 permission mode
3. dangerous permissions 为什么要被移除或 strip
4. auto mode 和 bypass mode 为什么需要单独处理

你要回答：

1. 为什么权限上下文必须在主流程早期构造
2. 为什么一些过宽的权限规则会被主动剥离
3. 为什么“是否能显示权限弹窗”也属于权限上下文的一部分

### 步骤 6：看 `permissions.ts` 的真实决策流水线

这是今天最关键的部分。

阅读 [restored-src/src/utils/permissions/permissions.ts](../restored-src/src/utils/permissions/permissions.ts) 时，不要试图记住全部分支，只要把大框架抓住：

1. 先看 deny rules
2. 再看 ask rules 或工具特定权限检查
3. 再看 mode 是否允许
4. 再看整个工具是否允许
5. 再看 classifier、hook、auto mode 等额外逻辑
6. 最终得到 allow / ask / deny

你要理解的是：

权限判断不是“一次 if 判断”，而是一条多阶段决策流水线。

### 步骤 7：理解“展示前过滤”和“调用时判断”是两层机制

回想第 4 天的 `filterToolsByDenyRules(...)` 和今天的 `permissions.ts`。

你要回答：

1. 为什么 deny rules 会在工具展示前先过滤一层
2. 为什么真正运行工具时还要再走一次权限流水线
3. 这两层如果只保留一层，会有什么问题

你要形成的心智模型是：

前者解决“不要暴露不该看到的能力”，后者解决“真正执行时是否允许放行”。

### 步骤 8：补看规则加载与持久化

阅读 [restored-src/src/utils/permissions/permissionsLoader.ts](../restored-src/src/utils/permissions/permissionsLoader.ts)。

重点看：

1. permission rules 从哪里加载
2. managed settings 和 project settings 的区别
3. 为什么有 `allowManagedPermissionRulesOnly`
4. 为什么规则不只是读取，还要支持删除和新增

你要理解：

权限系统不是纯内存逻辑，它还和项目配置、托管策略、持久化设置有关。

### 步骤 9：最后回到工具调用主线

今天最后回头看第 4 天的 `getTools(permissionContext)` 和第 3 天的主流程装配。

你要把这条主线串起来：

1. 主流程先构造 permission context
2. 工具注册表按 permission context 裁剪可见工具
3. 用户或模型发起调用
4. 真实调用前再经过权限决策流水线
5. 最终才真正执行工具

你要形成的整体图景是：

权限系统并不是工具系统旁边的附属物，而是工具系统的核心约束层。

## 今日建议时间安排

1. `20 分钟`：阅读 `Tool.ts`，重点看工具契约和权限上下文
2. `20 分钟`：阅读 `constants/tools.ts`，整理静态工具边界
3. `20 分钟`：阅读 `PermissionMode.ts` 和 `permissionSetup.ts`
4. `30 分钟`：阅读 `permissions.ts`，抓住决策主线
5. `15 分钟`：阅读 `permissionsLoader.ts`，理解规则来源
6. `20 分钟`：完成笔记与作业

总时长建议：`125 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 5 学习笔记

## 1. 我对 Tool.ts 的一句话定义

## 2. ToolPermissionContext 包含什么
- mode:
- allow rules:
- deny rules:
- ask rules:
- extra dirs:
- prompt behavior:

## 3. 系统级静态工具边界
- async agents:
- custom agents:
- coordinator mode:
- in-process teammates:

## 4. 权限决策主线
1.
2.
3.
4.
5.

## 5. 展示前过滤和调用时判断的区别

## 6. 我今天最重要的理解

## 7. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：定义权限上下文

请用你自己的话解释什么是 `ToolPermissionContext`。

要求：

1. 不超过 80 字
2. 必须包含 mode
3. 必须包含 allow / deny / ask 中至少两个
4. 必须说明它服务于工具系统

### 作业 2：解释两类限制

请分别解释：

1. 为什么 `constants/tools.ts` 要定义静态工具限制
2. 为什么 `permissions.ts` 还需要做运行时权限判断

要求：

1. 两部分不能写成一个意思
2. 必须体现“架构边界”和“运行时决策”的区别

### 作业 3：画出权限决策流水线

请按你的理解写出一条 5 到 8 步的权限判断链。

要求：

1. 必须包含 deny 或 deny rules
2. 必须包含工具特定权限检查
3. 必须包含 mode
4. 必须包含 allow / ask / deny 中的最终结果

### 作业 4：比较不同 agent 的工具边界

请比较以下几类执行体的工具边界：

1. 普通主线程
2. async agent
3. custom agent
4. coordinator mode

要求：

1. 至少写出两点差异
2. 必须说明为什么会有这些差异
3. 至少提到一次递归或主线程状态问题

### 作业 5：解释“展示前过滤”和“调用时判断”

请回答：

1. `filterToolsByDenyRules(...)` 在解决什么问题
2. `permissions.ts` 的运行时判断又在解决什么问题

要求：

1. 必须举出一个如果只做其中一层会出问题的例子
2. 必须体现两层机制的职责不同

### 作业 6：解释规则来源

请回答：

1. permission rules 可能来自哪些来源
2. 为什么 managed settings 和 project settings 需要区分
3. `allowManagedPermissionRulesOnly` 大概在解决什么问题

要求：

1. 至少提到 `permissionsLoader.ts`
2. 必须体现“规则不是硬编码在内存里”

### 作业 7：写一段 Day 5 总结

写一个 `220-380` 字的小总结，要求包含：

1. 你今天如何理解工具权限模型
2. 你认为哪一个环节最容易被误解
3. 你准备在第 6 天继续追哪条主线

## 自检清单

今天结束前，逐项确认：

- 我知道 `Tool.ts` 定义的是工具公共契约
- 我知道 `ToolPermissionContext` 为什么是工具权限的核心上下文
- 我知道静态工具限制和运行时权限判断不是一回事
- 我知道 async agents、coordinator mode 等执行体为什么有不同工具边界
- 我知道 permission mode 会真实改变权限决策
- 我知道 deny / ask / allow 不是简单 UI 文案，而是执行结果
- 我知道权限系统既影响工具展示，也影响工具执行
- 我能复述一条完整的工具权限决策主线

## 明日预告

第 6 天建议进入文件系统、shell 与高风险工具的具体权限场景，把抽象权限模型落到真实工具行为上。

明天的重点文件预计是：

1. [restored-src/src/utils/permissions/filesystem.ts](../restored-src/src/utils/permissions/filesystem.ts)
2. [restored-src/src/utils/permissions/shellRuleMatching.ts](../restored-src/src/utils/permissions/shellRuleMatching.ts)
3. [restored-src/src/tools/BashTool](../restored-src/src/tools/BashTool)
4. [restored-src/src/tools/FileReadTool](../restored-src/src/tools/FileReadTool)
5. [restored-src/src/tools/FileEditTool](../restored-src/src/tools/FileEditTool)

明天的目标是把今天的权限模型继续推进到“具体文件读写和 shell 执行为什么会被放行、询问或拒绝”。
