# claude-code-sourcemap

[![linux.do](https://img.shields.io/badge/linux.do-huo0-blue?logo=linux&logoColor=white)](https://linux.do)

> [!WARNING]
> This repository is **unofficial** and is reconstructed from the public npm package and source map analysis, **for learning and research purposes only**.
> It does **not** represent Anthropic's original internal development repository.
>
> 本仓库为**非官方**整理版，基于公开 npm 发布包与 source map 分析还原，**仅用于学习与研究**。
> **不代表** Anthropic 官方内部开发仓库。

## 仓库定位

这个 fork 当前专注于两件事：

1. 保留基于 `@anthropic-ai/claude-code` npm 包与 `cli.js.map` 还原出的可研究源码
2. 提供一套面向开源读者的学习文档，帮助系统理解 Claude Code 的架构与主链路

如果你是第一次进入这个仓库，不建议直接从大文件硬啃。  
建议先读学习文档，再回到 `restored-src/src/` 对照源码。

## 项目来源

- npm 包：[@anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- 当前还原版本：`2.1.88`
- 还原方式：提取 `cli.js.map` 中的 `sourcesContent`
- 仓库性质：研究与学习导向，不是官方开发仓库

## 学习入口

学习资料从这里开始：

- [学习总目录](docs/README.md)
- [14 天学习计划 - 第 1 天](docs/day-01-learning-plan.md)
- [14 天学习计划 - 第 14 天](docs/day-14-learning-plan.md)
- [项目架构图](docs/claude-code-architecture.drawio)

推荐阅读顺序：

1. 先看 [学习总目录](docs/README.md)
2. 再按 `day-01` 到 `day-14` 顺序阅读
3. 阅读过程中只把源码主目录锁定在 `restored-src/src/`

## 推荐学习主线

建议按这条主线理解项目：

`CLI 启动 -> Main 装配 -> Commands/Tools 注册 -> AppState/REPL -> 输入分流 -> Query -> Tool Execution -> Session Persistence`

对应核心文件大致如下：

- `restored-src/src/entrypoints/cli.tsx`
- `restored-src/src/main.tsx`
- `restored-src/src/commands.ts`
- `restored-src/src/tools.ts`
- `restored-src/src/state/AppStateStore.ts`
- `restored-src/src/screens/REPL.tsx`
- `restored-src/src/query.ts`
- `restored-src/src/Tool.ts`
- `restored-src/src/utils/sessionStorage.ts`
- `restored-src/src/utils/sessionRestore.ts`

## 目录说明

```text
restored-src/src/         主要学习对象，还原后的 TypeScript 源码
restored-src/node_modules/ 还原出的依赖内容，通常不作为首要学习目标
package/                  npm 包原始产物
docs/                     学习文档、架构图与后续学习材料
extract-sources.js        source map 提取脚本
```

## 学习建议

- 优先读 `restored-src/src/`，不要先钻 `package/cli.js`
- 优先读注册表与入口文件，再追单个能力实现
- 把 `commands`、`tools`、`query`、`session` 当成四条主轴
- 每读完一个阶段，至少输出自己的模块图或术语表

## 版权与声明

- 源码版权归 [Anthropic](https://www.anthropic.com) 所有
- 本仓库仅用于技术学习、研究与架构分析
- 请勿将本仓库内容作为官方源码或官方实现说明引用
- 如有侵权或不适宜公开的内容，请提交 issue 或联系删除
