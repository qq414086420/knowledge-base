# 通信模式：共享频道型 vs 内部总线型

## 核心分类

多 Agent 系统按通信方式可分为两大类：

```
1. 共享频道型（中介传话）
   ├── OpenClaw（Gateway 路由）
   ├── Clowder-ai（AgentRouter + stdout/MCP）
   └── 群聊机器人（把多个 AI 丢进一个群）

2. 内部总线型（同一间办公室）
   └── Claude Code Teams（文件邮箱 + 任务列表）
```

## 共享频道型

Agent 之间不直接认识，通过一个共享的消息频道（Gateway / Router / 群聊）作为中介转发消息。

### 通信拓扑

```
Agent A ──→ 共享频道 ──→ Agent B
              ↑
         中介层转发
```

### 代表实现

**OpenClaw — Gateway 路由**

```json
// 配置绑定规则，Gateway 自动分发
{ "channel": "slack", "peer": "#dev", "agent": "codex" }
```

- Agent 间通信通过 `sessions_send` 工具，经 Gateway 中转
- 默认关闭，需显式开启
- 路由规则预配置，确定性分发

**Clowder-ai — AgentRouter + stdout/MCP**

```
Agent A (CLI进程)
  → stdout 输出 / MCP 回调
  → AgentRouter 解析 @mention
  → spawn Agent B (CLI进程)
```

- 两种通道：stdout（公开，所有人可见）和 MCP callback（私有，点对点）
- @mention 路由由 AgentRouter 处理
- Agent 是 CLI 子进程，通过 I/O 管道通信

### 共同特征

| 特征 | 说明 |
|------|------|
| 通信经外部中介 | Agent 无法直接调用对方 |
| 消息格式类似聊天 | 自然语言 + 结构化混合 |
| 松耦合 | Agent 可以是不同模型、不同进程 |
| 扩展容易 | 加一个 Agent 只需接入频道 |

### 适用场景

- 跨模型协作（Claude + Codex + Gemini）
- 多渠道接入（需要外部消息平台）
- Agent 之间关系松散，不需要紧密协调

## 内部总线型

Agent 共享同一个运行时，通过框架内置的消息机制直接通信。

### 通信拓扑

```
Agent A ──→ SendMessage ──→ 文件邮箱 ──→ Agent B 直接读取
              ↑
         框架内置，无外部中介
```

### 代表实现：Claude Code Teams

**文件邮箱系统**

```
~/.claude/teams/{team_name}/inboxes/{agent_name}.json
```

```typescript
// 发送消息 → 写入对方邮箱文件
SendMessage({
  to: "researcher",
  summary: "found the bug",
  message: "null pointer in auth/validate.ts:42"
})

// 对方自动读取邮箱中的未读消息
```

**任务系统**

```typescript
// Lead 创建任务
TaskCreate({ subject: "Fix auth bug", description: "..." })

// Worker 认领任务
TaskUpdate({ taskId: "1", owner: "researcher", status: "in_progress" })

// Worker 完成任务
TaskUpdate({ taskId: "1", status: "completed" })
```

**消息格式**

```
// Agent 之间不仅有文本消息，还有结构化协议
{ type: "shutdown_request", reason: "Task complete" }
{ type: "shutdown_response", request_id: "...", approve: true }
{ type: "plan_approval_response", request_id: "...", approve: false }
```

### 核心特征

| 特征 | 说明 |
|------|------|
| 通信不经外部 | 框架内置邮箱 + 任务列表 |
| 同一运行时 | Agent 跑在同一进程或 tmux pane |
| 紧耦合 | Agent 共享上下文、权限、工具 |
| 低延迟 | 文件写入即送达，无网络开销 |

### 适用场景

- 同一模型的并行任务分配
- 需要紧密协作的编程任务
- 对通信效率有要求的场景

## 两类模式的本质区别

| 维度 | 共享频道型 | 内部总线型 |
|------|-----------|-----------|
| **Agent 关系** | 互不认识，通过中介传话 | 同一间办公室，直接对话 |
| **通信延迟** | 高（经外部路由/解析） | 低（文件/内存直写） |
| **耦合度** | 松（可跨模型跨进程） | 紧（共享运行时） |
| **扩展性** | 容易（新 Agent 接入频道即可） | 较难（受框架能力限制） |
| **容错** | 中介挂了全断 | 单个 Agent 挂了影响局部 |
| **跨模型** | 天然支持 | 通常限于同一模型 |

## 什么时候用哪种

```
需要跨模型协作？
  ├── 是 → 共享频道型（OpenClaw / Clowder-ai 模式）
  └── 否
      ├── 需要紧密协调？
      │   ├── 是 → 内部总线型（Claude Code Teams 模式）
      │   └── 否 → 两者皆可，选简单的
      └── 需要外部渠道接入？
          ├── 是 → 共享频道型
          └── 否 → 看团队偏好
```
