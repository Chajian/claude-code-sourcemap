# 架构总览

## 仓库边界

本仓库是基于 npm 包和 source map 还原的学习型代码库，不是 Anthropic 官方开发仓库。维护工作应同时关注“源码主链是否易读”和“文档导航是否稳定”。

## 六层视图

1. 还原产物层：`package/`、`extract-sources.js`
2. 入口层：`restored-src/src/entrypoints/cli.tsx`、`restored-src/src/main.tsx`
3. 注册层：`restored-src/src/commands.ts`、`restored-src/src/tools.ts`
4. 运行时层：`restored-src/src/state/`、`restored-src/src/screens/REPL.tsx`
5. 执行层：`restored-src/src/query.ts`、`restored-src/src/Tool.ts`
6. 扩展层：`restored-src/src/services/mcp/`、skills、plugins、Buddy 相关模块

## 关键入口

- CLI 启动：`restored-src/src/entrypoints/cli.tsx`
- 主装配：`restored-src/src/main.tsx`
- Query 主链：`restored-src/src/query.ts`
- 工具契约：`restored-src/src/Tool.ts`

## 相关文档

- [运行时与会话主链](runtime-and-session-flow.md)
- [命令、工具与权限系统](command-tool-and-permission-system.md)
- [扩展点](extension-points.md)
- [流程页：用户输入到输出](../flows/user-input-to-output/index.html)
- [架构图素材](../assets/architecture/claude-code-architecture.drawio)
