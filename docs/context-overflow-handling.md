# OpenCode 上下文溢出处理方案

## 问题背景

LLM 有固定的上下文窗口限制（如 100K、200K tokens）。当对话历史、文件内容、工具输出累积超过限制时，会导致：
- API 返回错误
- 模型输出被截断
- 任务无法完成

## OpenCode 的处理策略

### 1. 自动会话摘要 (Automatic Summarization)

当 token 使用超过 90% 上下文阈值时触发：

```typescript
if (tokens > Math.max((model.info.limit.context - outputLimit) * 0.9, 0)) {
  // 调用 LLM 生成摘要
}
```

**摘要提示词：**
> "Provide a detailed but concise summary of our conversation above. Focus on information that would be helpful for continuing the conversation, including what we did, what we're doing, which files we're working on, and what we're going to do next."

### 2. 文件读取限制

| 限制项 | 默认值 | 说明 |
|--------|--------|------|
| 单次读取行数 | 2000 行 | 可用 offset/limit 分块读取 |
| 单行最大长度 | 2000 字符 | 超过部分截断并加 `...` |
| 图片/二进制 | 禁止读取 | 抛出明确错误 |

### 3. Git 快照机制

每个 step 开始时创建快照：

```typescript
case "start-step":
  snapshot = await Snapshot.track()
```

如果出错可恢复：

```typescript
await Snapshot.restore(snapshotHash)
```

### 4. 停止条件

```typescript
stopWhen: async ({ steps }) => 
  steps.length >= 1000 || processor.getShouldStop()
```

---

## 最佳实践

### 主动策略

1. **及时新建会话**
   - 当任务复杂、预计需要多轮交互时
   - 接近上下文 50-60% 时主动新建会话
   - 新会话带上历史摘要或关键文件路径

2. **分块读取大文件**
   ```typescript
   // 先读前 2000 行
   read(filePath: "xxx", offset: 0, limit: 2000)
   // 再根据需要读后续
   read(filePath: "xxx", offset: 2000, limit: 2000)
   ```

3. **使用 glob/grep 而非全文搜索**
   - 先定位文件，再精确读取
   - 避免一次性加载大量内容

4. **任务分解**
   - 复杂任务拆分为多个子任务
   - 每个子任务用独立会话

### 被动策略 (OpenCode 自动处理)

1. 自动摘要保留关键信息
2. 截断超长输出
3. Git 快照兜底

---

## 会话恢复示例

当新建会话继续工作时，提供：

```markdown
## 上下文摘要
- 之前任务：实现用户登录功能
- 已完成：User 实体、UserService 基础方法
- 待完成：UserController、登录 API
- 关键文件：src/main/java/com/aidemo/service/UserService.java
```

---

## 成本考量

每次摘要调用消耗额外 token，建议：
- 简单任务：一个会话完成
- 复杂任务：宁可通过子 agent 分而治之，也不要依赖自动摘要

---

## 总结

| 策略 | 触发条件 | 成本 | 推荐度 |
|------|----------|------|--------|
| 新建会话 | 接近限制前 | 低 | ⭐⭐⭐⭐⭐ |
| 自动摘要 | >90% 上下文 | 中 | ⭐⭐⭐ |
| 截断输出 | 超长输出 | 低 | ⭐⭐⭐ |
| Git 快照 | 每 step | 低 | ⭐⭐⭐⭐ |
