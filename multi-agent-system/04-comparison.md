# 三系统全维度对比

## 系统概览

| | OpenClaw | Clowder-ai (Cat Café) | Claude Code Teams |
|---|---|---|---|
| **定位** | 个人 AI 助手网关 | 多 Agent 协作编程平台 | 单模型并行任务框架 |
| **核心场景** | 20+ 渠道消息路由 | 软件开发中的代码协作 | 编程任务的并行执行 |
| **解决的问题** | 让 AI 在所有聊天平台都能用 | 让不同 AI 像团队一样写代码 | 让一个模型并行干多件事 |
| **技术栈** | TypeScript / Node.js | TypeScript / Fastify / Next.js | TypeScript / Node.js |
| **开源** | MIT | MIT | 闭源（有源码逆向） |

## 架构模式

| | OpenClaw | Clowder-ai | Claude Code Teams |
|---|---|---|---|
| **编排模式** | 星型中心化（Gateway） | P2P 去中心化（无 Boss） | 层级制（Lead + Workers） |
| **中心节点** | Gateway（必须） | AgentRouter（转发，不决策） | Team Lead（决策+分配） |
| **Agent 关系** | 隔离，通过 Gateway 通信 | 平等，可互相 @ | 主从，Lead 管理 Worker |

## Agent 能力

| | OpenClaw | Clowder-ai | Claude Code Teams |
|---|---|---|---|
| **多模型** | 支持不同模型 | **核心特色**：Claude/Codex/Gemini 跨厂商 | 全是 Claude（可换大小） |
| **Agent 身份** | SOUL.md 定义人格 | 固定角色：布偶猫/缅因猫/暹罗猫 | 按任务动态分配 |
| **生命周期** | 持久化，有独立状态 | 持久化，有身份和记忆 | 临时 Worker 或持久 Teammate |
| **Agent 能力** | 取决于底层模型 | 完整 Agent（文件/命令/MCP） | 完整 Agent（文件/命令/工具） |

## 通信机制

| | OpenClaw | Clowder-ai | Claude Code Teams |
|---|---|---|---|
| **通信分类** | 共享频道型 | 共享频道型 | 内部总线型 |
| **通信方式** | Gateway WebSocket 路由 | stdout + MCP callback + @mention | 文件邮箱 + 任务列表 |
| **A2A 通信** | 默认关闭，需显式开启 | 核心功能，@mention 直接触发 | 框架原生，SendMessage 工具 |
| **消息格式** | 类聊天消息 | 类聊天消息 + NDJSON | 文本 + 结构化协议 |
| **通信延迟** | 高（经 Gateway） | 中（经 AgentRouter） | 低（文件直写） |

## 任务管理

| | OpenClaw | Clowder-ai | Claude Code Teams |
|---|---|---|---|
| **任务系统** | 渠道绑定路由 | Feature lifecycle + 质量门 | TaskCreate/Update/List/Get |
| **任务分配** | Gateway 按规则分发 | Agent 主动 @ 其他 Agent | Lead 创建 → Worker 认领 |
| **进度追踪** | 无内置 | Feature 状态机 | 显式状态：pending → in_progress → completed |
| **跨模型 Review** | 无 | **强制要求** | 无（同一模型） |

## 认证与调用

| | OpenClaw | Clowder-ai | Claude Code Teams |
|---|---|---|---|
| **调用方式** | CLI + WebSocket API | CLI 子进程（spawn） | 内部 API 调用（非 CLI） |
| **认证** | CLI OAuth / API Key | CLI OAuth（订阅额度） | 框架内部，无需单独认证 |
| **费用来源** | 订阅或 API 额度 | 订阅额度 | 订阅额度（Claude Max） |

## 适用场景

| 场景 | 推荐系统 | 原因 |
|------|---------|------|
| 让 AI 管理所有聊天渠道 | OpenClaw | 20+ 渠道原生支持 |
| 多模型协作写代码 | Clowder-ai | 跨模型 review、知识共享 |
| 单模型并行跑任务 | Claude Code Teams | 内部总线效率最高 |
| 需要跨模型交叉验证 | Clowder-ai | 不同模型的盲区不同 |
| 个人 AI 助手 | OpenClaw | 渠道覆盖最广 |

## 一句话总结

- **OpenClaw** 解决**接入广度** — 让 AI 无处不在
- **Clowder-ai** 解决**协作深度** — 让不同 AI 像团队一样互相制衡
- **Claude Code Teams** 解决**执行效率** — 让一个模型并行干多件事

三者目标不同，不是竞争关系，可按场景选择或组合使用。
