# 案例3：模块重构

> 使用MCP方式进行大型模块重构（保持上下文）

## 任务描述

重构一个遗留的用户模块，实现以下目标：
1. 分离Controller、Service、Repository层
2. 实现依赖注入
3. 优化数据库查询性能
4. 添加单元测试
5. 改进错误处理

## 场景背景

**现有代码问题**：
- Controller直接操作数据库（违反单一职责）
- 业务逻辑散落在各处（难以维护）
- 存在N+1查询问题（性能差）
- 没有单元测试（难以重构）
- 错误处理不统一

**目标架构**：
```
Controller（路由+验证）
    ↓
Service（业务逻辑）
    ↓
Repository（数据访问）
    ↓
Database
```

## 为什么用MCP方式？

**关键理由**：重构需要**整天持续开发**，多轮对话：

1. **第1轮**：分析现有代码，设计新架构
2. **第2轮**：重构Controller层
3. **第3轮**：Gemini审查，发现问题
4. **第4轮**：Codex根据审查意见改进
5. **第5轮**：继续重构Service层
6. **第6轮**：优化数据库查询
7. **...持续多轮**

**MCP的价值**：`conversationId`让三个AI共享同一个上下文，每轮对话都知道之前所有讨论！

## 前置准备

### 步骤1：创建Codex MCP Server

创建文件 `codex-mcp-server.js`：

```javascript
#!/usr/bin/env node
const { spawn } = require('child_process');
const readline = require('readline');
const fs = require('fs');
const path = require('path');

const CONTEXT_DIR = path.join(process.env.HOME, '.mcp-context');
if (!fs.existsSync(CONTEXT_DIR)) {
  fs.mkdirSync(CONTEXT_DIR, { recursive: true });
}

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.on('line', (line) => {
  const request = JSON.parse(line);
  const { prompt, conversationId } = request;

  // 读取历史上下文
  let context = '';
  if (conversationId) {
    const contextFile = path.join(CONTEXT_DIR, `${conversationId}.json`);
    if (fs.existsSync(contextFile)) {
      const history = JSON.parse(fs.readFileSync(contextFile, 'utf-8'));
      context = history.map(item => `${item.role}: ${item.content}`).join('\\n\\n');
    }
  }

  const fullPrompt = context ? `${context}\\n\\n${prompt}` : prompt;

  const codex = spawn('codex', ['exec', fullPrompt]);

  let output = '';
  codex.stdout.on('data', (data) => {
    output += data.toString();
  });

  codex.on('close', () => {
    if (conversationId) {
      const contextFile = path.join(CONTEXT_DIR, `${conversationId}.json`);
      let history = [];
      if (fs.existsSync(contextFile)) {
        history = JSON.parse(fs.readFileSync(contextFile, 'utf-8'));
      }
      history.push({ role: 'user', content: prompt });
      history.push({ role: 'assistant', content: output });
      fs.writeFileSync(contextFile, JSON.stringify(history, null, 2));
    }

    console.log(JSON.stringify({
      result: output,
      conversationId: conversationId || `conv_${Date.now()}`
    }));
  });
});
```

同样方式创建 `gemini-mcp-server.js`。

### 步骤2：连接MCP Server

```bash
# 给脚本添加执行权限
chmod +x codex-mcp-server.js gemini-mcp-server.js

# 连接Codex MCP Server
claude mcp add-json --scope user codex '{
  "type": "stdio",
  "command": "node",
  "args": ["/absolute/path/to/codex-mcp-server.js"]
}'

# 连接Gemini MCP Server
claude mcp add-json --scope user gemini '{
  "type": "stdio",
  "command": "node",
  "args": ["/absolute/path/to/gemini-mcp-server.js"]
}'

# 验证连接
claude mcp list
```

## 执行流程

### 第1轮：分析现有代码

在Claude Code中：

```
用户：重构用户模块，现有代码在src/users/目录

Claude：好的，我先分析现有代码结构...
       [生成conversationId: conv_refactor_123]

       发现的问题：
       1. UserController直接操作数据库
       2. 业务逻辑和数据访问混在一起
       3. 没有使用依赖注入
       4. 存在N+1查询问题

       重构方案：
       - 创建UserService（业务逻辑）
       - 创建UserRepository（数据访问）
       - Controller只负责路由和验证
       - 使用依赖注入容器
```

### 第2轮：重构Controller层

```
用户：先重构Controller层

Claude：好的，让Codex生成新的Controller...
       [通过MCP调用Codex，传入conversationId: conv_refactor_123]

Codex：[读取conv_refactor_123上下文，知道重构方案]
       生成的UserController：
       - 只负责路由和验证
       - 依赖注入UserService
       - 统一错误处理
       [代码保存到conv_refactor_123]

Claude：Controller已生成，让Gemini审查...
```

### 第3轮：Gemini审查

```
Claude：[通过MCP调用Gemini，传入conversationId: conv_refactor_123]

Gemini：[读取conv_refactor_123，看到重构方案和新Controller]
        审查发现：
        1. ✅ Controller职责清晰
        2. ⚠️ 依赖注入方式不够优雅，建议用装饰器
        3. ⚠️ 错误处理可以提取到中间件
        [审查报告保存到conv_refactor_123]
```

### 第4轮：改进Controller

```
用户：根据Gemini的建议改进

Claude：好的，让Codex改进...
       [通过MCP调用Codex，传入conversationId: conv_refactor_123]

Codex：[读取conv_refactor_123，看到Gemini的审查建议]
       改进后的Controller：
       - 使用装饰器实现依赖注入
       - 提取错误处理到中间件
       [改进代码保存到conv_refactor_123]
```

### 第5轮：继续重构Service层

```
用户：现在重构Service层

Claude：[conversationId: conv_refactor_123继续传递]
       让Codex生成UserService...

Codex：[读取上下文，知道Controller已完成，继续Service层]
       ...
```

### 持续多轮...

整天的重构过程中，所有对话都在**同一个上下文**（`conv_refactor_123`）中进行！

## 上下文文件示例

查看完整上下文：

```bash
cat ~/.mcp-context/conv_refactor_123.json
```

内容示例：
```json
[
  {
    "role": "user",
    "content": "重构用户模块..."
  },
  {
    "role": "assistant",
    "content": "分析结果：发现的问题..."
  },
  {
    "role": "user",
    "content": "先重构Controller层"
  },
  {
    "role": "assistant",
    "content": "生成的UserController代码..."
  },
  {
    "role": "user",
    "content": "审查Controller"
  },
  {
    "role": "assistant",
    "content": "审查报告：发现的问题..."
  }
  // ...持续记录所有对话
]
```

## 预期时间

- **整天持续开发**（6-8小时）
- 多轮迭代（10-20轮对话）
- 每轮对话3-10分钟

## 预期输出

### 重构后的文件结构

```
src/users/
├── controllers/
│   └── UserController.ts      # 路由+验证（重构后）
├── services/
│   └── UserService.ts         # 业务逻辑（新增）
├── repositories/
│   └── UserRepository.ts      # 数据访问（新增）
├── dto/
│   ├── CreateUserDto.ts       # 数据传输对象
│   └── UpdateUserDto.ts
├── entities/
│   └── User.ts                # 数据模型
└── tests/
    ├── UserController.test.ts
    ├── UserService.test.ts
    └── UserRepository.test.ts
```

## 学习要点

1. **MCP方式的核心价值**
   - `conversationId`机制实现上下文共享
   - 三个AI共享同一个记忆
   - 适合长期、多轮的协作开发

2. **大型重构策略**
   - 分层重构（Controller → Service → Repository）
   - 每层重构后审查
   - 根据审查意见改进
   - 持续迭代

3. **上下文管理**
   - 所有对话保存在`.mcp-context/`
   - 随时查看历史上下文
   - 新的对话继承旧的上下文

## 扩展练习

完成基础重构后，尝试：

1. **性能优化**
   ```
   用户：优化UserService的查询性能
   # conversationId继续传递，Codex知道现有实现
   ```

2. **添加缓存**
   ```
   用户：为UserService添加Redis缓存
   # Codex根据现有代码添加缓存层
   ```

3. **集成测试**
   ```
   用户：添加完整的集成测试
   # Codex知道所有重构后的代码结构
   ```

---

**提示**：这是最复杂的案例，需要理解MCP机制。建议先完成案例1和案例2，再尝试这个案例。

**核心收获**：理解`conversationId`如何实现多AI协作的"共享记忆"！
