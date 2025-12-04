# 智能多CLI编排系统 - 功能清单与兼容性检查报告

生成时间：2025-12-04

---

## 一、项目功能清单

### 1. MCP Servers（模型上下文协议服务器）

| 组件 | 路径 | 功能 | 状态 |
|------|------|------|------|
| **Gemini MCP Server** | `mcp-servers/gemini-server/index.js` | 封装Gemini CLI，提供代码审查能力 | ✅ 已修复 |
| **Codex MCP Server** | `mcp-servers/codex-server/index.js` | 封装Codex CLI，提供代码生成能力 | ✅ 已修复 |
| **统一配置** | `mcp-config.json` | MCP服务器统一配置（代理、权限、参数） | ✅ 已创建 |

### 2. Claude Code扩展

| 组件 | 路径 | 功能 | 状态 |
|------|------|------|------|
| **Command** | `.claude/commands/ai-team.md` | AI团队协作命令 | ⚠️ 需检查格式 |
| **Skill** | `.claude/skills/ai-orchestrator/` | AI编排技能 | ❌ 格式不符合规范 |

### 3. 工作流阶段

| 阶段 | 执行者 | 输入 | 输出 |
|------|--------|------|------|
| **Phase 1: 需求分析** | Claude Code | 用户任务描述 | `.ai-orchestrator/phase1_requirements.json` |
| **Phase 2: 代码生成** | Codex CLI | 需求JSON | `.ai-orchestrator/phase2_code.md` |
| **Phase 3: 代码审查** | Gemini CLI | 生成的代码 | `.ai-orchestrator/phase3_review.md` |

---

## 二、兼容性检查结果

### ❌ 严重问题

#### 1. Skill格式不符合规范

**当前结构：**
```
.claude/skills/ai-orchestrator/
├── skill.yaml          ❌ 错误：应该是SKILL.md
├── scripts/
│   └── orchestrate.sh
└── prompts/
    ├── phase1-analyze.md
    ├── phase2-code.md
    └── phase3-review.md
```

**正确结构：**
```
.claude/skills/ai-orchestrator/
├── SKILL.md            ✅ 必须是SKILL.md（Markdown + YAML frontmatter）
├── scripts/
│   └── orchestrate.sh
└── prompts/
    ├── phase1-analyze.md
    ├── phase2-code.md
    └── phase3-review.md
```

**修复方案：** 将`skill.yaml`转换为`SKILL.md`格式

#### 2. 缺少MCP配置文件

**问题：** 项目根目录没有`.mcp.json`文件

**影响：** Claude Desktop无法自动发现和加载MCP服务器

**修复方案：** 创建`.mcp.json`文件

### ⚠️ 警告问题

#### 1. Command缺少YAML frontmatter

**当前：** `.claude/commands/ai-team.md` 没有标准的frontmatter

**建议：** 添加description等元数据

### ✅ 已修复问题

#### 1. Codex sandbox权限
- **问题：** 原配置为`read-only`，无法写入文件
- **修复：** 改为`workspace-write`

#### 2. Gemini代理配置
- **问题：** 缺少代理配置，无法连接Google服务器
- **修复：** 添加代理支持和环境变量配置

---

## 三、修复任务清单

### 高优先级

- [ ] 将`skill.yaml`转换为`SKILL.md`格式
- [ ] 创建`.mcp.json`配置文件
- [ ] 为`ai-team.md`添加YAML frontmatter

### 中优先级

- [ ] 测试Gemini MCP Server
- [ ] 测试Codex MCP Server
- [ ] 测试完整工作流

### 低优先级

- [ ] 添加Hooks配置（可选）
- [ ] 创建Subagent配置（可选）

---

## 四、配置参考

### mcp-config.json（已创建）

```json
{
  "proxy": {
    "enabled": true,
    "http": "http://127.0.0.1:15236",
    "https": "http://127.0.0.1:15236"
  },
  "gemini": {
    "command": "gemini",
    "defaultArgs": ["-o", "stream-json", "--yolo"],
    "sandbox": "workspace-write",
    "environment": {
      "GEMINI_IDE_INTEGRATION": "false"
    }
  },
  "codex": {
    "command": "codex",
    "defaultArgs": ["exec", "--skip-git-repo-check"],
    "sandbox": "workspace-write",
    "approvalPolicy": "on-failure"
  }
}
```

### .mcp.json（需创建）

```json
{
  "mcpServers": {
    "codex": {
      "type": "stdio",
      "command": "node",
      "args": ["mcp-servers/codex-server/index.js"]
    },
    "gemini": {
      "type": "stdio",
      "command": "node",
      "args": ["mcp-servers/gemini-server/index.js"]
    }
  }
}
```

---

## 五、测试计划

### 1. 单元测试

| 测试项 | 命令 | 预期结果 |
|--------|------|---------|
| Gemini MCP tools/list | `echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' \| node mcp-servers/gemini-server/index.js` | 返回工具列表 |
| Codex MCP tools/list | `echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' \| node mcp-servers/codex-server/index.js` | 返回工具列表 |
| Gemini调用测试 | 调用gemini工具 | 返回AI响应 |
| Codex调用测试 | 调用codex工具 | 返回代码生成 |

### 2. 集成测试

| 测试项 | 步骤 | 预期结果 |
|--------|------|---------|
| 完整工作流 | 执行`/ai-team "实现Hello World"` | 三阶段全部完成 |
| 上下文传递 | 使用相同conversationId多次调用 | 保持对话历史 |

---

报告结束
