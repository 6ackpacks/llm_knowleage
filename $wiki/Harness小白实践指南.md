---
title: Harness 小白实践指南
created: 2026-04-03
updated: 2026-04-03
sources:
  - $wiki/Harness Engineering.md
  - $wiki/Agent Scaling 三个维度.md
output_origin: $output/2026-04-03_Harness小白实践全流程.md
tags:
  - Harness Engineering
  - 实践指南
  - 入门
  - Agent
related:
  - [[Harness Engineering]]
  - [[Agent Scaling 三个维度]]
---

# Harness 小白实践指南

> 本文是 [[Harness Engineering]] 的实践衍生文章，聚焦"怎么做"而非"是什么"。面向刚接触 Agent 工程的开发者，提供可直接落地的分阶段路径。

## 核心认知：三个必须放弃的旧习惯

| 旧习惯 | 为什么错 | 数据 |
|-------|---------|-----|
| "换更好的模型就能解决" | 瓶颈往往在 Harness，不在模型 | 只改 Harness，通过率 52.8%→66.5% |
| "写一个超长 prompt 就万事大吉" | 挤压上下文，Agent 模式匹配而非真正理解，且立刻腐化 | — |
| "一次写好永久有效" | Manus 6 个月重构 5 次，正常节奏 | Build to Delete |

## 搭建顺序：三层不能跳

```
第一层：Prompt Engineering   ← 约束语句 > 期望语句
第二层：Context Engineering  ← 先补这一层（知识进 repo）
第三层：Harness Engineering  ← 三根支柱
```

跳过第二层直接上第三层，Harness 会因为 Agent"什么都不知道"而失效。

## 第二层关键动作：把知识放进 Repo

Agent 在运行时看不到的东西，等于不存在。Slack/Google Docs/人脑里的知识对 Agent 是空白。

```
项目根目录/
├── AGENTS.md          ← 目录，约 100 行，告诉 Agent "去哪找什么"
├── ARCHITECTURE.md    ← 分层规则
└── docs/
    ├── design/        ← 设计决策记录
    ├── product/       ← 产品需求
    └── references/    ← llms.txt 等 Agent 友好文档
```

## 三根支柱：分阶段实施

### 支柱一：评估闭环（最先搭）

**核心：Agent 不能自己给自己打分。**

| 阶段 | 方案 | 适用场景 |
|-----|------|---------|
| 阶段 1 | 任务描述里附验收清单，强制 Agent 逐项回答"是/否" | 今天就能做 |
| 阶段 2 | `task_spec.md`（做什么）+ `eval_contract.md`（怎么算好），两次独立调用 | 任务复杂度中等 |
| 阶段 3 | Planner + Generator + Evaluator 三角色，Evaluator 操作真实应用验证 | Agent 频繁"自我说服"时 |

**升级信号**：Agent 开始说"这个 bug 影响不大"时，上阶段 3。

### 支柱二：架构约束（第二搭）

**核心：约束比指令有效。约束越多，反而越可靠。**

| 阶段 | 方案 |
|-----|------|
| 阶段 1 | AGENTS.md 里用禁止语（"禁止在 UI 里直接调用数据库"） |
| 阶段 2 | 定义分层架构 + `check_arch.py` 在 CI 里跑，错误信息写成修复指引 |
| 阶段 3 | 让 Agent 自己生成并维护 linter |

**立刻能用**：减少工具数量。Vercel 删掉 80% 工具，效果更好。

### 支柱三：记忆治理（有多 Agent 再搭）

**核心：一个 Agent 的幻觉会通过共享知识库污染所有 Agent。**

单 Agent → 先不管。多 Agent → 加验证门：

```
Agent 写入经验 → "待验证" → 独立调用验证 → "已确认" → 其他 Agent 可读
```

## 熵对抗：防止系统腐化

定期运行"清洁 Agent"（每周一次）：

```markdown
扫描 docs/，找出：
1. 代码已删除但文档还在描述的功能
2. AGENTS.md 里指向不存在文件的链接
3. 架构规则被绕过的代码模式
→ 对每个发现开修复 PR，等待人工确认
```

## 场景路径图

### 场景 A：个人开发者

最小可行 Harness = `AGENTS.md`（地图）+ 验收清单 + 禁止语规则

工具：Claude Code / Cursor，不需要额外框架。

### 场景 B：小团队

= 完整 docs/ + 独立 eval contract + CI 架构检查 + 每周清洁 Agent

工具：LangChain DeepAgents / OpenHands。

### 场景 C：多 Agent 并行

先判断你面临哪个 Scaling 维度：

| 痛点 | 维度 | 解法 |
|-----|------|-----|
| 单个 Agent 跑久了就偏 | 时间 | Planner+Generator+Evaluator |
| 多 Agent 并行互相干扰 | 空间 | 递归 Planner-Worker，Worker 完全隔离 |
| Agent 太多人来不及管 | 交互 | ticket 驱动 + Proof of Work |

工具：DeerFlow 2.0。

## 三个最常见的坑

1. **AGENTS.md 写成手册** → 超 300 行就腐化，细节推到 `docs/`
2. **完美主义拖住吞吐量** → 纠错比等待便宜，设定"够好就合并"标准
3. **Agent 失败去改 Agent** → 这是信号，改 Harness，不改 Agent

## 相关资料

- [[Harness Engineering]] — 概念定义与三根支柱详解
- [[Agent Scaling 三个维度]] — 时间/空间/交互三个维度深度解析

## 原始来源

- 由 `$output/2026-04-03_Harness小白实践全流程.md` 归档整理
