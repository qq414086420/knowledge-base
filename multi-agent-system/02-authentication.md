# 认证方式：OAuth vs API Key，SDK vs CLI 的选择

## 两种认证体系

AI 服务商通常维护**两套独立的认证和计费体系**：

| | API Key | OAuth（订阅） |
|---|---|---|
| 认证方式 | `sk-ant-xxx` 静态密钥 | OAuth token（登录后自动获取） |
| 计费方式 | 按 token 预付费/后付费 | 月费 + 用量上限 |
| 典型入口 | `api.anthropic.com` | CLI 登录（`claude login`） |
| 代表 SDK | `@anthropic-ai/sdk` | `@anthropic-ai/claude-code` |

**关键点**：两套体系互不相通。有 Max 订阅不等于有 API 额度，反过来也一样。

## OAuth 简述

OAuth（Open Authorization）是一种授权协议，让用户在不必交出密码的情况下授权第三方访问资源。

在 AI CLI 场景中的流程：

```
1. 用户执行 `claude login`
2. 浏览器弹出登录页，用户输入账号密码
3. 登录成功，Anthropic 返回 OAuth token
4. CLI 将 token 存储在本地（如 ~/.claude/）
5. 之后每次调用 `claude -p "..."`，CLI 自动携带 token
6. Anthropic 认出 token → 走订阅额度扣费
```

用 OAuth 不需要配置任何 API Key。`claude` CLI 就是这样工作的——登录一次，之后直接使用。

## SDK vs CLI 之争

### cat-cafe-tutorials Lesson 01 的故事

Cat Café 项目最初用 SDK 开发，36 个测试全过，后来发现：

| SDK | 认证方式 | 能用订阅吗？ |
|-----|---------|------------|
| `@anthropic-ai/claude-agent-sdk` | API Key only | 原教程声称不能 |
| `@openai/codex-sdk` | API Key only | 原教程声称不能 |
| `@google/generative-ai` | API Key only | 原教程声称不能 |

最终项目从 SDK 迁移到 CLI 子进程模式。

### 教程说法的修正

原教程说"SDK 不能用订阅"——这个表述**过于简化**了。

更准确的说法是：

1. **原始 API SDK**（`@anthropic-ai/sdk`）确实只走 API Key 计费，与订阅是独立的账户体系
2. 但 `@anthropic-ai/claude-code` SDK **支持 OAuth**，能走订阅——`claude` CLI 本身就是基于它构建的
3. Gemini 确实没有 Agent SDK，这一点是成立的

### CLI 子进程方案仍然合理的真正原因

不是因为"SDK 技术上不可能用订阅"，而是：

| 考量 | 分析 |
|------|------|
| **统一性** | 三家 CLI 接口差异小（都是 NDJSON），SDK 差异大 |
| **Gemini 无 Agent SDK** | 只能走 CLI，不如统一架构 |
| **完整能力** | CLI 天然自带工具、权限、Session 管理 |
| **维护成本** | 不用跟踪各家 SDK 的 breaking changes |

## 实践建议

| 场景 | 推荐方式 |
|------|---------|
| 有订阅，需要 Agent 能力 | CLI 子进程 |
| 有 API 额度，简单聊天 | API SDK |
| 有订阅，需要深度集成 | `@anthropic-ai/claude-code` SDK |
| 多模型统一接入 | CLI 子进程（最统一） |
