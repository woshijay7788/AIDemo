# OpenCode 初始化与项目理解原理

## 每次启动都需要 /init 吗？

**不是！** `/init` 只需要在以下情况运行：

1. **首次使用项目** - 第一次在该项目使用 OpenCode
2. **项目结构重大变更** - 添加了新框架、大幅重构后
3. **手动更新 AGENTS.md** - 想更新项目规则时

## 工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                        首次使用                                  │
├─────────────────────────────────────────────────────────────────┤
│  cd my-project          # 进入项目目录                         │
│  opencode                # 启动 OpenCode (自动创建会话)         │
│  /connect                # 连接 AI provider (只需一次)        │
│  /init                   # 初始化项目，生成 AGENTS.md           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      后续使用                                    │
├─────────────────────────────────────────────────────────────────┤
│  cd my-project          # 进入项目目录                         │
│  opencode                # 启动，会自动加载 AGENTS.md           │
│  /sessions               # 可选：恢复之前的会话                  │
└─────────────────────────────────────────────────────────────────┘
```

## /init 做了什么？

1. **扫描项目结构**
   - 检测使用的语言和框架 (Java/Node/Python...)
   - 识别包管理器 (Maven/npm/Cargo...)
   - 发现测试框架和构建工具

2. **生成 AGENTS.md**
   - 包含项目特定的指令
   - 构建/测试命令
   - 代码风格规范
   - 项目结构说明

3. **创建 .opencode/ 目录** (可选)
   - 项目级别的配置文件
   - 模型偏好设置
   - Agent 配置

## AGENTS.md 的优先级

OpenCode 启动时按顺序查找规则文件：

| 优先级 | 位置 | 作用域 |
|--------|------|--------|
| 1 | `~/.claude/CLAUDE.md` | 全局 (Claude Code 兼容) |
| 2 | `~/.config/opencode/AGENTS.md` | 全局 (个人规则) |
| 3 | 当前目录向上遍历 `AGENTS.md` / `CLAUDE.md` | 项目/团队 |
| 4 | `CONTEXT.md` | 手动指定上下文 |

**建议**: 将项目级 `AGENTS.md` 提交到 Git，团队共享。

## 会话机制

### 会话是什么？

每次与 OpenCode 的对话就是一个**会话** (Session)：
- 包含完整的消息历史
- 存储在 `.opencode/sessions/` 目录
- 可以随时恢复

### 会话命令

```bash
/sessions      # 查看所有会话
/resume        # 恢复上一个会话
/clear         # 开始新会话（保留 AGENTS.md）
```

### 会话持久化

```
.opencode/
├── sessions/
│   ├── session-xxx/
│   │   ├── messages.json    # 对话历史
│   │   └── state/          # 快照状态
│   └── ...
└── config.json              # 项目配置
```

## 项目理解原理

OpenCode 通过以下方式理解项目：

### 1. AGENTS.md (主要)
```markdown
# AGENTS.md 示例

## 构建命令
mvn clean install

## 代码规范
- 使用 4 空格缩进
- 类名用 PascalCase
```

### 2. 运行时分析
- 读取 `package.json` / `pom.xml` / `go.mod` 获取依赖
- 检测 `.git` 目录确定项目根目录
- LSP 服务器提供代码语义分析

### 3. 工具调用
- `glob` - 查找文件
- `grep` - 搜索代码
- `read` - 读取文件内容
- `bash` - 执行构建/测试命令

### 4. System Prompt 注入
启动时，OpenCode 自动将以下信息注入 System Prompt：
```
Working directory: /Users/jay/git/AIDemo
Platform: darwin
Today's date: 2026-02-23
```

## 总结

| 阶段 | 需要做什么 | 频率 |
|------|-----------|------|
| 首次 | `/connect` + `/init` | 每个 provider 一次 |
| 日常 | `opencode` 直接开始 | 每次 |
| 恢复 | `/sessions` 选择会话 | 需要继续工作时 |
| 更新 | `/init` 重新生成 | 项目结构大改时 |

**关键点**: AGENTS.md 是一次性生成、长期使用的。日常启动 OpenCode 时会自动加载它，不需要每次都运行 `/init`。
