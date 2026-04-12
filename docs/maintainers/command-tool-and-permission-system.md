# 命令、工具与权限系统

## 注册入口

- `restored-src/src/commands.ts`
- `restored-src/src/tools.ts`

## 工具执行链

1. Query 生成工具调用
2. Tool Orchestration 分批调度
3. 权限上下文参与展示与执行判断
4. 工具返回结果并回写消息流

## 权限分层

- 静态工具边界
- `ToolPermissionContext`
- 文件系统规则
- shell 规则匹配
- hooks / stop / recovery 相关控制点

## 重点源码

- `restored-src/src/Tool.ts`
- `restored-src/src/tools/`
- `restored-src/src/utils/permissions/filesystem.ts`
- `restored-src/src/utils/permissions/shellRuleMatching.ts`

## 相关文档

- [流程页：工具执行](../flows/tool-execution/index.html)
- [学习档案：第 04 天](../archive/learning-path/day-04.md)
- [学习档案：第 13 天](../archive/learning-path/day-13.md)
