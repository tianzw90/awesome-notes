# Claude Code + Mem0 使用教程

让你的 Claude Code 拥有跨会话的持久记忆——记住项目架构、编码偏好、技术决策和踩过的坑。

项目地址：https://github.com/mem0ai/mem0

m0-FvsAoMdU8dbolfrhY1WfB7WityIeoKca5UMGKatN

---

## 前置准备

1. 已安装 Claude Code（命令行中可运行 `claude`）
2. 注册 Mem0 账号：访问 https://app.mem0.ai，获取 API Key（格式：`m0-xxxxxxxxx`）

---

## 安装方式

有两种方式，推荐方式 A。

### 方式 A：插件市场安装（推荐）

这是官方推荐的方式，包含 MCP 服务器 + 生命周期 Hook + SDK Skill，实现**自动**记忆管理。

在 Claude Code 中运行：

```
/plugin marketplace add mem0ai/mem0
/plugin install mem0@mem0-plugins
```

然后设置环境变量。在你的 shell 配置文件（`~/.bashrc` 或 `~/.zshrc`）中添加：

```bash
export MEM0_API_KEY="m0-你的API Key"
```

重启终端和 Claude Code。

**安装后你会获得：**

- MCP 工具：search_memories、add_memory、update_memory、delete_memory
- 生命周期 Hook：自动在关键时刻捕获和注入记忆
- SDK Skill：教 Claude Code 如何在代码中使用 Mem0

### 方式 B：手动 MCP 配置（仅 MCP 工具，无自动 Hook）

如果插件市场不可用，可以手动配置。

**方法 1：一条命令**

```bash
claude mcp add mem0 -- npx -y @mem0/mcp --mem0-api-key 你的KEY
```

**方法 2：编辑配置文件**

编辑 `~/.claude/settings.json`，在 `mcpServers` 中添加：

```json
{
  "mcpServers": {
    "mem0": {
      "command": "npx",
      "args": ["-y", "@mem0/mcp"],
      "env": {
        "MEM0_API_KEY": "m0-你的API Key"
      }
    }
  }
}
```

重启 Claude Code。

> 注意：方式 B 只有 MCP 工具，没有自动 Hook。Claude Code 不会自动搜索和保存记忆，需要你手动触发或通过 CLAUDE.md 引导。

---

## 两种方式的区别

| | 方式 A（插件市场） | 方式 B（手动 MCP） |
|---|---|---|
| MCP 工具 | ✅ | ✅ |
| 自动记忆注入 | ✅ 每次新会话自动搜索 | ❌ 需手动触发 |
| 自动记忆保存 | ✅ 任务完成/会话结束时自动保存 | ❌ 需手动触发 |
| 上下文压缩前保存 | ✅ 压缩前自动存摘要 | ❌ |
| 配置复杂度 | 低 | 低 |

---

## 方式 A 的自动化行为详解

通过插件市场安装后，Mem0 会在以下时机自动工作：

**会话开始时：**
自动调用 search_memories，加载与当前项目相关的历史记忆并注入上下文。

**每次你发消息时：**
在处理你的消息之前，搜索 Mem0 中与当前提示相关的记忆并注入。短提示（< 20 字符）会被跳过以减少延迟。

**上下文压缩前：**
当对话变长需要压缩时，自动保存一份完整的会话摘要——包括目标、完成的工作、决策、修改的文件和当前状态。

**任务完成后：**
自动提取并保存关键知识：成功的策略、失败的尝试、架构决策、新的编码约定。

**会话结束时：**
保存所有未存储的知识，并通过 Mem0 REST API 捕获会话记录。

---

## 方式 B 需要额外配置 CLAUDE.md

如果你用的是方式 B（手动 MCP），需要在 CLAUDE.md 中添加规则来引导 Claude Code 主动使用记忆。

在项目根目录的 `CLAUDE.md`（项目级）或 `~/.claude/CLAUDE.md`（全局）中添加：

```markdown
# MCP Servers

- **mem0**: 跨会话持久记忆。
  每次会话开始时，必须先调用 search_memories 搜索相关上下文，不要让用户重复解释。
  在以下情况发生时，必须调用 add_memory 保存：
  - 发现项目架构信息
  - 确定编码约定
  - 调试得到关键洞察
  - 做出重要技术决策
  - 了解到用户偏好
  当已有记忆过时时，使用 update_memory 更新。
  保存的内容例如："项目使用 PostgreSQL + Prisma"、"测试用 pytest -v 运行"、"认证使用 JWT，在中间件中验证"。
  宁可多存，不要少存——未来的会话会因此受益。
```

---

## 实际使用示例

### 场景 1：首次会话

```
你：帮我梳理一下这个项目的架构

Claude Code：（分析代码后）
这个项目是一个 Next.js 14 电商应用...
- 前端：Next.js App Router + Tailwind CSS
- 后端：API Routes + Prisma ORM
- 数据库：PostgreSQL
- 认证：NextAuth.js + JWT
- 部署：Vercel

（Mem0 自动保存这些架构信息）
```

### 场景 2：几天后的新会话

```
你：给订单模块加一个退款功能

Claude Code：（自动搜索记忆，找到之前保存的架构信息）
基于之前的分析，项目使用 Next.js App Router + Prisma + PostgreSQL。
订单模型在 prisma/schema.prisma 中...
我来在现有架构基础上添加退款功能。

（不需要你重新解释项目结构）
```

### 场景 3：手动存储重要信息

```
你：记住，这个项目的 API 响应格式统一用 { code, data, message }

Claude Code：好的，我把这个约定保存到记忆中了。
（调用 add_memory 保存）
```

### 场景 4：手动搜索记忆

```
你：搜一下之前关于数据库迁移的记忆

Claude Code：（调用 search_memories）
找到以下相关记忆：
1. 2026-05-10：将用户表 email 字段改为唯一索引，迁移文件在 prisma/migrations/...
2. 2026-05-08：决定使用软删除而非硬删除，添加了 deletedAt 字段
```

---

## 可用的命令和工具

安装后，你可以在 Claude Code 中使用以下自然语言指令：

| 你说的话 | Claude Code 调用的工具 |
|---------|----------------------|
| "记住这个项目用的是 PostgreSQL" | add_memory |
| "搜一下之前关于认证的记忆" | search_memories |
| "更新记忆：我们已经从 Express 迁移到 Fastify 了" | update_memory |
| "删除关于旧数据库配置的记忆" | delete_memory |
| "看看我有哪些记忆" | search_memories（广泛搜索） |

---

## 多项目隔离

Mem0 默认按 user_id 区分记忆。如果你有多个项目需要隔离，可以设置不同的 MEM0_USER_ID：

**方法 1：在每个项目的 .env 中设置不同 ID**

```bash
# 项目 A 的 .env
MEM0_USER_ID=project-ecommerce

# 项目 B 的 .env
MEM0_USER_ID=project-blog
```

**方法 2：在 CLAUDE.md 中指定**

```markdown
## 记忆规则
- 所有 memory 操作的 metadata 中必须包含 project: "ecommerce-app"
- 搜索时必须按 project 过滤
```

---

## 验证安装

### 检查 MCP 连接

在 Claude Code 中直接问：

```
你：你有哪些可用的记忆工具？
```

如果安装成功，它会列出 search_memories、add_memory 等工具。

### 测试记忆功能

```
你：记住我喜欢用 TypeScript，偏好函数式编程风格

（等几秒）

你：搜索一下我的编程偏好

Claude Code 应该返回刚才保存的内容。
```

### 检查环境变量

```bash
echo $MEM0_API_KEY
# 应该输出你的 Key
```

---

## 常见问题

**Q：提示 "Connection failed"**
确认环境变量已设置：`echo $MEM0_API_KEY`。如果为空，重新在 shell 配置文件中 export 并重启终端。

**Q：没有工具出现**
重启 Claude Code。如果用的方式 B，检查 `~/.claude/settings.json` 的 JSON 格式是否正确。

**Q：记忆没有被自动保存**
确认你是通过插件市场安装的（方式 A）。方式 B 的 MCP-only 模式需要手动操作或 CLAUDE.md 引导。

**Q：搜索不到刚存的记忆**
记忆处理是异步的，新记忆可能需要几秒钟才能被搜索到。

**Q：会消耗多少 token？**
每次记忆搜索/保存会消耗少量 Claude Code token（工具调用开销）+ Mem0 侧的 LLM token（提取事实）。但长远来看，避免重复解释项目背景节省的 token 远大于这个开销。

---

## 总结

| 你的情况 | 推荐方式 |
|---------|---------|
| 想要全自动、省心 | 方式 A（插件市场） |
| 插件市场不可用或想精细控制 | 方式 B（手动 MCP）+ CLAUDE.md 规则 |
| 完全不想数据上云 | 自托管 Mem0 + MCP 接入 |

最简单的路径：注册 Mem0 → 拿到 API Key → 插件市场一键安装 → 重启 → 开始用。