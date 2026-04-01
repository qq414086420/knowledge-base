# 附录：MCP 搜索引擎代理配置踩坑指南

## 问题

Brave Search、Serper 等 MCP 搜索引擎在需要代理（如公司内网、国内网络环境）时，**不会读取 `HTTP_PROXY` / `HTTPS_PROXY` 环境变量**。直接在 `env` 中配置代理变量无效，请求仍然走直连，导致超时或连接失败。

## 原因

这些 MCP Server 使用 Node.js 内建的 `fetch`（基于 `undici`），而 `undici` 默认不尊重 `HTTP_PROXY` 环境变量。这与浏览器和 `curl` 等工具的行为不同，是一个常见的认知陷阱。

## 解决方案：Wrapper 模式

通过一个轻量 wrapper 脚本，在启动原始 MCP Server 之前，用 `undici` 的 `setGlobalDispatcher` 注入代理配置。

### 1. 创建项目目录

```bash
mkdir -p ~/.claude/mcp-wrappers
cd ~/.claude/mcp-wrappers
npm init -y
npm install @modelcontextprotocol/server-brave-search undici
```

> **注意**：`package.json` 中需要设置 `"type": "module"` 以支持 ES Module 语法。

### 2. 创建 Wrapper 脚本

`~/.claude/mcp-wrappers/brave-search-with-proxy.js`：

```javascript
#!/usr/bin/env node

import { setGlobalDispatcher, ProxyAgent } from 'undici';

// 从环境变量读取代理配置
const proxyUrl = process.env.HTTPS_PROXY || process.env.HTTP_PROXY;

if (proxyUrl) {
  console.error(`[Wrapper] Setting up proxy: ${proxyUrl}`);
  const proxyAgent = new ProxyAgent(proxyUrl);
  setGlobalDispatcher(proxyAgent);
  console.error('[Wrapper] Proxy configured');
} else {
  console.error('[Wrapper] No proxy, using direct connection');
}

// 启动原始 MCP Server
await import('@modelcontextprotocol/server-brave-search/dist/index.js');
```

### 3. 修改 Claude Code 配置

**原始配置（代理不生效）：**

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your-key",
        "HTTP_PROXY": "http://127.0.0.1:7897"
      }
    }
  }
}
```

**修改后配置（代理生效）：**

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "node",
      "args": ["/home/your-user/.claude/mcp-wrappers/brave-search-with-proxy.js"],
      "env": {
        "BRAVE_API_KEY": "your-key",
        "HTTP_PROXY": "http://127.0.0.1:7897",
        "HTTPS_PROXY": "http://127.0.0.1:7897"
      }
    }
  }
}
```

关键区别：`command` 从 `npx` 改为 `node`，`args` 指向 wrapper 脚本而非原始包。

### 4. 同理适用于 Serper

只需替换 wrapper 中的 `import` 目标：

```javascript
// 将最后一行替换为 Serper 的入口
await import('serper-mcp/dist/index.js');
```

## 目录结构

```
~/.claude/mcp-wrappers/
├── package.json              # type: "module", 依赖 undici + 原始 MCP 包
├── brave-search-with-proxy.js
└── serper-with-proxy.js      # 同理
```

## 总结

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 环境变量代理不生效 | Node.js undici 不读 `HTTP_PROXY` | Wrapper 注入 `setGlobalDispatcher` |
| 修改原始包代码 | 不优雅，更新丢改动 | Wrapper 模式零侵入 |
| 多个 MCP 都有此问题 | 通用问题 | 统一用 Wrapper 模式 |
