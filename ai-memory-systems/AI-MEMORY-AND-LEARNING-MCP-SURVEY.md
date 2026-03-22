# AI Agent 记忆与自学习 MCP 调查报告

**调查日期：** 2026-03-22
**搜索方法：** Brave Search MCP（通过代理包装器）
**关键词：** AI agent 记忆, MCP 服务器, 自学习, 知识图谱, MemGPT, Letta, Zep

---

## 🔍 发现的 MCP 服务器和工具

### MCP 记忆服务器

#### 1. Hindsight MCP Memory Server
- **描述：** 开源持久化记忆，通过 MCP 为 AI agents 提供结构化记忆
- **特性：**
  - 一个 Docker 命令即可本地运行完整技术栈
  - 会话间持久化记忆
  - 结构化记忆格式
- **网址：** https://hindsight.vectorize.io/
- **来源：** https://hindsight.vectorize.io/blog/2026/03/04/mcp-agent-memory

#### 2. doobidoo/mcp-memory-service
- **描述：** 为 AI agent 管道提供的开源持久化记忆
- **特性：**
  - REST API + 知识图谱 + 自主整合
  - 支持 LangGraph、CrewAI、AutoGen 和 Claude
  - 知识图谱集成
- **GitHub：** https://github.com/doobidoo/mcp-memory-service

#### 3. Mem0 OpenMemory
- **描述：** 为 coding agents 提供持久化 MCP AI 记忆层
- **特性：**
  - 支持 Cursor、VS Code、Claude 及所有 MCP 兼容的 coding agent
  - 跨编辑器支持
  - 生产就绪
- **网址：** https://mem0.ai/openmemory

#### 4. khanglvm/agent-memory
- **描述：** 使用 MCP 标准的 Obsidian vault 通用 AI 桥接器
- **特性：**
  - 符合 MCP 标准
  - Obsidian 集成
- **网址：** https://lobehub.com/mcp/khanglvm-agent-memory

---

### AI 记忆系统框架

#### 5. Zep ⭐
- **描述：** 时序知识图谱记忆系统
- **特性：**
  - 在 Deep Memory Retrieval (DMR) 基准测试中超越 MemGPT/Letta
  - 基于会话的交互存储
  - 将对话记录存储为记忆块
  - 业界领先的性能表现
- **研究论文：** https://arxiv.org/html/2603.04740
- **博客：** https://blog.getzep.com/state-of-the-art-agent-memory/
- **网址：** https://www.getzep.com/

#### 6. Letta（原名 MemGPT）⭐
- **描述：** 具有 agent 自控记忆管理功能的 AI 记忆系统
- **特性：**
  - Agents 通过工具调用主动管理自己的记忆块
  - 读取、写入和搜索档案
  - Agent 对记忆的显式控制
  - Letta Code 用于构建自定义个人 agents
- **文档：** https://docs.letta.com/
- **对比：** https://medium.com/asymptotic-spaghetti-integration/from-beta-to-battle-tested-picking-between-letta-mem0-zep-for-ai-memory-6850ca8703d1

#### 7. Mem0 ⭐
- **描述：** 面向生产环境的 AI 记忆层
- **特性：**
  - 生产就绪
  - 广泛采用
  - 多种集成选项
- **网址：** https://mem0.ai/

---

### 知识图谱构建工具

#### 8. Graphiti（by Zep）
- **描述：** 为 AI Agents 构建实时知识图谱
- **GitHub：** https://github.com/getzep/graphiti

#### 9. Neo4j + Agentic Knowledge Graph Construction
- **描述：** DeepLearning.AI 课程：使用 agents 自动化构建知识图谱
- **讲师：** Andreas Kollegger（Neo4j 开发者布道师）
- **网址：** https://learn.deeplearning.ai/courses/agentic-knowledge-graph-construction/information

#### 10. Stardog Knowledge Graph
- **描述：** 面向 Agentic AI 的知识图谱
- **特性：**
  - 可信的、上下文化的知识图谱
  - AI agents 的单一真实来源
  - 安全、可靠的 AI 自动化
- **网址：** https://www.stardog.com/agentic-ai-knowledge-graph/

#### 11. Memgraph
- **描述：** 使用 AI agents 和向量搜索构建知识图谱
- **特性：**
  - 内置向量搜索
  - 消歧知识图谱构建
- **演示：** https://memgraph.com/blog/ai-agent-vector-search-dismbiguated-knowledge-graph-demo

---

### Agent 工作流框架

#### 12. lastmile-ai/mcp-agent
- **描述：** 使用 MCP 和简单工作流模式构建高效 agents
- **特性：**
  - 增强的 LLMs，封装 provider SDKs
  - Agent 的工具、记忆和结构化输出助手
  - generate、generate_str、generate_structured 方法
- **GitHub：** https://github.com/lastmile-ai/mcp-agent

---

## 📊 对比矩阵

| 系统 | 类型 | 部署方式 | 核心特性 | 最佳适用场景 |
|------|------|---------|---------|------------|
| **Hindsight** | MCP Server | Docker | 结构化记忆 | 本地开发 |
| **MCP Memory Service** | MCP Server | REST API | 知识图谱 + 自主整合 | 多框架集成 |
| **Mem0 OpenMemory** | MCP Layer | 云端/本地 | 跨编辑器支持 | Coding agents |
| **Zep** | Memory System | 云端/本地 | 时序知识图谱，最佳 DMR 得分 | 生产环境 |
| **Letta/MemGPT** | Memory System | 本地 | Agent 自控记忆 | 研究/实验 |
| **Mem0** | Memory System | 云端 | 生产就绪 | 企业部署 |
| **Graphiti** | Knowledge Graph | 库 | 实时构建 | 动态数据 |
| **Stardog** | Knowledge Graph | 企业 | 可信的真实来源 | 企业 AI |

---

## 🔬 研究论文

1. **"Memory in the Age of AI Agents"（2025年12月）**
   - Agent 记忆领域的系统性映射
   - 统一框架提案
   - 来源：arXiv:2603.04740

2. **"Top 10 AI Memory Products 2026"（Medium，2026年2月）**
   - 对比：Mem0, Zep, Letta, Supermemory, Cognee, Anthropic Memory, Memori
   - 网址：https://medium.com/@bumurzaqov2/top-10-ai-memory-products-2026-09d7900b5ab1

3. **"AI Agent Memory Systems in 2026: Mem0, Zep, Hindsight, Memvid"（Medium，2026年3月）**
   - 记忆系统的详细对比
   - Mem0 论文发现：Zep 的记忆占用 > 600k tokens vs Mem0 的 1,764 tokens
   - 网址：https://yogeshyadav.medium.com/ai-agent-memory-systems-in-2026-mem0-zep-hindsight-memvid-and-everything-in-between-compared-96e35b818da8

---

## 🎯 后续步骤

### 与 Continuous Learning v2 的对比
- 对比记忆类型：情景记忆、语义记忆、程序记忆、工作记忆、联想记忆
- 评估 instinct 提取模式
- 评估自学习能力
- 对比整合机制

### 实施建议
- 本地测试 Hindsight MCP server
- 评估 Zep 用于生产环境
- 与现有 MCP memory server 对比
- 考虑与 continuous-learning-v2 skill 集成

---

## 📚 相关资源

### 已安装的 MCP Servers（当前系统）
- sequential-thinking - 结构化推理
- brave-search - 网络搜索（通过代理包装器）
- filesystem - 文件操作
- memory - 基础知识图谱
- git - 版本控制
- bb-browser - 浏览器自动化

### ECC Skills（共 116 个）
- continuous-learning-v2 - 基于 instinct 的学习
- instinct-status - 查看学习到的模式
- 以及其他 114 个...

---

**生成者：** Claude Code
**日期：** 2026-03-22
**方法：** Brave Search MCP（通过 undici 代理包装器）
