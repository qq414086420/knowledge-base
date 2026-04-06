# 架构模式：集中式、去中心化、层级制

## 三种典型架构

### 1. 星型中心化（Hub-and-Spoke）

代表：**OpenClaw**

```
         ┌──────────┐
WhatsApp ─┤          │
Telegram ─┤ Gateway  ├─ Agent A
Slack ────┤ (中心路由)├─ Agent B
Discord ──┤          ├─ Agent C
iMessage ─┤          │
         └──────────┘
```

所有消息必须经过中心 Gateway 路由。Agent 之间不直接通信，由 Gateway 根据绑定规则（bindings）分发。

**特点**：
- 路由规则预配置，确定性分发
- Gateway 是单点控制，也是单点故障
- 扩展渠道容易（加个 adapter），扩展 Agent 间协作难

### 2. P2P 去中心化（Peer-to-Peer）

代表：**Clowder-ai (Cat Café)**

```
  Claude ◄────@────► Codex
    │                    │
    │        Gemini      │
    └──────@◄────────────┘
```

没有中心节点。任何 Agent 可以直接 @ 任何其他 Agent。通过 AgentRouter 做消息转发，但 Router 不做决策——决策权在 Agent 自己。

**特点**：
- Agent 平等，无 Boss
- 强制跨模型 review（Codex 写的代码必须 Claude 审）
- 通信经 AgentRouter 中转，但逻辑上是点对点

### 3. 层级制（Lead + Workers）

代表：**Claude Code Teams**

```
         ┌──────────┐
  用户 ──┤ Team Lead├── Worker A
         │ (协调者)  ├── Worker B
         └──────────┘── Worker C
```

Team Lead 负责任务分配和结果综合，Worker 负责执行。Lead 通过 TaskCreate 分配任务，Worker 通过 TaskUpdate 汇报进度。

**特点**：
- Lead 是决策中心，Worker 是执行者
- Worker 之间可通过 SendMessage 直接通信（但实践中很少）
- 有两种运行模式：临时 Worker（coordinator mode）和持久 Teammate（team mode）

## 如何选择

| 场景 | 推荐架构 | 原因 |
|------|---------|------|
| 多渠道 AI 助手 | 星型中心化 | 需要统一接入各种消息平台 |
| 跨模型协作编程 | P2P 去中心化 | 需要不同模型的交叉审查 |
| 单模型并行任务 | 层级制 | 高效分配，低通信开销 |
| 需要强一致性 | 星型或层级制 | 有中心节点容易做状态同步 |

## 架构不是非此即彼

实际项目中可以混合使用。例如 Claude Code Teams 有两层：
- **Coordinator mode**：临时 Worker（层级制，用完即弃）
- **Team mode**：持久 Teammate（更接近 P2P，但仍有 Lead 角色）
