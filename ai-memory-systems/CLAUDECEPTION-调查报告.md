# Claudeception 调查报告

**调查日期：** 2026-03-22
**搜索方法：** Brave Search MCP + WebReader
**关键词：** Claudeception, Claude Code skill, 自主学习, 技能提取

---

## 🔍 Claudeception 是什么？

### 核心概念

**Claudeception** 是一个用于 Claude Code 的持续学习技能，能够从工作会话中自主提取可重用的知识并保存为新技能。

**命名由来：**
致敬"盗梦空间"(Inception)的递归概念 —— "AI-within-an-AI"（AI 中的 AI）

---

## 🎯 解决的问题

### 传统 AI Agent 的困境

```
第一次使用：
  遇到错误 → 调试 1 小时 → 解决问题 → 会话结束

第二次遇到同样问题：
  又要重新调试 1 小时 → 从零开始 ❌
```

### Claudeception 的解决方案

```
第一次使用：
  遇到错误 → 调试 1 小时 → 解决问题 →
  自动提取为技能 → 保存到技能库 ✅

第二次遇到同样问题：
  自动加载技能 → 立即应用解决方案 →
  节省 1 小时 ✅
```

---

## 🏗️ 工作原理

### 1. **Claude Code 技能系统**

Claude Code 启动时：
```
加载所有技能 → 读取技能描述（每个约100 tokens）
     ↓
工作过程中：
  当前上下文 → 匹配技能描述 → 拉取相关技能
```

### 2. **技能提取机制**

当 Claudeception 检测到可提取的知识时：

```
工作会话 → 检测到非显而易见的解决方案
     ↓
提取技能 → 创建 Markdown 文件
     ↓
优化描述 → 确保未来能被检索到
     ↓
保存到技能库 → 下次自动加载
```

**关键：** 描述非常重要
- ❌ "帮助解决数据库问题" → 匹配效果差
- ✅ "PrismaClientKnownRequestError 在 serverless 环境的修复方案" → 精准匹配

### 3. **质量门槛**

不是所有任务都会产生技能。只有满足以下条件才会提取：

- ✅ 需要实际探索（不是简单的文档查询）
- ✅ 对未来任务有帮助
- ✅ 有明确的触发条件
- ✅ 已经验证有效
- ✅ 6个月后仍然有用

---

## 📋 技能格式

```markdown
---
name: prisma-connection-pool-exhaustion
description: |
  Fix for PrismaClientKnownRequestError: Too many database connections
  in serverless environments (Vercel, AWS Lambda). Use when connection
  count errors appear after ~5 concurrent requests.
author: Claude Code
version: 1.0.0
date: 2024-01-15
---

# Prisma Connection Pool Exhaustion

## Problem
[这个技能解决什么问题]

## Context / Trigger Conditions
[确切的错误消息、症状、场景]

## Solution
[逐步修复步骤]

## Verification
[如何确认修复有效]
```

---

## 🚀 安装与使用

### 安装方式

**用户级安装（推荐）：**
```bash
git clone https://github.com/blader/Claudeception.git ~/.claude/skills/claudeception
```

**项目级安装：**
```bash
git clone https://github.com/blader/Claudeception.git .claude/skills/claudeception
```

### 激活方式

#### 自动模式
Claudeception 会在以下情况自动激活：
- 刚完成调试并发现了非显而易见的解决方案
- 通过调查或试错找到了变通方案
- 解决了根本原因不明显的错误
- 通过调查学习了项目特定的模式或配置
- 完成了任何需要实际探索的任务

#### 显式模式
```bash
# 触发学习回顾
/claudeception

# 显式请求技能提取
"Save what we just learned as a skill"
```

### Hook 设置（推荐）

为确保每次会话都评估可提取的知识，建议设置 hook：

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/claudeception-activator.sh"
          }
        ]
      }
    ]
  }
}
```

---

## 🔬 与其他系统的对比

### vs Continuous Learning v2

| 维度 | Claudeception | Continuous Learning v2 |
|------|--------------|----------------------|
| **学习单位** | 完整技能 | 原子化本能 (instincts) |
| **触发方式** | 语义匹配 + hook | PreToolUse/PostToolUse hooks |
| **置信度** | 无 | 0.3-0.9 加权 |
| **作用域** | 用户/项目级 | 项目/全局级 |
| **进化能力** | 直接生成技能 | 本能 → 聚类 → 技能/命令/代理 |
| **观察可靠性** | 依赖 Claude 判断 | 100% 工具调用捕获 |
| **检索方式** | Claude 原生技能匹配 | 无向量检索 |

### 共同点
- ✅ 都是从工作会话中学习
- ✅ 都是为了避免重复学习
- ✅ 都支持项目级和全局级知识

### 差异点
- **Claudeception**: 直接生成可用的技能，更快见效
- **Continuous Learning v2**: 更细粒度（本能级别），渐进式置信度，可进化

---

## 📚 学术背景

Claudeception 的设计受到以下研究启发：

1. **Voyager (Wang et al., 2023)**
   - 游戏中的 agents 可以构建可重用技能库
   - 避免重新学习已知内容

2. **CASCADE (2024)**
   - 引入"元技能"概念（获取技能的技能）
   - 这正是 Claudeception 的本质

3. **SEAgent (2025)**
   - Agents 通过试错学习新软件环境
   - 启发了回顾性特征

4. **Reflexion (Shinn et al., 2023)**
   - 自我反思有助于学习
   - Claudeception 的回顾机制

**核心洞察：** 持久化学习内容的 agents 比每次重新开始的 agents 表现更好。

---

## 💡 应用场景

### 1. **调试经验积累**
```
遇到 Prisma 连接池耗尽 → 调试 1 小时 →
提取技能 → 下次 5 秒解决 ✅
```

### 2. **项目特定模式**
```
学习项目的 API 调用规范 → 提取为技能 →
未来自动应用 ✅
```

### 3. **错误解决方案**
```
解决循环依赖问题 → 提取技能 →
遇到类似问题自动应用 ✅
```

### 4. **最佳实践固化**
```
发现某个配置最优 → 提取技能 →
成为项目标准 ✅
```

---

## 🎯 示例技能

### Next.js Server-Side Error Debugging
```markdown
---
name: nextjs-server-side-error-debugging
description: |
  Technique for debugging Next.js errors that don't appear
  in browser console. Use when client-side logs show no
  error details but server logs do.
---

# Next.js Server-Side Error Debugging

## Problem
Next.js server-side errors don't show in browser console

## Context / Trigger Conditions
- Error occurs but browser console is empty
- Server-side rendering or API route issues
- "Internal Server Error" with no details

## Solution
1. Check terminal where `npm run dev` is running
2. Look for error stack traces in server logs
3. Add console.log() in server components
4. Use server-side debugging tools

## Verification
Error details visible in server terminal
```

---

## 🔗 相关资源

### 官方资源
- **GitHub 仓库：** https://github.com/blader/Claudeception
- **FastMCP 技能库：** https://fastmcp.me/skills/details/1903/claudeception
- **Agent Skill 目录：** https://agentskill.work/en/skills/blader/Claudeception

### 相关概念
- **Claude Artifacts 的 "Claude in Claude"**
  - Claude 调用 Claude 的递归能力
  - https://claudeception.com/

- **Recursive Sub-Agent Spawning**
  - Claude Code 生成子 agents，子 agents 再生成子 agents
  - Reddit: r/ClaudeCode 讨论

---

## 🎯 总结

### 优势
✅ 自动提取可重用知识
✅ 避免重复调试和学习
✅ 与 Claude Code 原生集成
✅ 描述优化确保检索准确性
✅ 质量门槛保证技能价值

### 局限
⚠️ 依赖 Claude 判断（可能误判）
⚠️ 无置信度系统（不知道技能可靠性）
⚠️ 无向量检索（仅依赖描述匹配）
⚠️ 无时序建模（不知道技能是否过时）

### 与你当前系统的关系

**互补关系：**
- **Claudeception**: 快速提取完整技能，立即可用
- **Continuous Learning v2**: 细粒度本能，渐进式置信度，可进化

**建议：**
1. **短期：** 安装 Claudeception，快速积累技能
2. **中期：** 两个系统并行，互相补充
3. **长期：** 考虑融合两个系统的优点

---

## 🚀 行动建议

### 立即可做
```bash
# 安装 Claudeception
git clone https://github.com/blader/Claudeception.git ~/.claude/skills/claudeception

# 设置 hook（可选）
mkdir -p ~/.claude/hooks
cp ~/.claude/skills/claudeception/scripts/claudeception-activator.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/claudeception-activator.sh
```

### 与现有系统集成
1. Claudeception 提取的技能可以作为 Continuous Learning v2 的输入
2. 本能（instincts）可以聚类成 Claudeception 格式的技能
3. 两个系统可以共享同一个技能库目录

---

**生成者：** Claude Code
**日期：** 2026-03-22
**方法：** Brave Search MCP + WebReader
