# LLM 知识库 / LLM Knowledge Base

用 LLM 自动构建和维护的个人知识库，灵感来自 [Andrej Karpathy 的工作流](https://x.com/karpathy/status/2039805659525644595)。

## 目录结构

```
$raw/      原始资料（文章、推文、论文等）
$wiki/     编译产物（概念文章、实践指南），由 LLM 维护
$output/   分析报告、问答产出，可选择性归档回 $wiki/
```

## 当前 Wiki 文章

- [Harness Engineering]($wiki/Harness%20Engineering.md) — Agent 工程的核心框架
- [Agent Scaling 三个维度]($wiki/Agent%20Scaling%20%E4%B8%89%E4%B8%AA%E7%BB%B4%E5%BA%A6.md) — 时间/空间/交互三个扩展维度
- [Harness 小白实践指南]($wiki/Harness%E5%B0%8F%E7%99%BD%E5%AE%9E%E8%B7%B5%E6%8C%87%E5%8D%97.md) — 从零开始的落地路径
- [LLM 知识库工作流]($wiki/LLM%E7%9F%A5%E8%AF%86%E5%BA%93%E5%B7%A5%E4%BD%9C%E6%B5%81.md) — Karpathy 的 raw→wiki 方法论

## 工作流

```
$raw/（原始资料）
  ↓ LLM 编译
$wiki/（结构化知识）
  ↓ LLM 问答/分析
$output/（产出报告）
  ↓ 有价值的归档回
$wiki/
```

详见 [$wiki/RULES.md]($wiki/RULES.md)。
