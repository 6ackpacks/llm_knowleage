---
title: LLM 知识库工作流
created: 2026-04-03
updated: 2026-04-03
sources:
  - $raw/Thread by @karpathy.md
tags:
  - LLM
  - 知识库
  - PKM
  - Obsidian
  - workflow
related:
  - [[RAG]]
  - [[个人知识管理]]
---

# LLM 知识库工作流

## 核心定义

利用 LLM 自动构建和维护个人知识库的工作流：将原始资料归入 `raw/` 目录，LLM 将其增量"编译"为结构化的 Markdown wiki，再通过问答和持续整理不断扩充知识库。由 Andrej Karpathy 提出并实践（2026-04-02）。

## 关键要点

- **数据分层**：原始资料（`raw/`）与编译产物（`wiki/`）严格分离，LLM 负责 wiki 的全部写作与维护
- **增量编译**：每次有新原始资料时，LLM 只补充或更新受影响的 wiki 文章，不全量重写
- **自维护索引**：LLM 自动维护 `index.md` 和各文章的简要摘要，无需 RAG 即可在小规模（~100 篇 / ~40 万字）下做复杂问答
- **Obsidian 作为前端**：用 Obsidian 查看 raw 数据、wiki 文章和衍生可视化；Marp 插件可渲染幻灯片输出
- **输出归档**：问答产生的 markdown / 图表等输出可再"归档"回 wiki，使知识库持续增长
- **Linting / 健康检查**：定期让 LLM 检查数据一致性、补全缺失信息、发现新条目候选
- **自定义工具**：可 vibe-code 一个轻量搜索引擎，供 LLM 通过 CLI 工具调用处理大型查询

## 详细内容

### 数据导入

Karpathy 使用 **Obsidian Web Clipper** 浏览器扩展将网页文章转为 `.md` 文件，再用快捷键将页面图片下载到本地，确保 LLM 能直接引用图片。源文档类型包括：文章、论文、代码仓库、数据集、图片等。

### 为什么不需要 RAG

在 ~100 篇文章、~40 万字的规模下，LLM 可以直接读取 index 文件和相关文章摘要来完成问答，无需向量数据库或嵌入检索。关键在于 LLM 自动维护的索引文件质量足够高。

### 输出格式

- Markdown 文件（Obsidian 内查看）
- Marp 格式幻灯片
- Matplotlib 图表
- 问答结果可归档回 wiki 继续积累

### 未来方向

随着知识库规模增大，可考虑**合成数据生成 + 微调**，让 LLM 在模型权重中"记住"这些知识，而不仅依赖上下文窗口。

## 相关资料

- [[RAG]]
- [[个人知识管理]]
- [[Obsidian]]

## 原始来源

- [Thread by @karpathy (2026-04-02)]($raw/Thread%20by%20%40karpathy.md)
