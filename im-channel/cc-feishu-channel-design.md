# Claude Code 飞书 Channel 插件设计文档

## 目标

为 Claude Code 编写一个飞书 Channel 插件，让 CC 直接通过飞书收发消息，保留完整的 skills、MCP tools 和原生能力，不经过 OpenClaw ACP 中间层。

## 背景

当前安延（编码 agent）通过 OpenClaw 的 ACP（acpx）桥接 Claude Code，存在以下问题：
- OpenClaw 注入自己的 system prompt，覆盖/干扰 CC 原生 prompt
- CC 自装的 skills 和 MCP tools 对 OpenClaw 不可见
- ACP 模式下 CC 的 `/skills`、`/plugin` 等斜杠命令不可用

通过 CC Channel 插件直接对接飞书，CC 以完整原生模式运行。

## 架构

```
飞书服务器 ←— WebSocket —→ 飞书 SDK (WSClient)
                                  ↕ (MCP Server, stdio)
                            Claude Code (完整原生模式)
```

CC 以 `--channels` 模式启动，插件作为 MCP Server 子进程运行，通过 stdio 与 CC 通信。

## 技术方案

### 1. 插件类型

Claude Code Channel 插件，基于 MCP Server 协议。

### 2. 消息接收：飞书 WSClient

使用飞书官方 Node SDK 的 `WSClient`（WebSocket 长连接），不需要公网地址或穿透：

```typescript
import * as Lark from '@larksuiteoapi/node-sdk';

const wsClient = new Lark.WSClient({
  appId: process.env.FEISHU_APP_ID,
  appSecret: process.env.FEISHU_APP_SECRET,
});

wsClient.start({
  eventDispatcher: new Lark.EventDispatcher({}).register({
    'im.message.receive_v1': async (data) => {
      // 将消息推送给 CC
    }
  })
});
```

### 3. 消息推送给 CC

通过 MCP notification 将飞书消息推入 CC 会话：

```typescript
await mcp.notification({
  method: 'notifications/claude/channel',
  params: {
    content: messageText,
    meta: {
      source: 'feishu',
      senderId: data.sender.sender_id.user_id,
      chatId: data.message.chat_id,
      messageId: data.message.message_id,
      msgType: data.message.message_type,
    }
  }
});
```

### 4. CC 回复飞书

暴露一个 MCP Tool，让 CC 调用来回复消息：

```typescript
mcp.tool('reply_feishu', '回复飞书消息', {
  chatId: z.string().describe('飞书聊天 ID'),
  text: z.string().describe('回复文本内容'),
  msgType: z.enum(['text', 'post']).optional().default('text').describe('消息类型'),
}, async ({ chatId, text, msgType }) => {
  const res = await client.im.message.create({
    params: { receive_id_type: 'chat_id' },
    data: {
      receive_id: chatId,
      msg_type: msgType,
      content: msgType === 'text' 
        ? JSON.stringify({ text }) 
        : JSON.stringify({ title: '', content: [{ tag: 'text', text }] }),
    },
  });
  return { status: 'sent', messageId: res.data?.message_id };
});
```

### 5. 发送者白名单

防止未授权用户通过 bot 给 CC 注入 prompt：

```typescript
const ALLOWED_SENDERS = process.env.FEISHU_ALLOWED_SENDERS?.split(',') || [];

// 在消息处理中校验
if (ALLOWED_SENDERS.length > 0 && !ALLOWED_SENDERS.includes(senderId)) {
  return; // 忽略未授权消息
}
```

### 6. System Prompt（instructions）

告诉 CC 如何处理飞书事件：

```
你是安延，一个编码助手。飞书消息以 <channel source="feishu"> 格式到达。
收到消息后直接回复。使用 reply_feishu 工具发送回复。
如果消息包含编码任务，直接使用你的工具（文件读写、命令执行等）完成任务后回复结果。
```

## 配置

### 环境变量（~/.claude/channels/feishu/.env）

```env
FEISHU_APP_ID=cli_xxxxx
FEISHU_APP_SECRET=xxxxx
FEISHU_ALLOWED_SENDERS=ou_xxxxx,ou_yyyyy
```

### MCP 配置（~/.claude.json 或项目 .mcp.json）

```json
{
  "mcpServers": {
    "feishu": {
      "command": "bun",
      "args": ["path/to/feishu-channel/index.ts"]
    }
  }
}
```

### 启动

```bash
claude --dangerously-load-development-channels server:feishu
```

> 注意：研究预览期间自定义 channel 需要加 `--dangerously-load-development-channels` 参数。

## 项目结构

```
feishu-channel/
├── index.ts          # 入口：MCP Server + WSClient 启动
├── feishu-client.ts  # 飞书 API 封装（发消息、解析消息等）
├── package.json
└── tsconfig.json
```

## 依赖

- `@modelcontextprotocol/sdk` — MCP Server SDK
- `@larksuiteoapi/node-sdk` — 飞书官方 Node SDK（含 WSClient）
- `zod` — schema 校验
- `bun` 或 `node` 运行时

## 消息格式处理

飞书消息的 `content` 字段是 JSON 字符串，需要解析：

| msg_type | content 格式 | 解析后 |
|---|---|---|
| text | `{"text":"hello"}` | 纯文本 |
| post | 富文本 JSON | 需要提取纯文本（遍历 content 数组取 text 节点） |
| image | `{"image_key":"xxx"}` | 需要调用飞书 API 下载图片，转 base64 传给 CC |

## 注意事项

1. **研究预览限制**：自定义 channel 目前需要 `--dangerously-load-development-channels` 参数启动
2. **claude.ai 登录**：Channel 功能要求使用 claude.ai 登录，不支持 API key 认证。但用户当前通过 claude-code-router 代理，使用的是 API key 方式，**可能需要确认兼容性**
3. **会话持久化**：CC session 关闭后消息不会投递，需要保持 CC 长期运行（如 tmux、systemd service）
4. **图片消息**：需要额外调用飞书 API 下载图片文件，暂时可以先只支持文本消息
