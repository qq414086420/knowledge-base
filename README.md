# 📚 文档与分析报告

本目录包含按主题组织的研究报告、分析文档和技术调查。

## 📂 目录结构

```
docs/
├── ai-memory-systems/          # AI agent 记忆与自学习系统
├── mcp-servers/                # Model Context Protocol 服务器和工具
├── proxy-configurations/       # 网络代理配置方案
├── workflow-patterns/          # 开发工作流模式和最佳实践
└── tools-integration/          # 工具集成和自动化配置
```

## 📋 文件命名规范

- **调查报告：** `主题-调查-YYYY-MM-DD.md`
- **对比分析：** `主题-对比.md`
- **深度分析：** `主题-分析-*.md`
- **指南文档：** `主题-指南.md` 或 `主题-教程.md`

## 🎯 主题说明

### ai-memory-systems/
AI agent 记忆系统、知识图谱和自学习架构的研究。

**主要系统：**
- Zep（时序知识图谱）
- Letta/MemGPT（agent 自控记忆）
- Mem0（生产就绪的记忆层）
- MCP Memory Servers

**当前文件：**
- `AI-MEMORY-AND-LEARNING-MCP-SURVEY.md` - MCP 调查报告
- `AI-MEMORY-SYSTEMS-COMPARISON.md` - 系统对比分析

### mcp-servers/
Model Context Protocol 服务器的文档、集成和自定义实现。

### proxy-configurations/
网络代理配置方案、包装器模式和配置指南。

**示例：**
- MCP server 代理包装器
- Node.js fetch 代理配置
- WSL 网络配置

### workflow-patterns/
开发工作流模式、自动化策略和最佳实践。

### tools-integration/
工具集成、自动化脚本和跨平台配置。

### im-channel-architecture-comparison.md
Claude Code Channels / cc-connect / OpenClaw 三种 AI Agent 接入 IM 平台的架构方案对比，含飞书集成建议和 Issue #36800 分析。

---

## 📝 添加新报告

创建新分析报告时：

1. **选择合适的文件夹** 根据主要主题
2. **遵循命名规范**（见上方）
3. **包含元数据头部**：
   ```markdown
   # 报告标题

   **日期：** YYYY-MM-DD
   **作者：** Claude Code
   **分类：** 调查 | 分析 | 对比 | 指南
   **关键词：** 关键词1, 关键词2, 关键词3
   ```

4. **更新此 README** 如果添加新的主题文件夹

---

## 📊 已完成的研究

### AI Memory Systems (2026-03-22)
- ✅ 12+ 记忆系统调查
- ✅ Zep vs Letta vs Mem0 对比
- ✅ 与 Continuous Learning v2 的对比分析
- ✅ 改进建议（短期/中期/长期）

### MCP Server Proxy Wrapper (2026-03-22)
- ✅ Node.js fetch 代理问题分析
- ✅ undici ProxyAgent 解决方案
- ✅ Wrapper 模式实施指南
- ✅ 代码模板和配置示例

---

**最后更新：** 2026-04-06
