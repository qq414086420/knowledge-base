# LLM 编译式知识库搭建：从 Karpathy 的 Idea 到落地实践

> 基于 Karpathy 的 "LLM as Knowledge Compiler" 理念，调研并搭建个人知识库系统的完整记录。

## 一、问题：AI Agent 的知识管理痛点

使用 Claude Code 等 AI Agent 做日常开发时，遇到几个知识管理问题：

1. **知识不可持久** — Agent 的上下文窗口有限，之前调研的内容下次会话全忘了
2. **资料碎片化** — 调研文档散落在各处（docs repo、Obsidian、浏览器书签），没有统一管理
3. **关联缺失** — 不同主题的知识之间可能有联系，但没人帮忙建立
4. **人肉维护成本高** — 传统笔记系统（Notion、Obsidian）需要手动整理、分类、打标签

核心矛盾：**Agent 能理解内容但记不住，笔记系统能存储但不理解内容。**

## 二、思路来源：Karpathy 的 "Compiler Model"

Andrej Karpathy（OpenAI 联合创始人、前 Tesla AI 总监）在 2025 年提出了一个想法：

> 原始资料是不可变的源代码，LLM 是编译器，Wiki 文章是编译产物。一次编译，增量维护。

关键点：

- **Raw 不可变** — 源文件一旦摄入就不再修改，所有综合分析在另一层
- **Agent 是唯一作者** — 人只负责喂原始资料，Agent 负责编译、链接、维护
- **编译而非检索** — 不是每次查询都重新推导，而是维护一个持续更新的知识库
- **工具无关** — Karpathy 的 [gist](https://gist.github.com/karpathy/0e07921f03ab9a0a90e1efd6b26539ed) 只是一个 idea file，设计为复制粘贴给任何 LLM Agent

这个思路在社区获得了 5000+ stars 和 3255+ forks，但 Karpathy 本人没有实现任何具体工具。

## 三、方案选型：两个社区实现对比

基于 Karpathy 的理念，社区主要有两个实现：

### 3.1 rvk7895/llm-knowledge-bases

- Claude Code 专用 plugin
- Hash 增量编译（只重处理变更的源文件）
- Auto-evolve（查询时后台 Agent 自动补充 wiki）
- Opus/Sonnet/Haiku 三层模型分工
- 7 项 Lint 检查（断链、孤立文件、过期文章、矛盾信息等）

### 3.2 nvk/llm-wiki

- 可移植到任何 LLM（通过 AGENTS.md）
- Hub + Topic 隔离架构
- 双重链接策略（同时支持 Obsidian 和 Agent）
- 多角色并行研究（5-10 个 Agent 从不同角度调研）
- 论点驱动研究（正反方证据 → 结论）

### 3.3 对比与选择

| 维度 | rvk7895 | nvk/llm-wiki |
|------|---------|-------------|
| 链接策略 | 纯 `[[wikilinks]]` | **双重链接**（wikilink + markdown） |
| 增量编译 | Hash 基准 | 无 hash 追踪 |
| 被动增长 | Auto-evolve | 无 |
| 可移植性 | 仅 Claude Code | AGENTS.md 通用 |
| Lint | 7 项并行检查 | Structural Guardian |

**选择 nvk/llm-wiki**：双重链接策略适配多工具场景（Claude Code + Obsidian + GitHub 浏览），AGENTS.md 可移植设计降低供应商锁定风险。

## 四、架构设计：Hub + Topic

### 4.1 为什么不用 PARA

PARA（Projects/Areas/Resources/Archive）是给人设计的文件柜：

```
knowledge-base/
├── 00_Inbox/        # 时效驱动
├── 01_Projects/     # "正在用" vs "以后用"
├── 02_Areas/
├── 03_Resources/
└── 04_Archive/
```

Hub + Topic 是给 Agent 设计的数据库：

```
/mnt/i/wiki/                    # Hub — 注册中心
└── topics/
    ├── im-channel/             # 隔离主题
    │   ├── raw/                # 原始资料（不可变）
    │   ├── wiki/               # Agent 编译的文章
    │   ├── output/             # 生成的报告
    │   └── inbox/              # 快速投入区
    └── cc-tooling/
```

核心区别：PARA 按时效分类，Hub+Topic 按主题领域隔离。查询 im-channel 时不会搜到 cc-tooling 的内容。

### 4.2 双重链接策略

每个交叉引用同时写两种格式：

```markdown
[[gut-brain-axis|Gut-Brain Axis]] ([Gut-Brain Axis](../concepts/gut-brain-axis.md))
```

| 格式 | 谁识别 |
|------|--------|
| `[[wikilink]]` | Obsidian（图谱、反向链接） |
| `[text](path.md)` | Claude Code、GitHub、任何 Markdown 查看器 |

这解决了知识库在不同工具间互操作的核心矛盾。

### 4.3 跨 Topic 关联

Topic 之间默认隔离，通过 Hub 层做有限关联：

- 查询时 Agent 会 peek 其他 Topic 的 `_index.md` 发现相关内容
- 发现后会提示但不自动合并（保证查询不串噪音）
- 强关联可手动写跨 topic 链接

## 五、安装与配置

### 5.1 安装

```bash
claude plugin install github:nvk/llm-wiki
```

### 5.2 Hub 路径配置

默认 hub 在 `~/wiki/`。由于需要 Windows 侧 Obsidian 也能访问，将 hub 迁移到 NTFS 盘：

```bash
# 移动 hub 到 I: 盘
mv ~/wiki /mnt/i/wiki

# 配置自定义路径
mkdir -p ~/.config/llm-wiki
echo '{"hub_path": "/mnt/i/wiki"}' > ~/.config/llm-wiki/config.json
```

**关键点**：放在 NTFS 盘（I:）上，WSL2 通过 `/mnt/i/wiki/` 访问，Windows 通过 `I:\wiki\` 访问。Obsidian 直接打开 `I:\wiki\topics\<name>\` 作为独立 vault。

### 5.3 双轨运行策略

迁移期间保持两套系统并行，通过 rules 文件切换：

```markdown
# ~/.claude/rules/common/knowledge-system.md

## 双轨运行期
| 系统 | 路径 | 角色 |
|------|------|------|
| llm-wiki | /mnt/i/wiki/ | 主系统 |
| knowledge-base | /mnt/i/knowledge-base/ | 遗留 |

## 规则
- 知识操作默认使用 llm-wiki
- 禁止同时往两个系统写入相同内容
```

稳定后废弃遗留系统。

## 六、使用方式

### 6.1 自动创建 Topic 并研究

```bash
/wiki:research "Rust 异步编程" --new-topic
```

自动完成：创建目录结构 → 启动 5 个并行 Agent（学术/技术/应用/新闻/反方角度）→ 搜索网页 → 摄入资料 → 编译成 wiki 文章。

加 `--deep` 用 8 个 Agent，`--min-time 1h` 持续研究一小时。

### 6.2 论点驱动研究

```bash
/wiki:thesis "间歇性禁食通过糖淋巴系统上调减少神经炎症"
```

拆解论点 → 分配支持方/反方/机制分析 Agent → 汇总证据 → 给出结论（supported / contradicted / mixed / insufficient）。

### 6.3 日常操作

```bash
/wiki:ingest https://example.com/article   # 摄入资料
/wiki:compile                               # 编译为文章
/wiki:query "如何配置飞书 Webhook？"          # 查询
/wiki:lint --fix                            # 健康检查
/wiki:output report                         # 生成报告
```

### 6.4 Obsidian 查看

在 Obsidian 中 "Open folder as vault" 指向 `I:\wiki\topics\<name>\`，每个 Topic 是独立 vault，图谱和反向链接正常工作。

## 七、实际效果

### 7.1 已迁移内容

| Topic | 文章数 | 内容 |
|-------|--------|------|
| im-channel | 3 | IM Channel 技术调研（总览、方案对比、飞书设计） |
| cc-tooling | 2 | CC 工具链（llm-wiki 调研、Obsidian 搭建记录） |

### 7.2 自动研究成果

使用 `/wiki:research` 对 cat-cafe-tutorials 和 clowder-ai 进行了自动研究：

- Round 1：5 个 Agent，7 个源，5 篇文章
- Round 2（deep）：8 个 Agent，6 个新源，5 篇新文章
- 共产出 10 篇编译后的 wiki 文章

## 八、经验总结

### 8.1 适用场景

- **适合**：技术调研、学习新领域、项目知识沉淀
- **不适合**：日记、快速 TODO、临时笔记

### 8.2 成本控制

- Standard 模式（5 Agent）适合大多数场景
- Deep/Retardmax 模式 token 消耗高，仅在深度研究时使用
- `--min-time` 配合 Standard 模式性价比最高

### 8.3 与其他工具的关系

| 工具 | 角色 | 与 llm-wiki 的关系 |
|------|------|-------------------|
| Claude Code | 编译器 | 通过 `/wiki:*` 命令操作 |
| Obsidian | 查看器 | 只读查看 wiki 文章和图谱 |
| Auto Memory | 短期记忆 | 用户偏好、协作模式（不同域，不冲突） |
| llm-wiki | 长期知识 | 技术调研、学习笔记、项目知识 |

## 参考资料

- [Karpathy 原始 Gist](https://gist.github.com/karpathy/0e07921f03ab9a0a90e1efd6b26539ed)
- [nvk/llm-wiki](https://github.com/nvk/llm-wiki)
- [rvk7895/llm-knowledge-bases](https://github.com/rvk7895/llm-knowledge-bases)
