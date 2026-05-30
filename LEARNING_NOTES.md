# Moltis 学习笔记

> 仓库：https://github.com/moltis-org/moltis
> 原始语言：Rust
> Fork 时间：2026-05

## 这是什么

Moltis 是一个用 Rust 写的**个人持久化 Agent 服务**。一个二进制文件跑起来就是完整的个人助手——支持多 LLM 提供商、语音、长期记忆、多平台接入（Telegram/WhatsApp/Discord）、MCP 工具调用。

## 核心设计

```
┌─────────────────────────────────┐
│         Moltis Agent            │
│                                 │
│  ┌─────────┐  ┌──────────────┐  │
│  │ Memory  │  │ MCP Tools    │  │
│  │ (持久化) │  │ (600+ 工具)  │  │
│  └────┬────┘  └──────┬───────┘  │
│       │              │          │
│  ┌────┴──────────────┴───────┐  │
│  │    LLM Provider Layer     │  │
│  │  (OpenAI/Claude/Gemini...) │  │
│  └────────────┬──────────────┘  │
│               │                 │
│  ┌────────────┴──────────────┐  │
│  │   Multi-Channel Output    │  │
│  │  Telegram/WhatsApp/Discord│  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

## 与个人伴随类智能体的关系

东方国信智能体系统部的「个人伴随类智能体」需要：

1. **持久化记忆** → Moltis 有完整的记忆系统，用户画像 + 对话历史
2. **多平台接入** → Moltis 支持 Telegram/WhatsApp/Discord，即「跟着人走」
3. **MCP 工具调用** → Moltis 原生支持 MCP，Agent 能调用外部工具
4. **安全沙箱** → Moltis 强调 sandboxed execution，这是企业级产品必备

Moltis 是目前最接近「个人伴随类智能体」形态的开源项目。虽然用 Rust 写（学习曲线陡），但架构思路可以直接借鉴。

## 关键技术点

1. **Rust 单体架构**：一个二进制包含 Agent 核心 + Web UI + 多渠道接入。性能好、部署简单
2. **多 LLM Provider**：通过统一的 Provider 接口抽象，切换模型不改业务代码
3. **MCP 原生**：支持 Model Context Protocol，Agent 能发现和调用工具
4. **事件总线**：Server-Sent Events 实现实时推送，Agent 状态变化即时通知前端
5. **Web UI**：TypeScript + Preact + Tailwind，清晰的前后端分离

## 学到的东西

1. **个人 Agent 该如何记忆**：不能只靠对话上下文（短期），需要 SQLite/向量数据库做持久化（长期）。Moltis 的设计是短期 = 上下文窗口，长期 = 持久化存储 + 向量检索

2. **Agent 的输出不只是文本**：Moltis 支持语音、Web UI、多平台消息——Agent 需要多渠道分发能力

3. **安全不是可选项**：用户的对话历史、API Key、个人数据都在 Agent 里，沙箱隔离是刚需

4. **「一个二进制」的优势**：Rust 编译成单个可执行文件，部署 = 复制文件 + 运行。对 C 端产品来说，这比 Docker 更适合

## 局限性

- Rust 学习曲线高，团队如果不是 Rust 技术栈，参考架构即可
- 偏向个人使用场景，企业级多租户、权限管理等较弱
- 中文生态支持有限
