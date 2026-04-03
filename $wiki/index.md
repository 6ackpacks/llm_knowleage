# 知识库索引

> 此文件由 LLM 自动维护，请勿手动编辑。
> 最后更新：2026-04-03

## 统计

| 指标 | 数量 |
|------|------|
| 原始资料 | 4 |
| Wiki 文章 | 4 |
| 概念条目 | 4 |
| Output 归档 | 1 |

## 文章列表

| 文章 | 类型 | 标签 | 更新日期 |
|------|------|------|----------|
| [[Harness Engineering]] | 概念 | Harness Engineering, Agent, LLM, AI工程, 架构 | 2026-04-03 |
| [[Agent Scaling 三个维度]] | 概念 | Harness Engineering, Scaling, Anthropic, OpenAI, Cursor | 2026-04-03 |
| [[Harness小白实践指南]] | 实践指南 | Harness Engineering, 入门, 实践 | 2026-04-03 |
| [[LLM知识库工作流]] | 概念 | LLM, 知识库, PKM, Obsidian, workflow | 2026-04-03 |

## 概念图谱

### 核心概念

```
Harness Engineering（核心）
├── 三根支柱
│   ├── 评估闭环 → Anthropic 实践
│   ├── 架构约束 → OpenAI Codex 实践
│   └── 记忆治理 → PrismerCloud
├── Agent Scaling 三个维度
│   ├── 时间 Scalability（Anthropic）— evaluator 三角色架构
│   ├── 空间 Scalability（Cursor）— 递归 Planner-Worker
│   └── 交互 Scalability（OpenAI）— Symphony ticket 驱动
└── 关联概念
    └── LLM 知识库工作流 — Karpathy 的 raw→wiki 方法论
        （Context Infrastructure，与 Harness 互补）
```

### 概念关系

- **Harness Engineering** 解决 Agent 的工作方式和协调
- **Context Infrastructure / LLM知识库工作流** 解决 Agent 的认知密度（两者互补）
- **Agent Scaling 三个维度** 是理解 Harness Engineering 各家实践的统一框架

## 待深入研究的主题（建议）

1. **Symphony 开源实现** — Elixir/BEAM 构建的 agent job scheduler，值得深入研究其架构设计
2. **evaluator 独立性设计** — Anthropic 三角色架构中 evaluator 与 generator 解耦的具体实现
3. **为 Agent 优化的 repo 结构** — Cursor 发现 monolith → 多 crate 大幅提升吞吐量，设计原则待整理
4. **模型角色适配** — 不同 scaling 维度对模型能力的不同要求（长上下文一致性 vs 自主判断力）
5. **Context Infrastructure** — 与 Harness 互补的"认知密度"方向（yage.ai 的参考实现）
