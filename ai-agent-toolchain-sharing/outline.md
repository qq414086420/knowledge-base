# AI 编码 Agent 工具链搭建

> 30 分钟分享 | 面向程序员

---

## 一、开场引入（3 min）

- **裸 Agent vs 工具链 Agent 的差距演示**
  - 同一个任务，裸跑 vs 配置工具链后的效果对比
- **核心观点**
  - Agent 的能力 = 模型能力 × 工具链配置质量
- **本场覆盖 6 个类别**，按实用优先级排列

---

## 二、Search & Knowledge Access — 搜索与知识获取（4.5 min）

**痛点**：Agent 知识有截止日期，对最新 API/框架版本会"幻觉"，大多数用户根本没给 Agent 配搜索

### 1. 为什么搜索是第一优先级

- **段子**：程序员没有 Google 就无法工作，Agent 没有搜索引擎，就像程序员没有 Google
- 没有搜索 → Agent 编造不存在的 API
- 有了搜索 → 始终基于最新信息回答
- 现场对比演示：问一个最新框架的问题

### 2. 三个搜索引擎对比

| 搜索工具 | 底层 | 特点 | 门槛 |
|----------|------|------|------|
| **Serper** | Google SERP | 每月免费 2500 次，Google 搜索结果 | 最低，注册即用 |
| **Brave Search** | 独立索引 | 隐私友好，不依赖 Google | 需绑信用卡 |
| **Grok Search** | xAI 实时搜索 + X/Twitter | 实时社区动态 + 语义分析 | 依赖 xAI API key |

- 推荐：入门用 Serper（免费额度够用），进阶加 Brave 或 Grok

### 3. 配置示例

以 Serper 为例（settings.json）：

```json
{
  "mcpServers": {
    "serper": {
      "command": "npx",
      "args": ["-y", "serper-mcp"],
      "env": {
        "SERPER_API_KEY": "your-api-key"
      }
    }
  }
}
```

### 4. 其他搜索能力（一嘴带过）

- **Context7**：按版本注入库文档，解决 API 幻觉
- **WebReader**：读取网页内容，配合搜索使用
- **内建 WebSearch**：无需配置即可使用

---

## 三、Deep Thinking — 深度思考（4 min）

**痛点**：Agent 遇到复杂问题直接给错误答案，缺乏推理过程

### 1. Sequential Thinking MCP

- 让模型"分步思考"而非直接给答案，思考过程可见、可审查
- 工作原理：模型自主决定思考轮数（3-10轮不等）和每轮内容，MCP 提供框架但思考内容由模型自行生成
- 配置示例（settings.json）：

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

- **实际效果对比**：修复一个并发 bug

  **无 Sequential Thinking**：Agent 直接给出一个方案

  ```
  Agent：在扣款方法上加个 synchronized 就行了
  ```

  **有 Sequential Thinking**：Agent 先经过多轮思考再回答

  ```
  Thought 1/5：分析并发扣款重复的根因...
  Thought 2/5：考虑加锁方案的利弊...
  Thought 3/5：评估幂等 token 方案...
  （模型自主决定每轮内容，总轮数也由模型判断）
  → 最终给出更全面的方案
  ```

  核心价值：不是保证答案一定对，而是**把推理过程外显化**，方便审查和纠偏

### 2. Extended Thinking（Claude Code 内建）

- CC 内建能力，不确定其他 Agent 是否具备，可作为 CC 优势的一个小例子
- 按 `Tab` 键切换开/关，`~/.claude/settings.json` 中 `alwaysThinkingEnabled` 永久开启
- 开启后 Claude 在回答前自动进行内部推理（adaptive thinking，按问题复杂度自动调节深度）
- 与 Sequential Thinking 互补：Extended Thinking = 想得更深，Sequential Thinking = 按步骤想、能回退

### 3. 什么时候该用

- **需要**：架构设计、复杂 bug 排查、多文件重构
- **不需要**：简单 bug 修复、单文件改动

---

## 四、Self-Learning — 自主学习（4 min）

**痛点**：Agent 不会从错误中学习，花一小时排查的 bug，下次遇到又是一小时

### 1. Claudeception（轻量入门）

- 最简单的自主学习 skill：发现知识 → 保存为 skill → 未来自动加载
- 工作原理：
  - Claude Code 在工作中发现了非显而易见的知识（调试技巧、workaround、项目特定模式）
  - 自动提取为一个新的 skill markdown 文件（带 YAML frontmatter）
  - 未来遇到类似问题时，通过语义匹配自动加载
- 安装方式：`git clone https://github.com/blader/Claudeception.git ~/.claude/skills/claudeception`
- 推荐配合 UserPromptSubmit hook 使用，每次提交 prompt 时检查是否有可提取的知识

### 2. Continuous Learning v1 + v2（进阶）

- **v1（手动模式）**：
  - 会话结束时通过 Stop hook 评估，或手动 `/learn` 触发
  - 提取模式保存为 skill 文件到 `~/.claude/skills/learned/`
  - 简单可靠，适合受限环境（内网、本地 LLM）
  - **注意**：`/learn` 提取的知识只输出为对话文本，不会自动保存为 `SKILL.md` 格式
    - Claude Code 的 skill 发现机制要求 `<name>/SKILL.md` 目录结构 + YAML frontmatter
    - 裸 `.md` 文件放在 `learned/` 下无法被自动发现和触发
    - 需要手动将提取的知识整理为正确的 skill 目录结构，或修改 `/learn` 命令模板使其自动保存

- **v2（自动模式）**：
  - PreToolUse/PostToolUse hooks 自动捕获每次工具调用（100% 可靠）
  - 提取为原子化的 "instincts"（带置信度评分 0.3-0.9）
  - 进化管道：observations → instincts → cluster → skills/commands/agents
  - 项目作用域隔离：React 项目的模式不会污染 Python 项目

- **实际踩坑**：
  - v2 的 observe.sh 在公司内网环境可能无法正常运行（本地 LLM 限制）
  - 此时用 v1 的手动 `/learn` 机制作为降级方案
  - 但 v2 的晋升机制（project → global）是良好实践，值得借鉴

### 3. 自主学习的三个核心问题

| 问题 | 说明 | 现状 |
|------|------|------|
| **积累** | 如何从会话中提取知识 | Claudeception / CL v1 v2 已解决 |
| **晋升** | 项目级知识何时升级为全局知识 | CL v2 的 project → global 晋升机制 |
| **淘汰** | 过时/错误的知识如何清理和替换 | **当前生态均未解决**，是行业性空白 |

- **淘汰是行业性空白**：调研了主流自主学习方案，均无完善的知识淘汰机制
  - CL v2 有置信度衰减概念（长期未触发 → 降权），但未实现自动清理
  - claude-reflect-system 有 30 天备份自动清理，但清理的是备份文件而非知识本身
  - 学术界有探索：MemOS 论文提出 Hot/Warm/Cold/Expired 分级，但未工程化实现
  - **Clowder AI（F100 Self-Evolution）** 是我见过的一个比较成熟的实现:
    - 五级知识成熟度阶梯：L0 Episode → L1 Pattern → L2 Draft → L3 Validated → L4 Standard
    - **降级机制**：L2 最近 3 次 <50% → 退回 L1；L3 最近 5 次 <60% → 退回 L2；L4 高风险越界 → freeze
    - **过期防护**：`superseded_by` 字段 + `supersedes/invalidates` 关系，过时高相似决策比查不到更危险
    - 三模式闭环：Mode A（Scope Guard 防御）+ Mode B（Process Evolution 改进）+ Mode C（Knowledge Evolution 成长）
    - 项目地址：https://github.com/zts212653/clowder-ai
- **理想淘汰策略**：置信度衰减（长期未被触发 → 降权）+ 矛盾检测（新知识与旧知识冲突 → 替换）
- 这是当前 Agent 自主学习体系的最大短板：**知识只会积累不会淘汰**

### 4. 三个工具对比

| | Claudeception | CL v1 | CL v2 |
|---|---|---|---|
| **粒度** | skill 级别 | skill 级别 | instinct 级别（细粒度） |
| **触发** | hook 或手动 | Stop hook / `/learn` | 自动 hooks |
| **存储** | markdown 文件 | markdown 文件 | 结构化 yaml + 置信度 |
| **晋升** | 无 | 无 | project → global |
| **淘汰** | 无 | 无 | 置信度衰减（未实现自动清理） |
| **适合** | 入门 | 受限环境降级 | 理想环境进阶 |

- *穿插提及：浏览器控制（bb-browser）可作为自主学习的信息来源之一*

---

## 五、Memory — 记忆持久化（4 min）

**痛点**：每次会话从零开始，之前学到的项目知识全部丢失

> **与自主学习的关联**：Self-Learning 解决"怎么学"，Memory 解决"学了的东西存哪"
> **与 Skill 的区别**：Skill 有 Agent 提供的触发机制（语义匹配、slash command），会自动在合适的时机加载；Memory 不一定是被动存储，但无论存取方式如何，**都需要检索和索引机制**来保证知识在需要时能被找到，否则存了也等于没存

### 1. Memory MCP（Knowledge Graph）

- 最直观的记忆方案：实体 → 关系 → 观察，类似人脑的联想记忆
- 适合存储结构化知识：架构决策、团队约定、API 规则
- 配置示例：

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-memory"]
    }
  }
}
```

- 局限：图谱规模大后检索效率下降，没有自动过期/淘汰机制

### 2. 记忆系统全景对比

| 层级 | 方案 | 核心机制 | 检索方式 | 适合 |
|------|------|---------|---------|------|
| **内建** | Auto Memory | 文件 + MEMORY.md 索引 | 索引文件自动加载 | 用户偏好、项目上下文 |
| **内建** | Auto Dream | 记忆整理（类 REM 睡眠） | 自动触发（24h + 5 会话） | 记忆去重与纠偏 |
| **MCP** | Memory MCP | 知识图谱（实体/关系/观察） | 图遍历查询 | 结构化知识 |
| **MCP** | Hindsight | Docker 本地部署，结构化记忆 | REST API | 本地开发 |
| **MCP** | Mem0 OpenMemory | 跨编辑器持久化记忆层 | 语义检索 | Coding agents |
| **框架** | Letta (MemGPT) | Agent 自控记忆块，主动读写 | Agent 自主决定读写 | 研究/实验 |
| **框架** | Zep | 时序知识图谱，DMR 基准最优 | 混合检索（图谱+向量） | 生产环境 |
| **Skill** | Pro Workflow | SQLite 纠错记录 + SessionStart hook | 启动时全量加载 | 纠错记忆 |

- **关键差异在检索**：内建/MCP 方案检索能力有限，框架级方案（Zep/Letta）有专门的检索引擎
- **Agent 自控 vs 被动存储**：Letta 让 Agent 自己决定什么该记、什么该忘，是最接近人类记忆的模型
- **Auto Dream（记忆整理）**：Claude Code 内建的自动记忆整理机制，类似人脑的 REM 睡眠。触发条件：距上次整理 >24h 且累计 ≥5 个会话。四阶段流程：① Orientation（定位现有记忆）→ ② Gather Signal（收集新信息）→ ③ Consolidation（合并重复、转换相对日期为绝对日期、删除已被推翻的事实）→ ④ Prune & Index（修剪至 MEMORY.md ≤200 行）。核心价值：解决"记忆只增不减"的问题，是少数具备自动淘汰能力的内建方案

### 3. 选一个主记忆，不要"全家桶"

- **记忆与其他工具不同**：搜索、思考类工具是无状态的，多装一个只是多一个信息源；记忆是有状态的，多装一个是多一个需要维护一致性的副本
- **记忆冲突的代价**：两个系统存了同一条知识，更新时只改了一边 → Agent 拿到矛盾信息 → 比"没记忆"更糟
- **推荐策略：选一个主方案，按域隔离**
  - **Auto Memory（内建，零配置）**：个人偏好、协作模式、项目上下文 → 适合大多数人的默认选择
  - **Memory MCP**：结构化关系型知识（架构决策、团队约定） → 需要图查询场景的主选择
  - **原则**：同一个域的知识只存一个地方，不要交叉存储
- **Auto Dream 的价值**：内建的自动整理机制，解决"记忆只增不减"问题（去重、纠偏、修剪）
- 自主学习（上一章节）的观察结果 → 统一写入选定的主记忆 → 跨会话可用

---

## 六、Hooks & Automation — 钩子与自动化（4 min）

**痛点**：每次都要手动提醒 Agent 做安全检查、跑测试、格式化

### 1. 四类 Hook

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| PreToolUse | 工具执行前 | 参数校验、拦截敏感操作 |
| PostToolUse | 工具执行后 | 自动格式化、lint |
| Stop | 会话结束时 | 验证测试通过、总结变更 |
| SessionStart | 会话启动时 | 注入上下文、环境检测 |

### 2. 实用配置示例

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "npx prettier --write $FILE_PATH"
      }
    ],
    "Stop": [
      {
        "command": "npm test"
      }
    ]
  }
}
```

### 3. 进阶：IM 通知（人不在工位也能协作）

> **场景**：让 Agent 跑长任务，人离开工位。Agent 需要你时主动通知你回来。

**方案一：AskUserQuestion 匹配（精准）**

Agent 遇到需要用户决策的问题时，hook 拦截并发 IM：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "AskUserQuestion",
        "command": "curl -s 'https://your-webhook-url' -d '{\"text\": \"Agent 需要你的输入，请回来看\"}'"
      }
    ]
  }
}
```

**方案二：Notification hook（更通用）**

Claude Code 提供 `Notification` hook，在 Agent 完成任务或等待输入时触发，覆盖面比方案一更广：

```json
{
  "hooks": {
    "Notification": [
      {
        "command": "curl -s 'https://your-webhook-url' -d '{\"text\": \"Agent 任务完成或需要你注意\"}'"
      }
    ]
  }
}
```

**方案三：按运行时长通知（变通实现）**

Hook 系统没有原生"运行 N 分钟后触发"，但可以用 SessionStart + PostToolUse 组合实现：

```bash
#!/bin/bash
# notify-long-running.sh
START_FILE="/tmp/cc-session-start"
[[ ! -f "$START_FILE" ]] && date +%s > "$START_FILE"
ELAPSED=$(( $(date +%s) - $(cat "$START_FILE") ))
if (( ELAPSED > 300 )); then  # 超过 5 分钟
  curl -s 'https://your-webhook-url' -d '{"text": "Agent 已运行超过 5 分钟"}'
  rm "$START_FILE"  # 只通知一次
fi
```

```json
{
  "hooks": {
    "SessionStart": [{ "command": "rm -f /tmp/cc-session-start" }],
    "PostToolUse": [{ "command": "bash notify-long-running.sh" }]
  }
}
```

- 代价：每次工具调用都执行一次脚本（开销小，但略脏）

### 4. 从工具到工作流

- 单个 Agent + Hooks = 自动化工作流
- 不需要多 Agent 也能实现持续集成级别的保障
- IM 通知让"人离开、Agent 继续跑"成为可能

---

## 七、Context Engineering — 上下文工程（4.5 min）

**痛点**：装了一堆 MCP 工具，Agent 反而变蠢了？

> **转折点**：前面讲了 5 类工具怎么"加"，现在讲加多了的副作用和管理方法

### 1. Context 污染问题

- 每个 MCP 工具都占用 context window
- "dumb zone"：context 使用超 50% 后推理质量明显下降
- 工具越多 ≠ Agent 越强

### 2. 诊断与管理

- **`/context` 命令**（内建，v1.0.86+）：查看 context window 使用明细
  - 显示各组件 token 占比：system prompt、MCP tools、memory files、custom agents、messages
  - v2.1.74+ 提供可操作的优化建议（哪个 MCP 占太多、CLAUDE.md 是否过大等）
  - 配合 `@server-name disable` 动态开关 MCP，释放 context 空间
- **`/context-budget` 命令**（ECC skill）：深度审计 context 预算
  - 四阶段流程：Inventory（扫描所有组件估算 token）→ Classify（按必要性分级）→ Detect Issues（发现冗余/膨胀）→ Report（输出优化报告）
  - 覆盖范围：agents、skills、rules、MCP servers、CLAUDE.md
  - 核心洞察：**MCP 是最大的 context 杀手**——每个 tool schema 约 500 tokens，一个 30 工具的 MCP server 比所有 skills 加起来还大
  - 加 `--verbose` 查看逐文件明细
- **Context 预算控制**（环境变量）：
  - `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`：auto-compact 触发阈值（默认 ~95%，设更低如 50 可更早压缩）
  - `CLAUDE_CODE_AUTO_COMPACT_WINDOW`：有效 context 大小（1M 模型可设为 500K）
- **最佳实践**：
  - CLAUDE.md 三层结构分层注入，Rules 按 `paths` 前端按需加载，Skills 按需加载
  - 完成子任务后手动 `/compact`，避免 auto-compact 在任务中途打断

---

## 八、总结与 Q&A（2 min）

- **一句话总结**：搜索打底、思考加深、学习进化、记忆持久、自动化保障、上下文管理
- **配置模板分享**：提供完整 settings.json 模板供直接使用
- Q&A

---

> 总计：3 + 4.5 + 4 + 4 + 4 + 4 + 4.5 + 2 = **30 min**
