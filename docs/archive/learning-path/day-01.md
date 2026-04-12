# 第 1 天学习文档

项目：`claude-code-sourcemap`  
日期：`Day 1`

## 今日目标

今天不进入复杂源码实现，只建立正确的项目认知。

完成今天后，你应该能够明确回答：

1. 这个仓库和官方 Claude Code 源码是什么关系
2. 这个仓库的源码是如何被还原出来的
3. 后续学习时应该重点阅读哪个目录
4. 哪些目录目前不值得投入精力

## 学习成果要求

今天结束后，你至少要产出以下内容：

1. 一句话定义当前项目
2. 一张根目录结构说明
3. 一段 sourcemap 还原流程说明
4. 一份“后续重点阅读目录”清单
5. 一份自己的疑问列表

## 核心结论

先记住这 4 句话：

1. 这个仓库不是官方原始开发仓库，而是基于 npm 包和 sourcemap 还原出来的研究仓库。
2. `package/` 里主要是发布后的打包产物，不是最适合学习实现逻辑的地方。
3. `restored-src/` 是通过 `package/cli.js.map` 提取 `sourcesContent` 后生成的还原源码目录。
4. 后续学习主战场应该是 `restored-src/src/`。

## 今日学习材料

按顺序阅读以下文件：

1. [README.md](../README.md)
2. [extract-sources.js](../extract-sources.js)
3. [package/package.json](../package/package.json)
4. [package/README.md](../package/README.md)

辅助观察目录：

1. [项目根目录](..)
2. [restored-src](../restored-src)
3. [restored-src/src](../restored-src/src)

## 学习步骤

### 步骤 1：阅读仓库说明

阅读 [README.md](../README.md)，重点回答：

1. 仓库来源是什么
2. 还原目标是什么
3. 当前研究的是哪个版本
4. 仓库作者如何定义这个项目的合法使用范围

你需要提炼出的结论：

`claude-code-sourcemap` 是一个基于公开 npm 包和 source map 还原出来的非官方研究仓库。

### 步骤 2：理解还原脚本

阅读 [extract-sources.js](../extract-sources.js)。

只需要理解以下主流程：

1. 读取 `package/cli.js.map`
2. 解析 `sources` 和 `sourcesContent`
3. 清洗 source path
4. 将文件写入 `restored-src/`

你不需要今天深究 `source-map` 这个库的 API 细节，但必须理解：

这个仓库里的大量源码文件并不是手工整理出来的，而是从 sourcemap 反推出的。

### 步骤 3：确认 npm 包身份

阅读 [package/package.json](../package/package.json)。

重点记录：

1. npm 包名
2. 版本号
3. 命令入口
4. Node 版本要求

今天要写进笔记里的关键事实：

1. 包名是 `@anthropic-ai/claude-code`
2. 当前研究版本是 `2.1.88`
3. CLI 二进制名是 `claude`
4. 发布入口是 `cli.js`

### 步骤 4：确认产品对外定位

阅读 [package/README.md](../package/README.md)。

你要提炼出的不是实现，而是产品定位：

1. 它是一个 terminal 中运行的 agentic coding tool
2. 它可以理解代码库
3. 它可以编辑文件
4. 它可以执行命令
5. 它可以处理 git workflow

### 步骤 5：整理目录认知

观察根目录后，给目录分层：

#### 需要重点关注

1. `restored-src/src/`

#### 需要知道但不必深挖

1. `package/`
2. `restored-src/`
3. `claude-code-2.1.88.tgz`

#### 现阶段先跳过

1. `restored-src/node_modules/`

## 今日建议时间安排

1. `15 分钟`：阅读仓库说明
2. `20 分钟`：阅读还原脚本
3. `15 分钟`：阅读 npm 包信息
4. `20 分钟`：观察目录结构并整理分类
5. `20 分钟`：完成笔记与作业

总时长建议：`90 分钟`

## 今日笔记模板

你可以直接照下面这个模板填写：

```md
# Day 1 学习笔记

## 1. 项目一句话定义

## 2. 当前仓库与官方源码的关系

## 3. sourcemap 还原流程
1.
2.
3.
4.

## 4. 根目录结构理解
- package:
- restored-src:
- restored-src/src:
- restored-src/node_modules:
- extract-sources.js:

## 5. 结论
- 后续重点阅读目录：
- 当前不重点阅读目录：

## 6. 我今天的疑问
1.
2.
3.
```

## 今日作业

### 作业 1：一句话定义项目

请用你自己的话写一句不超过 50 字的项目定义。

要求：

1. 必须包含“npm 包”或“source map”
2. 必须体现“非官方研究仓库”这个事实

### 作业 2：手写还原流程

不要抄代码，凭理解写出还原流程 4 步。

要求：

1. 每一步不超过 20 字
2. 必须包含 `cli.js.map`
3. 必须包含 `restored-src`

### 作业 3：目录分级

把下面目录分成三类：重点阅读、了解即可、暂时跳过。

1. `package/`
2. `restored-src/src/`
3. `restored-src/node_modules/`
4. `claude-code-2.1.88.tgz`
5. `extract-sources.js`

### 作业 4：回答 4 个问题

请书面回答：

1. 为什么后续重点看 `restored-src/src/`？
2. 为什么 `package/cli.js` 不适合作为主要学习入口？
3. `restored-src/` 和 `package/` 的关系是什么？
4. 这个项目为什么不能直接当成官方内部仓库来理解？

### 作业 5：输出一份 Day 1 总结

写一个 `150-300` 字的小总结，要求包含：

1. 你今天新建立的认知
2. 你认为这个仓库最值得研究的地方
3. 你明天准备进入哪个文件

## 自检清单

今天结束前，逐项确认：

- 我知道这个仓库不是官方原始仓库
- 我知道源码是从 `cli.js.map` 还原的
- 我知道后续主要看 `restored-src/src/`
- 我知道 `restored-src/node_modules/` 现在不该优先读
- 我能用自己的话解释 `package/` 与 `restored-src/` 的关系

## 明日预告

第 2 天将进入 CLI 启动链路，重点阅读：

1. [restored-src/src/entrypoints/cli.tsx](../restored-src/src/entrypoints/cli.tsx)

明天的目标是弄清楚 Claude Code 启动后会如何分流到不同运行路径。
