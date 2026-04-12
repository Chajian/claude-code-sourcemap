# 运行时与会话主链

## 一次 turn 的生命周期

1. REPL 接收输入
2. 输入规范化并进入队列
3. Query 组装消息协议
4. 模型响应触发工具调用或渲染输出
5. 会话状态写回存储并准备恢复

## 重点模块

- `restored-src/src/screens/REPL.tsx`
- `restored-src/src/state/AppStateStore.ts`
- `restored-src/src/query.ts`
- `restored-src/src/utils/sessionStorage.ts`
- `restored-src/src/utils/sessionRestore.ts`

## 推荐搭配阅读

- [流程页：用户输入到输出](../flows/user-input-to-output/index.html)
- [流程页：REPL turn 构造](../flows/repl-turn-construction/index.html)
- [学习档案：第 09 天](../archive/learning-path/day-09.md)
- [学习档案：第 12 天](../archive/learning-path/day-12.md)
